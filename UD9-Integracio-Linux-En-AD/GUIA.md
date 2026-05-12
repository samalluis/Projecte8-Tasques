# 🐧 Guia d'Integració de Linux (Zorin OS) en Active Directory (AD)

Aquesta guia detalla els passos necessaris per integrar equips amb sistema operatiu Linux (específicament Zorin OS) dins d'un domini de Windows Server Active Directory.

---

## 🛠️ Preparació de l'Entorn

Abans de començar, assegura't de disposar dels següents elements configurats en el teu entorn de virtualització:

* **Màquines Virtuals Linux:** Una o dues VM amb **Zorin OS** (configurades en xarxa NAT).
* **Controlador de Domini:** Una VM amb **Windows Server 2022** (configurada en xarxa NAT).

---

## 🚀 OPCIÓ 1: Integració durant el procés d'instal·lació del sistema

> 💡 **Descripció:** Aquesta opció és ideal si estàs instal·lant el sistema operatiu Zorin OS des de zero i vols afegir-lo al Directori Actiu directament en l'assistent d'instal·lació.

### 1. Mode Prova (Try Zorin OS)
En comptes d'escollir l'opció directa d'instal·lació, seleccionarem **Try Zorin OS** per poder configurar la xarxa prèviament i garantir la connectivitat amb l'AD.

![Try Zorin OS](pics/Captura%20de%20pantalla%202026-05-12%20152527.png)

### 2. Configuració de la Xarxa
Deixem el protocol **DHCP activat**, però modifiquem el servidor **DNS** per introduir l'adreça IP del nostre controlador de domini.

![Configuració Xarxa DNS](pics/Captura%20de%20pantalla%202026-05-12%20152729.png)

### 3. Execució de la Instal·lació
Un cop configurada correctament la xarxa amb el DNS del domini, procedim a executar l'instal·lador de Zorin OS.

![Instal·lar Zorin OS](pics/Captura%20de%20pantalla%202026-05-12%20153414.png)

Selecciona l'idioma que prefereixis per al sistema (es recomana l'idioma castellà).

