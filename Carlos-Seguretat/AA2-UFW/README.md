# Pràctica: Configuració de Tallafocs (UFW) a Zorin

Aquest repositori conté la guia i els exercicis pràctics per aprendre a configurar i gestionar el tallafoc UFW (*Uncomplicated Firewall*) en un entorn Linux.

## Requisits Previs

Abans de començar l'activitat, assegura't de complir amb els següents requeriments al teu entorn virtual:

**1. Màquina Linux**
Necessites una màquina amb configuració de dues targetes de xarxa:
* Una interfície en mode **NAT** (per a accés a Internet).
* Una interfície en mode **Host-only** (per a comunicació local amb l'equip amfitrió).

**2. Instal·lació dels serveis imprescindibles**
Al llarg de l'activitat caldrà utilitzar els serveis d'SSH i HTTP. Executa aquestes dues comandes de forma independent al teu terminal:

`sudo apt install ssh -y`

`sudo apt install nginx -y`

---

## Part 1: Activitats-I (Comportament per defecte)

En aquesta primera fase, analitzarem l'estat inicial del tallafoc i les seves polítiques globals.

**1. Comprovació inicial:**
Comprova l'estat actual del tallafocs amb la comanda `sudo ufw status` i, si cal, habilita'l executant `sudo ufw enable`.

**2. Anàlisi de regles per defecte:**
Mostra les regles que tens definides utilitzant `sudo ufw status verbose`. Identifica i explica quines són les regles per defecte en el teu informe.
*Nota:* Si hi ha alguna regla definida prèviament, elimina-la amb la comanda `sudo ufw delete [número]`, deixant únicament el comportament per defecte.

**3. Prova de bloqueig d'entrada:**
Comprova la regla per defecte de `deny` per al trànsit d'entrada aplicant `sudo ufw default deny`. Per fer-ho, intenta connectar-te des de l'equip amfitrió a aquesta màquina virtual via SSH i observa que la connexió falla.

**4. Bloqueig de sortida:**
Aplica la regla per defecte de `deny` al trànsit de sortida executant `sudo ufw default deny outgoing`. Comprova que s'ha aplicat correctament intentant fer un `ping` a Google i observant l'error.

---

## Part 2: Activitats-II (Regles Específiques)

En aquesta segona fase, crearem excepcions i regles granulars per permetre o denegar trànsit específic.

**5. Restauració de sortida:**
Aplica la regla per defecte `allow` al trànsit de sortida mitjançant `sudo ufw default allow outgoing`. Comprova que ara sí que pots fer `ping` a Google de forma reeixida.

**6. Bloqueig d'un domini específic:**
Crea una regla per prohibir el trànsit cap a l'adreça IP de *capgros.elnacional.cat* fent servir `sudo ufw deny out to [IP_DOMINI]`. Comprova que la regla funciona realitzant un `ping` cap a aquell domini i veient que es bloqueja.

**7. Permís d'entrada a Nginx:**
Habilita el trànsit d'entrada pel servei Nginx **únicament** per a la IP de l'equip amfitrió (ex. *192.168.56.1*) utilitzant la comanda `sudo ufw allow from 192.168.56.1 to any port 80 proto tcp`. Comprova des del navegador de l'amfitrió que la web connecta correctament.

*Pas extra avaluable (Rúbrica):* Canvia la regla del servidor Nginx per assignar-la a una IP diferent a la de l'amfitrió i comprova al teu navegador que perds l'accés a la web.

**8. Revisió final:**
Mostra al terminal el conjunt de totes les regles que han quedat definides després de realitzar la pràctica fent servir la comanda `sudo ufw status numbered`.

---

## Criteris de Qualificació (Rúbrica)

Per obtenir la màxima puntuació en aquesta pràctica, el teu informe haurà de complir estrictament amb els següents nivells d'avaluació per a cada tasca:

| Criteri a Avaluar | 0 Punts | 1 Punt (Incomplet) | 2 Punts (Nota Màxima) |
| :--- | :--- | :--- | :--- |
| **Mostra regles definides. Explica quines són les regles per defecte** | No es fa. | Es mostren regles però no s'expliquen correctament. | **Es mostren i s'expliquen correctament.** |
| **Comprova regla deny per defecte** | No es fa. | Es documenta parcialment. | **Es documenta de forma correcta.** |
| **Aplica regla deny trànsit de sortida per defecte** | No es fa. | Regla s'aplica correctament (però no es comprova). | **S'aplica la regla i es comprova.** |
| **Aplica regla allow trànsit de sortida per defecte** | No es fa. | Regla s'aplica correctament (però no es comprova). | **S'aplica la regla i es comprova.** |
| **Regla per prohibir lloc a Internet** | No es fa. | Regla s'aplica correctament (però no es comprova). | **S'aplica la regla i es comprova.** |
| **Habilita regla per servidor nginx des de l'amfitrió** | No es fa. | Regla s'aplica correctament (però no es comprova). | **S'aplica la regla i es comprova.** |
| **Canvia regla per servidor nginx amb una IP diferent** | No es fa. | Regla s'aplica correctament (però no es comprova). | **S'aplica la regla i es comprova.** |
| **Mostra regles creades** | No es fa. | Manquen regles a la llista. | **Es mostren totes les regles creades.** |

>  **Puntualitat:** La data límit de lliurament és el **19 de maig**. Un lliurament tard implicarà una penalització de **-3.2 punts**.

---

[Guia de solucio](GUIA.md)
