# 📘 Informe Tècnic: Exploració de Xarxes amb Kali Linux
## RA4.AA4 — Pràctica de Laboratori (Entorn Domèstic)

> **Assignatura:** Mòdul 0226 — Seguretat Informàtica (CFGM SMiX)  
> **Autor:** Rui Huang — 2n A, Llista Nº 11  
> **Data d'elaboració:** Maig 2026

---

## 🏗️ 1. Preparació de l'Entorn de Laboratori

Per dur a terme aquesta pràctica d'exploració de xarxes, va ser necessari configurar prèviament la màquina virtual de Kali Linux amb els següents paràmetres de xarxa:

| Paràmetre | Configuració | Descripció |
|-----------|--------------|------------|
| **Mode d'adaptador** | Pont (Bridged) | Permet que la VM comparteixi la mateixa xarxa física que l'amfitrió |
| **Rang de xarxa** | `192.168.2.0/24` | Subxarxa assignada al grup 2n A |
| **Porta d'enllaç** | `192.168.2.254` | Adreça del router VyOS (primera IP del rang) |
| **DNS** | Configuració manual | Resolució de noms habilitada |

> 💡 **Nota:** Addicionalment, havien d'estar operatius tant la porta d'enllaç (VyOS) com el servidor Ubuntu ubicat a l'equip del projector.

Aquestes adreces IP serviran com a objectiu per als escanejos que es realitzaran a continuació.

![Configuració de xarxa de Kali Linux](<pics/Captura de pantalla 2026-05-18 193528.png>)

---

## 🔍 2. Descobriment d'Amfitrions amb Netdiscover

Netdiscover és una eina de línia de comandes dissenyada per al reconeixement d'equips en xarxes locals, operant tant en mode actiu (mitjançant l'enviament de paquets ARP) com en mode passiu (mitjançant escolta passiva del trànsit).

---

### 2.1 — Mode Actiu: Sondeig ARP Proactiu

En mode actiu, l'eina realitza un sondeig continu de la xarxa transmetent paquets ARP de sol·licitud per identificar tots els dispositius connectats.

| Comanda | Descripció |
|---------|------------|
| `netdiscover -r 192.168.2.0/24` | Escaneig actiu del rang complet de la subxarxa |

**Resultats obtinguts:** El sondeig va forçar respostes de tots els equips connectats a l'entorn, permetent identificar les seves adreces IP, adreces MAC i els fabricants corresponents de les interfícies de xarxa (MAC Vendor).