![Selecció d'Idioma](pics/Captura%20de%20pantalla%202026-05-12%20153446.png)

### 4. Creació de l'Usuari Local i Unió al Domini
És obligatori crear un usuari local per a l'administració de la màquina. Configura els paràmetres habituals, però és molt important activar la casella **"Utilizar Active Directory"**.

![Activar Active Directory](pics/Captura%20de%20pantalla%202026-05-12%20160425.png)

### 5. Configurar els Paràmetres de l'AD
Introdueix el nom del domini del servidor i el compte de l'administrador corresponent.

> [!IMPORTANT]
> Revisa i valida de forma obligatòria que hi hagi connexió amb el domini fent clic al botó de comprovació abans de continuar.

![Configuració Dominis](pics/Captura%20de%20pantalla%202026-05-12%20160559.png)

### 6. Finalització del Procés
En completar la instal·lació, reiniciem l'equip i verifiquem des de les eines de Windows Server que l'equip Linux s'ha afegit correctament.

![Verificació en Windows Server](pics/Captura%20de%20pantalla%202026-05-12%20171211.png)

### 7. Inici de Sessió i Comprovació de Permisos
Ara ja podem iniciar sessió utilitzant les credencials de l'usuari administrador del servidor (en aquest cas, "administrator").

> [!NOTE]
> Recorda introduir l'usuari utilitzant la sintaxi de domini: `domini\administrador`

![Login Domini](pics/Captura%20de%20pantalla%202026-05-12%20171320.png)

Finalment, comprovem i verifiquem que aquest usuari administrador del domini encara **no té permisos de sudoer** dins de la màquina local Linux.

![Comprovació Sudoer](pics/Captura%20de%20pantalla%202026-05-12%20171447.png)

---

## ⚙️ OPCIÓ 2: Equip Zorin OS ja instal·lat prèviament

> 💡 **Descripció:** Fes servir aquest mètode si l'equip amb Zorin OS ja està completament instal·lat i operatiu, i el vols unir a l'Active Directory a posteriori de forma manual.

### 1. Configuració de l'Estructura a l'AD
Des del controlador de domini Windows, crearem una Unitat Organitzativa (OU) anomenada **linux**. Dins d'aquesta OU, crearem un grup anomenat **linuxadmins** i hi afegirem com a mínim un usuari.

![Estructura AD](pics/Captura%20de%20pantalla%202026-05-12%20172540.png)

### 2. Configuració de la Xarxa al Client
Igual que en el mètode anterior, mantindrem el **DHCP activat** però editarem la configuració de xarxa per apuntar el **DNS** cap a la IP del nostre controlador de domini.

![Configuració DNS Client](pics/Captura%20de%20pantalla%202026-05-12%20152729.png)

### 3. Instal·lació del Servei SSSD i Eines Associades
El servei **SSSD** (*System Security Services Daemon*) és el component encarregat de gestionar l'autenticació amb directoris remots. Obrirem un terminal per instal·lar tots els paquets i dependències necessàries.

![Instal·lació de paquets SSSD](pics/Captura%20de%20pantalla%202026-05-12%20173153.png)

### 4. Configuració del FQDN (Fully Qualified Domain Name)
Modificarem el nom d'host de la màquina local client perquè el seu nom complet (FQDN) formi part del domini de l'Active Directory al qual es vol connectar.

![Configuració FQDN](pics/Captura%20de%20pantalla%202026-05-12%20173423.png)

### 5. Connexió i Unió al Domini AD
Abans de procedir, comprovem que tenim visibilitat i resposta de xarxa amb el servidor de l'Active Directory.

![Comprovació Connexió AD](pics/Captura%20de%20pantalla%202026-05-12%20173602.png)

Un cop confirmada la connectivitat, executem la comanda d'unió al domini. El sistema ens sol·licitarà l'usuari administrador del directori actiu i la seva contrasenya corresponent.

![Unió al Domini existint](pics/Captura%20de%20pantalla%202026-05-12%20173656.png)

### 6. Verificació al Servidor
Comprovem que a la secció **Computers** de les eines d'administració de l'Active Directory hi apareix registrat correctament el nostre client Linux.

![Client Linux a Computers](pics/Captura%20de%20pantalla%202026-05-12%20173737.png)

---

## 🛠️ Configuracions Finals Post-Integració

Afegeix les següents directives de configuració per assegurar-te que el comportament del sistema és l'adequat per als usuaris del domini.

### 1. Creació Automàtica de la Carpeta Personal (Home)
Apliquem la següent configuració al sistema per garantir que, quan un usuari del Directori Actiu iniciï sessió al client Zorin per primera vegada, es creï automàticament el seu directori personal d'usuari (`/home/`).

![Configuració pam_mkhomedir](pics/Captura%20de%20pantalla%202026-05-12%20180333.png)

#### Inici de sessió de prova
Reiniciem el client i provem d'accedir-hi utilitzant les credencials de l'usuari del domini que hem ubicat prèviament a la OU Linux.

![Login Usuari Domini](pics/Captura%20de%20pantalla%202026-05-12%20174124.png)

Verifiquem visualment o per terminal que la seva carpeta de perfil personal s'ha generat correctament en el sistema de fitxers.

![Carpeta Home creada](pics/Captura%20de%20pantalla%202026-05-12%20180423.png)

### 2. Assignació de Permisos d'Administrador (Sudoers)
Com a darrer pas d'administració, configurarem el fitxer de sudoers per permetre que determinats usuaris o grups del domini tinguin privilegis d'administració local a la màquina.

Afegirem l'usuari `administrator` de manera directa i també el grup corporatiu `linuxadmins` definit a l'AD.

![Configuració fitxer Sudoers](pics/Captura%20de%20pantalla%202026-05-12%20182254.png)

> [!WARNING]
> Recorda que per fer referència a grups de seguretat de l'AD dins del fitxer de configuració de Linux, és obligatori col·locar el símbol **`%`** just davant del nom del grup.

### 3. Control d'Accés d'Usuaris
Per defecte, qualsevol usuari registrat correctament dins del domini té permís per iniciar sessió en aquest equip Ubuntu/Zorin. Si es vol acotar la seguretat, podem limitar l'accés global.

* **Per denegar o permetre l'accés global en l'àmbit de sistema:**

![Filtre Accés Global](pics/Captura%20de%20pantalla%202026-05-12%20183755.png)

* **Per restringir i permetre l'accés exclusivament a un grup concret:**

![Filtre per Grup AD](pics/Captura%20de%20pantalla%202026-05-12%20183902.png)

### 4. Accés a Unitats de Xarxa Compartides
Finalment, si els usuaris necessiten interactuar i muntar recursos compartits allotjats a l'Active Directory, farem servir el protocol Samba integrat directament amb el navegador de fitxers de Zorin.

En el cas de Zorin OS, caldrà assegurar-se primer d'instal·lar el paquet de xarxa requerit:

![Instal·lació client Samba](pics/Captura%20de%20pantalla%202026-05-12%20183955.png)

Un cop el paquet estigui al sistema, només caldrà obrir l'explorador de fitxers, utilitzar la barra d'adreces amb el format `smb://nom-servidor/` i connectar amb les credencials corresponents.

![Connexió de xarxa SMB](pics/Captura%20de%20pantalla%202026-05-12%20184710.png)

![Recurs compartit muntat](pics/Captura%20de%20pantalla%202026-05-12%20185714.png)