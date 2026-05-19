# Guia de Configuració: Servidor FTP (vsftpd)

## 1. Configuració Inicial i Instal·lació

Per a aquesta activitat, s'utilitza un entorn amb Ubuntu Server configurat amb dues interfícies de xarxa: ``NAT`` per a la sortida a Internet i ``Host-only`` per permetre la comunicació amb la màquina física.

Abans d'instal·lar alguna cosa recordeu actualitzar si no sabeu com actualitzar aquest és la comanda.

```
Sudo apt update && sudo apt upgrade -y
```

Instal·lació: Executa la següent comanda com a root per instal·lar el servei: 

```
apt install vsftpd
```

> Si no saps com posar-te en root és 'Sudo su'

## 2. Preparació del Sistema de Fitxers (Directoris)

### A. Estructura per a l'accés Anònim

L'accés anònim permet que qualsevol usuari es connecti sense necessitat d'un compte personal.

Configurarem l'arxiu ``vsftpd.conf``

<img width="1105" height="534" alt="captura vsft" src="https://github.com/user-attachments/assets/e9330064-9e4e-421d-a464-99078d39ed94" />


Un cop dins l'arxiu Cal buscar la línia ``anonymous_enable=NO`` i canviar-la a ``YES``, reiniciant el servei després.


> Recordeu reniciar el servei una vegada canviat.

El directori de treball per defecte és ``/srv/ftp``. Crearem una estructura organitzada:

- Crear carpetes: ``mkdir -p /srv/ftp/pub/files /srv/ftp/pub/music /srv/ftp/pub/pics``.
- Afegir fitxers de prova: Pots crear fitxers buits per provar les descàrregues: ``touch /srv/ftp/pub/files/info.txt.``  ``touch /srv/ftp/pub/pics/sun.jpg``.

<img width="908" height="84" alt="image-2" src="https://github.com/user-attachments/assets/79e154dd-c76d-4dbc-983e-c280c0c1fcd5" />


Un cop acabats de crear els directoris i els recursos utilitzarem el tree per poder veure els directoris.

```
tree /srv/ftp
```

<img width="470" height="214" alt="image-3" src="https://github.com/user-attachments/assets/268ae0d9-eb5c-4f8a-ba38-20b9097ec88d" />


### B. Usuaris Locals

Crea els dos usuaris de prova que demana l'activitat per comprovar l'accés autenticat:

- ``adduser prova1``
- ``adduser prova2``

Nota: El sistema crearà automàticament les seves carpetes personals a ``/home/prova1`` i ``/home/prova2``.

## 3. Proves de funcionament

Fem:

```
ftp (i el teu ip del only host)
dir
```
<img width="575" height="293" alt="image-4" src="https://github.com/user-attachments/assets/5e542824-d368-4a55-afc7-711e080491be" />


```
get /srv/ftp/pub/pics/sun.jpg
```
<img width="646" height="96" alt="image-5" src="https://github.com/user-attachments/assets/4d9f8936-5663-467a-bd0a-a805d9570a0e" />


> Si intentes pujar un fitxer (put), rebràs un error 550 Permission denied. Per permetre-ho, caldria activar anon_upload_enable=YES.

## 4. FTP Autenticat i Permisos d'Escriptura

Configurarem el servidor perquè els usuaris reals puguin treballar a les seves carpetes.

1. Editar  l'arxiu ``/etc/vsftpd.conf``: Assegura't que aquestes línies estiguin actives (sense el # davant):

- ``local_enable=YES`` (permet entrada d'usuaris locals)
- ``write_enable=YES`` (permet pujar i esborrar fitxers)

<img width="767" height="420" alt="captura vsft" src="https://github.com/user-attachments/assets/a8ac18cd-cfb5-47c4-917f-11fa66c6e5f8" />


2. Reiniciar el servei: systemctl restart vsftpd.
3. Comprovació: L'usuari ``prova1`` ara pot fer un ``put`` d'un fitxer PDF a la seva carpeta ``/home/prova1`` amb èxit.



## 5. El Risc de Seguretat i l'Engabialament (chroot)

Sense més configuració, un usuari com ``prova1`` pot navegar per tot el servidor, incloent-hi carpetes del sistema com ``/etc``.

- Problema: L'usuari pot fer cd /etc i descarregar el fitxer passwd que conté la llista d'usuaris del sistema.
- Solució (Engabialament): Per evitar que l'usuari surti de la seva carpeta personal, afegeix al fitxer de configuració:
  - ``chroot_local_user=YES``
  - ``allow_writeable_chroot=YES``

## 6. Captura de Trànsit amb Wireshark

L'FTP és un protocol insegur perquè les transmissions no estan xifrades.

- Vulnerabilitat: Mitjançant un analitzador de trànsit com Wireshark, es pot realitzar un atac Man-in-the-Middle.
- Evidència: A les captures es veu clarament com el nom d'usuari i la contrasenya viatgen en text pla.
<img width="820" height="398" alt="captuura" src="https://github.com/user-attachments/assets/2666a1bc-db58-4c31-941a-7d4c9696bba1" />



---

[Tornar enrere](README.md)