![Resultats de l'escaneig actiu amb Netdiscover](<pics/Captura de pantalla 2026-05-18 194357.png>)

---

### 2.2 — Mode Passiu: Escolta ARP Silenciosa

Posteriorment, es va executar el sondeig en mode passiu, el qual no genera trànsit propi i es limita a capturar el trànsit ARP existent a la xarxa.

| Comanda | Descripció |
|---------|------------|
| `netdiscover -p` | Escaneig passiu sense generació de paquets propis |

**Resultats obtinguts:** En aquest mode no es va transmetre cap petició ARP; el sistema va romandre en estat d'escolta passiva, capturant únicament el trànsit ARP que circulava de forma natural per la xarxa local.

> 💡 **Nota tècnica:** Perquè apareguessin resultats en el mode passiu, va ser necessari generar trànsit ARP addicional navegant des d'un dispositiu mòbil connectat a la mateixa xarxa.

![Resultats de l'escaneig passiu amb Netdiscover](<pics/Captura de pantalla 2026-05-18 194602.png>)

---

### 2.3 — Anàlisi Comparatiu: Actiu vs. Passiu

| Característica | Mode Actiu | Mode Passiu |
|----------------|------------|-------------|
| **Generació de trànsit** | Envia peticions ARP constants | No genera paquets propis |
| **Velocitat de descobriment** | Molt ràpida i completa | Depèn del trànsit existent |
| **Visibilitat a la xarxa** | Alta — genera "soroll" detectable | Nul·la — totalment invisible |
| **Detecció per IDS/IPS** | Pot disparar alertes en Snort/Suricata | No activa sistemes de detecció |
| **Ús recomanat** | Inventari ràpid de xarxa | Fase de reconeixement furtiu |

**Conclusió de l'anàlisi:** La diferència fonamental resideix en la generació de trànsit. El sondeig actiu emet peticions ARP de forma contínua, la qual cosa el fa extremadament ràpid i exhaustiu, però a costa de generar un elevat nivell d'activitat a la xarxa. Per contra, el sondeig passiu resulta l'opció més adequada per a la fase de reconeixement prèvia a un atac, ja que en no generar paquets propis, no activa sistemes de detecció d'intrusos (IDS) com Snort o Suricata, mantenint una presència completament invisible a la xarxa.

---

## 🌐 3. Exploració de Xarxa amb Nmap

Nmap (*Network Mapper*) constitueix l'eina de referència per a l'escaneig de xarxes i dispositius, permetent identificar amfitrions actius i determinar quins ports i serveis es troben exposats.

---

### 3.1 — Sondeig d'Amfitrions a la Xarxa Domèstica

Es va realitzar un escaneig de descobriment d'amfitrions per verificar quins equips de la xarxa responien a les peticions, validant així l'inventari obtingut prèviament amb Netdiscover.

| Comanda | Descripció |
|---------|------------|
| `nmap -sn 192.168.2.0/24` | Sondeig d'amfitrions actius sense escaneig de ports |

![Resultats del sondeig d'amfitrions amb Nmap](<pics/Captura de pantalla 2026-05-18 194905.png>)

---

### 3.2 — Anàlisi Detallat del Router VyOS i Servidor Ubuntu

| Comanda | Descripció |
|---------|------------|
| `nmap -O -sV 192.168.2.3` | Detecció de SO i versions de serveis (VyOS) |
| `nmap -O -sV 192.168.2.8` | Detecció de SO i versions de serveis (Ubuntu) |

**Trobades de l'anàlisi:**

A partir de l'escaneig realitzat amb Nmap sobre les adreces IP del router VyOS (`192.168.2.3`) i del servidor Ubuntu (`192.168.2.8`), es va detectar que **cap dels dos dispositius presenta ports oberts** dins dels 1000 ports TCP més comuns analitzats, ja que tots apareixen en estat tancat (reset). Ports habitualment actius com el 22 (SSH), 23 (Telnet) o 443 (HTTPS) es troben completament inactius en ambdós equips.

Respecte a la identificació del sistema operatiu, Nmap no va proporcionar una coincidència exacta (missatge: *"Too many fingerprints match"*) a causa de l'absència de ports oberts amb els quals interactuar. No obstant això, l'eina va detectar clarament que l'adreça MAC d'ambdós dispositius pertany al fabricant **Oracle VirtualBox virtual NIC**, evidenciant de forma inequívoca que es tracta d'entorns virtualitzats.

![Resultats de l'escaneig detallat amb Nmap](<pics/Captura de pantalla 2026-05-18 195827.png>)

---

## 📝 4. Conclusions i Aprenentatges Obtinguts

Al llarg d'aquesta pràctica d'exploració de xarxes amb Kali Linux, s'han consolidat diversos coneixements i competències tècniques fonamentals en l'àmbit de la ciberseguretat i l'anàlisi de xarxes:

### 🔑 Competències Desenvolupades

| Àrea de coneixement | Aprenentatge clau |
|---------------------|-------------------|
| **Configuració de xarxa** | Comprensió de la importància de la correcta configuració de l'adaptador en mode pont perquè la VM comparteixi la subxarxa de l'amfitrió físic |
| **Reconeixement actiu** | Domini de Netdiscover en mode actiu per al descobriment ràpid d'equips mitjançant peticions ARP, identificant adreces IP, MAC i fabricants |
| **Reconeixement passiu** | Aplicació de tècniques d'escolta passiva per obtenir informació sense generar trànsit propi, essencial per a operacions de reconeixement furtiu |
| **Anàlisi comparatiu** | Capacitat per avaluar els avantatges i inconvenients de cada mètode de sondeig segons el context operatiu |
| **Escaneig amb Nmap** | Ús de Nmap per al descobriment d'amfitrions (`-sn`) i l'anàlisi detallat de sistemes (`-O -sV`), interpretant correctament els resultats |
| **Interpretació de resultats** | Comprensió d'estats de ports (obert, tancat, filtrat) i limitacions en la detecció de sistemes operatius quan no hi ha serveis actius |
| **Identificació de virtualització** | Reconeixement d'entorns virtualitzats a través de l'anàlisi d'adreces MAC i fabricants d'interfícies de xarxa |

### 🎯 Reflexió Final

Aquesta pràctica ha permès comprendre de primera mà la importància de les **fases inicials de reconeixement** en qualsevol auditoria de seguretat. L'elecció entre tècniques actives i passives no és merament tècnica, sinó que respon a una decisió estratègica: el sondeig actiu prioritza la velocitat i l'exhaustivitat, mentre que el passiu maximitza la stealth i evita la detecció per sistemes IDS/IPS.

Així mateix, els resultats obtinguts amb Nmap evidencien que l'absència de ports oberts no implica necessàriament una xarxa segura, sinó que pot deure's a configuracions de tallafocs o a que els serveis escolten en ports no estàndard. La capacitat d'interpretar correctament els resultats de l'escaneig —incloent-hi les limitacions com la impossibilitat d'identificar el SO per falta d'empremtes actives— constitueix una habilitat crítica per a qualsevol professional de la ciberseguretat.

Finalment, la detecció d'entorns virtualitzats a través de l'anàlisi d'adreces MAC demostra que fins i tot la informació aparentment trivial pot revelar detalls significatius sobre la infraestructura objectiu, reforçant la necessitat d'una metodologia de reconeixement sistemàtica i minuciosa.

---

> 📋 **Document tècnic** | Exploració de Xarxes amb Kali Linux | Mòdul 0226 Seguretat Informàtica  
> [← Tornar a l'índex](README.md)
