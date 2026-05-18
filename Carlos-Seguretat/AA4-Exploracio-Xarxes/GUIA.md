# INFORME TÈCNIC: RA4.AA4 Exploració de xarxes amb Kali Linux (Pràctica feta a casa)

**Autor:** Rui Huang - 2n A, Número de llista 11

**Mòdul:** 0226 Seguretat Informàtica (CFGM SMiX)

---

## 1. Preparació de l'Entorn
L'adaptador de xarxa ha d'estar en pont, utilitzant l'adreça habitual (192.168.2.x per al 2n A o 192.168.4.x per al 2n B), i cal configurar la porta d'enllaç i el DNS manualment per disposar de connectivitat. A més, han d'estar enceses la porta d'enllaç (VyOS) i el servidor Ubuntu ubicats a l'equip del projector.

*   El rang sencer de la meva xarxa és: **192.168.2.7/24**
*   El meu router (la porta d'enllaç) té la primera IP d'aquest rang: **192.168.2.254**.

Aquestes són les adreces que utilitzaré per als següents escanejos.

![alt text](<pics/Captura de pantalla 2026-05-18 193528.png>)

---

## 2. Exploració de la xarxa amb Netdiscover
Netdiscover és una eina de terminal que permet fer exploracions en xarxa local, tant actives (enviant paquets ARP) com passives. 

### 2.1 Exploració en Mode Actiu
En el mode actiu, l'eina explorarà la xarxa de forma contínua enviant paquets ARP per descobrir quins equips hi ha connectats.

*   **Comanda utilitzada:**
`netdiscover -r 192.168.2.0/24`

*   **Resultats:** S'han forçat les respostes de tots els equips connectats a casa, obtenint les seves adreces IP, adreces MAC i els fabricants de les targetes de xarxa (MAC Vendor).

![alt text](<pics/Captura de pantalla 2026-05-18 194357.png>)

### 2.2 Exploració en Mode Passiu
Seguidament, he realitzat l'exploració passiva, la qual no envia res i només escolta el trànsit.

*   **Comanda utilitzada:**
`netdiscover -p`

*   **Resultats:** En aquest mode no s'ha enviat cap paquet de petició; el sistema simplement s'ha quedat "escoltant" el trànsit ARP que circulava per la meva xarxa local de forma natural. *(Nota: Perquè apareguessin resultats, he hagut de generar trànsit navegant des del meu telèfon mòbil).*

![alt text](<pics/Captura de pantalla 2026-05-18 194602.png>)

### 2.3 Explicació de les diferències entre resultats
La diferència principal rau en l'emissió de trànsit. L'exploració activa envia constants peticions ARP, la qual cosa la fa molt ràpida i completa, però genera molt "soroll" a la xarxa. Per altra banda, les exploracions passives són les més adequades per preparar atacs de forma totalment invisible, ja que, en no generar paquets, no dispararan alertes en sistemes de detecció d'intrusos (IDS) com Snort o Suricata.

---

## 3. Exploració mitjançant Nmap
Nmap és l'eina més famosa per explorar xarxes i dispositius, permetent identificar equips i explorar quins ports estan oberts.

### 3.1 Exploració ràpida de la xarxa domèstica
He realitzat un escaneig per veure quins equips de la meva casa responen i així comprovar l'inventari obtingut anteriorment.

*   **Comanda utilitzada:**
`nmap -sn 192.168.0.0/24`

![alt text](<pics/Captura de pantalla 2026-05-18 194905.png>)

### 3.2 Detecció de ports del router domèstic
Tal com s'indica a les instruccions de la pràctica per als alumnes que la fem des de casa, he dirigit l'exploració detallada exclusivament cap al **meu router domèstic**.

S'ha utilitzat l'opció per determinar el sistema operatiu i poder veure clarament tant la informació del sistema com els serveis i ports oberts.

*   **Comanda utilitzada:**
`nmap -O -sV 192.168.2.3`

`nmap -O -sV 192.168.2.8`

**Resultats obtinguts:** A partir de l'escaneig amb Nmap sobre les IP router VyOS i al servidor Ubuntu (192.168.2.3 i 192.168.2.8), he detectat que no tenen cap port obert dins dels 1000 ports TCP més comuns escanejats, ja que tots apareixen en estat tancat (reset). Altres ports habituals com el 22 (ssh), 23 (telnet) o 443 (https) es troben completament inactius. Respecte al sistema operatiu, Nmap no dona una coincidència exacta (Too many fingerprints match) a causa de la manca de ports oberts per a interactuar, però detecta clarament que l'adreça MAC de tots dos dispositius pertany al fabricant Oracle VirtualBox virtual NIC, evidenciant que es tracta d'entorns virtualitzats.

![alt text](<pics/Captura de pantalla 2026-05-18 195827.png>)

---

[Tornar enrere](README.md)