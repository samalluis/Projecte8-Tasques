# INFORME TÈCNIC: Configuració de Tallafocs UFW a Zorin

## 0. Requisits Previs i Entorn
Necessitem una màquina Linux amb configuració amb dues targetes de xarxa: una en NAT i la segona en host-only. 
Al llarg de l'activitat caldrà utilitzar serveis de SSH i HTTP, per tant, és imprescindible instal·lar ssh i nginx executant les següents comandes:

```
sudo apt install ssh -y
sudo apt install nginx -y
```
![alt text](<pics/Captura de pantalla 2026-05-05 185036.png>)

![alt text](<pics/Captura de pantalla 2026-05-05 185116.png>)

---

## FASE 1: Inicialització i Comportament per Defecte (Activitats-I)

### 1. Estat i Regles per Defecte del Tallafocs (Criteri: "Mostra regles definides. Explica quines són les regles per defecte") 
*   **Comandes a executar:** 
    *   `sudo ufw status` per conèixer l'estat del firewall (habitualment ve deshabilitat en Ubuntu Server).
    *   `sudo ufw enable` per habilitar el firewall i que comenci a filtrar.
    *   `sudo ufw status verbose` per mostrar informació detallada de l'estat i les regles.

![alt text](<pics/Captura de pantalla 2026-05-05 185227.png>)

![alt text](<pics/Captura de pantalla 2026-05-05 185348.png>)

*   **Explicació Tècnica:** UFW funciona mitjançant regles que poden ser d'acceptació (allow) o denegació (deny). Al llançar la comanda en mode `verbose`, podem veure que el comportament i les regles per defecte són denegar el trànsit d'entrada (`deny incoming`) i permetre el de sortida (`allow outgoing`). Si hi ha alguna regla definida prèviament, s'ha d'eliminar per deixar només el comportament per defecte.

![alt text](<pics/Captura de pantalla 2026-05-05 185413.png>)

### 2. Comprovació de la regla "deny" per defecte (Criteri: "Comprova regla deny per defecte")
*   **Comandes a executar:** `sudo ufw default deny` per assegurar el bloqueig del trànsit d'entrada.

![alt text](<pics/Captura de pantalla 2026-05-05 185604.png>)

*   **Acció:** Connecta't des de l'amfitrió a aquest equip via SSH i observa que no et pots connectar.
*   **Explicació Tècnica:** A l'intentar establir una connexió TCP pel port 22 des de l'amfitrió, el tallafocs intercepta el paquet. En aplicar-se la directiva per defecte (`deny`), la petició es descarta i la connexió falla.

![alt text](<pics/Captura de pantalla 2026-05-05 191825.png>)

![alt text](<pics/Captura de pantalla 2026-05-05 191836.png>)

### 3. Aplicació de regla "deny" al trànsit de sortida (Criteri: "Aplica regla deny trànsit de sortida per defecte")
*   **Comandes a executar:** `sudo ufw default deny outgoing`.
*   **Acció:** Aplica la regla per defecte deny al trànsit de sortida i prova ara a fer un ping a Google.
*   **Explicació Tècnica:** Modifiquem la directiva general per denegar el trànsit de sortida. Els paquets ICMP que genera la comanda `ping` són descartats internament pel sistema abans de poder sortir per la interfície NAT.


![alt text](<pics/Captura de pantalla 2026-05-05 192015.png>)

---

## FASE 2: Gestió Específica de Trànsit (Activitats-II)

### 4. Restauració del trànsit de sortida (Criteri: "Aplica regla allow trànsit de sortida per defecte")
*   **Comandes a executar:** `sudo ufw default allow outgoing`.
*   **Acció:** Comprova que ara sí que pots fer el ping a Google.
*   **Explicació Tècnica:** Es restableix la política de sortida permetent el trànsit. Els paquets ICMP ara poden travessar el tallafoc cap a Internet i en rebem la resposta correctament.

![alt text](<pics/Captura de pantalla 2026-05-05 192127.png>)

### 5. Prohibició d'un lloc a Internet (Criteri: "Regla per prohibir lloc a Internet")
*   **Comandes a executar:** 
    *   Primer, resol la IP: `ping -c 1 capgros.elnacional.cat` (apunta la IP).
    *   Aplica la regla per denegar connexions a aquesta adreça de sortida: `sudo ufw deny out to [IP]`.
*   **Acció:** Crea la regla per bloquejar el trànsit cap a la IP de capgros.elnacional.cat i fes un ping per verificar-ho.
*   **Explicació Tècnica:** Les regles s'apliquen per ordre de preferència, sent molt important definir les específiques primer per evitar problemes. Aquesta regla prohibeix el trànsit si el paquet de sortida (out) té com a destinació exacta aquella IP específica. Com que la regla 1 ho prohibeix, se sobreposa a la regla general de permís.

![alt text](<pics/Captura de pantalla 2026-05-05 193101.png>)

### 6. Habilitació de Nginx per a l'amfitrió (Criteri: "Habilita regla per servidor nginx des de l'amfitrió")
*   **Comandes a executar:** `sudo ufw allow from 192.168.56.107 to any port 80 proto tcp`.
*   **Acció:** Habilita el trànsit d'entrada pel servei nginx des de la IP de l'amfitrió (192.168.56.107) i comprova-ho.
*   **Explicació Tècnica:** Es poden fer combinacions entre adreces IP i ports a l'hora de definir les regles. El tallafoc només permetrà connexions entrants al port 80 (TCP) si provenen exclusivament de la IP de l'equip amfitrió.

![alt text](<pics/Captura de pantalla 2026-05-05 193253.png>)

![alt text](<pics/Captura de pantalla 2026-05-05 193314.png>)

### 7. Comprovació de restricció d'IP a Nginx (Criteri: "Canvia regla per servidor nginx amb una IP diferent")
*   **Comandes a executar:** 
    *   `sudo ufw status numbered` (per veure el número de la regla definida a l'apartat anterior).
    *   `sudo ufw delete [número]` (per eliminar la regla de la IP 192.168.56.1).
    *   `sudo ufw allow from 192.168.56.222 to any port 80 proto tcp` (IP inventada).

![alt text](<pics/Captura de pantalla 2026-05-05 193649.png>)

*   **Acció:** Intenta recarregar la web de Nginx des del navegador de l'amfitrió i comprova el resultat.
*   **Explicació Tècnica:** En canviar la regla per a una IP diferent, l'amfitrió perd l'accés. Com que la seva IP ja no fa coincidència amb l'excepció, s'hi aplica la política de denegació de trànsit d'entrada per defecte.


![alt text](<pics/Captura de pantalla 2026-05-05 193804.png>)


### 8. Revisió Final (Criteri: "Mostra regles creades") 
*   **Comandes a executar:** `sudo ufw status numbered` per poder veure les regles definides de forma numerada.
*   **Explicació Tècnica:** Mostrem el conjunt de totes les regles que tenim definides de trànsit d'entrada, de sortida i d'enrutament. Això ens serveix com a llista de control d'accés final (ACL) i demostra visualment l'ordre d'aplicació.

![alt text](<pics/Captura de pantalla 2026-05-05 194044.png>)

---
[Tornar enrere](README.md)
