# Guia d'Implementació: Servidor sFTP Segur i Usuaris "Engabiats"

## 1. Introducció al protocol sFTP

L'sFTP és un protocol de transferència de fitxers que es basa en SSH (Secure Shell) per realitzar connexions segures.

-  Seguretat: A diferència de l'FTP tradicional, xifra la connexió.
-  Port: Utilitza el port 22, el mateix que el servei SSH, per a tota la comunicació

## 2. Instal·lació del servei

Si no tenim el servidor SSH instal·lat prèviament, caldrà instal·lar-lo. Es pot fer servir el mateix servidor que s'hagi utilitzat en activitats anteriors o crear-ne un de nou.

Comanda d'instal·lació:

```bash
sudo apt install openssh-server -y
```

<img width="754" height="182" alt="image7" src="https://github.com/user-attachments/assets/8573f3cd-0504-42f8-94de-766f429bf5fc" />


## 3. El problema de seguretat (Sense configurar)

Abans d'aplicar restriccions, si ens connectem amb un usuari estàndard via sFTP, aquest té la capacitat de navegar per tot el sistema de fitxers (per exemple, accedir a /etc), la qual cosa suposa una fuga de seguretat.

Per solucionar-ho, hem d'"engabiar" els usuaris. Això es fa millor definint un grup d'usuaris específic i aplicant les restriccions al grup sencer, en lloc de fer-ho usuari per usuari.


## 4. Configuració pas a pas

### Pas 4.1: Creació de Grup i Usuari

Crearem un grup anomenat admins (o el nom que vulguis per als usuaris restringits) i un usuari de prova admin1 dins d'aquest grup.

```
# Crear el grup
sudo addgroup admins

# Crear l'usuari afegint-lo al grup admins (-G)
sudo useradd -G admins admin1

# Assignar contrasenya a l'usuari
sudo passwd admin1
```

<img width="493" height="186" alt="image9" src="https://github.com/user-attachments/assets/e40a8e75-247a-4cfb-a161-ebd138b74358" />

### Pas 4.2: Configuració del servei SSH (sshd_config)

Hem d'editar l'arxiu de configuració per indicar on quedarà engabiat el grup i quins permisos tindrà.

``Editar l'arxiu:``

```
sudo nano /etc/ssh/sshd_config
```


Afegir les següents línies al final de l'arxiu:

```
Match Group admins
    ChrootDirectory /var/data
    X11Forwarding no
    AllowTcpForwarding no
    PermitTTY no
    ForceCommand internal-sftp
```
> Hauràs de baixar abaix de tot

<img width="574" height="582" alt="image8" src="https://github.com/user-attachments/assets/c403677b-784a-4505-8c58-d53a9a345804" />


Explicació de les directives:

-  Match Group admins: Aplica les regles següents només als membres del grup admins.

-  ChrootDirectory /var/data: Defineix la carpeta "presó". L'usuari no podrà sortir (pujar de nivell) d'aquesta carpeta.

-  ForceCommand internal-sftp: Força l'ús del mode sFTP i impedeix que l'usuari iniciï una sessió de terminal remota (shell).

-  PermitTTY no, X11Forwarding no: Mesures addicionals per evitar l'accés a consola o redirecció d'aplicacions.
Recorda reiniciar el servei SSH després de desar els canvis (sudo service ssh restart o similar).


### Pas 4.3: Creació de carpetes i permisos (Punt Crític)

Perquè el ChrootDirectory funcioni, hi ha una regla de seguretat estricta: el directori arrel de la gàbia (/var/data) ha de ser propietat de root i ningú més pot tenir permisos d'escriptura

Per tant, l'estructura ha de ser:

1. Carpeta arrel (/var/data): Propietat de root (només lectura per als usuaris).
2. Carpeta de treball (/var/data/files): On el grup admins podrà pujar i baixar fitxers.

Comandes:
```
# 1. Crear la carpeta arrel:

sudo mkdir /var/data

# 2. Verifica que el propietari sigui root:root.

ls -l /var

# 3. Crear la subcarpeta de treball:

sudo mkdir /var/data/files

# 4. Assignar permisos al grup: Canviem el grup propietari de la subcarpeta a 
admins i li donem permisos d'escriptura i lectura.

sudo chown :admins /var/data/files
sudo chmod 2770 /var/data/files
```
<img width="1413" height="736" alt="captura 100" src="https://github.com/user-attachments/assets/1e57d251-4cc4-4b5a-8dfd-82ef6a8e0069" />

<img width="553" height="84" alt="image11 (1)" src="https://github.com/user-attachments/assets/5ea01de6-d1b2-4a28-b50f-c8e188ac4803" />


## 5. Verificació i Proves

Un cop configurat, cal realitzar les següents comprovacions:

### 1. Intentar connexió SSH (Terminal)

Intenta connectar-te per terminal: ssh admin1@IP_DEL_SERVIDOR.

>Resultat esperat: La connexió ha de fallar. Hauries de veure el missatge: "This service allows sftp connections only". Això confirma que no tenen accés a la línia de comandes.



### 2. Comprovar connexió sFTP

Intenta connectar-te per sFTP: sftp admin1@IP_DEL_SERVIDOR.

La connexió s'hauria d'establir correctament.

<img width="1996" height="512" alt="captura comprovar sftp" src="https://github.com/user-attachments/assets/83325d0d-d7b7-4a98-af08-0d80883f71b6" />

### 3. Verificar l'engabiat (Chroot)

Dins de la sessió sFTP, intenta sortir de la carpeta arrel:
```
sftp> cd /etc
```

>Resultat esperat: Error "No such file or directory" o "not found". L'usuari veu /var/data com si fos la seva arrel / i no pot veure res més del sistema.

### 4. Verificar transferència de fitxers

Entra a la carpeta on tens permisos (cd files) i prova les comandes:

```bash
# Per pujar un fitxer al servidor.
put A.txt.txt
# Per baixar un fitxer del servidor. Ambdós processos han de completar-se al 100% sense errors de permisos.
get secreto.txt
```
