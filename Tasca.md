# 📘 Guia de Pràctiques: Configuració de SAMBA

> **Assignatura:** Administració de Sistemes Operatius en Xarxa
> **Pràctica:** Configuració d'un servidor SAMBA
> **Objectiu:** Compartir recursos entre sistemes Linux i Windows mitjançant el protocol SMB/CIFS.
> **Entorn:** Zorin OS (Servidor) + Windows 11 (Client)

---

## 📋 Índex

1. [Configuració Inicial de les Màquines](#1-configuració-inicial-de-les-màquines)
2. [Instal·lació i Configuració de SAMBA](#2-exercici-1-installació-i-configuració-de-samba)
3. [Accés a Carpeta Pública en Mode Anònim](#3-exercici-2-accés-a-carpeta-pública-en-mode-anònim)
4. [Carpetes Personals](#4-exercici-3-carpetes-personals)
5. [Unitats Compartides amb Permisos Específics](#5-exercici-4-unitats-compartides-amb-permisos-específics)
6. [Samba des de Windows 11 a Linux](#6-exercici-5-samba-des-de-windows-11-a-linux)
7. [Conclusions](#7-conclusions)

---

## 1. Configuració Inicial de les Màquines

### 🎯 Objectiu

Preparar les màquines virtuals amb els recursos necessaris i actualitzar el sistema abans de començar la instal·lació de SAMBA.

| Màquina | Rol | RAM | CPU |
|---|---|---|---|
| 🐧 Zorin OS | Servidor Samba | 4 GB | 2 |
| 🪟 Windows 11 | Client Samba | 8 GB | 4 |

> ℹ️ Assegureu-vos que ambdues màquines es troben a la mateixa xarxa i que es poden comunicar entre elles (podeu verificar-ho amb `ping`).

---

### 🔧 Actualització del sistema

Abans de res, actualitzeu el sistema per garantir compatibilitat i seguretat:

```bash
sudo apt update && sudo apt upgrade -y
```

| Comanda | Funció |
|---|---|
| `sudo` | Executa la comanda com a administrador |
| `apt update` | Actualitza la llista de paquets disponibles |
| `apt upgrade -y` | Instal·la les versions més recents dels paquets |

> 💡 Si el sistema demana reiniciar després de l'actualització, feu-ho abans de continuar.

---

## 2. Exercici 1: Instal·lació i Configuració de SAMBA

### 2.1 Instal·lació del paquet

```bash
sudo apt install samba -y
```

| Element | Descripció |
|---|---|
| `apt install` | Instal·la paquets al sistema |
| `samba` | Implementació del protocol SMB/CIFS per a Linux |

---


> <img width="657" height="257" alt="Captura de pantalla 2026-05-13 184854" src="https://github.com/user-attachments/assets/35c083bb-c820-4346-8faa-d612bc84659f" />

---

### 2.2 Gestió del servei

Reinicieu i verifiqueu l'estat del servei:

```bash
sudo systemctl restart smbd
sudo systemctl status smbd
```

> ✅ El servei ha d'aparèixer com **active (running)** en color verd.



### 2.3 Còpia de seguretat del fitxer de configuració

> ⚠️ **Pas obligatori.** Sempre feu una còpia de seguretat abans de modificar fitxers de configuració del sistema.

```bash
sudo cp /etc/samba/smb.conf /etc/samba/smb.conf.bak
```

Podeu verificar que s'ha creat correctament amb:

```bash
ls -l /etc/samba/
```

---


<img width="539" height="59" alt="Captura de pantalla 2026-05-13 193837" src="https://github.com/user-attachments/assets/8667da3f-957c-4833-9f35-4e2d591f0d15" />



```

---

### 2.4 Creació dels directoris compartits

```bash
sudo mkdir -p /srv/samba/publica
sudo mkdir -p /srv/samba/compartida
```
<img width="547" height="91" alt="Captura de pantalla 2026-05-18 165218" src="https://github.com/user-attachments/assets/6e0dae44-c5e9-47f1-9332-d76cff84c82d" />

Verifiqueu que s'han creat correctament:

```bash
ls -l /srv/samba
```

---

### 2.5 Propietaris i permisos dels directoris

Assigneu el grup `sambashare` com a propietari:

```bash
sudo chown root:sambashare /srv/samba/publica
sudo chown root:sambashare /srv/samba/compartida
```

Assigneu els permisos `770`:

```bash
sudo chmod 770 /srv/samba/publica
sudo chmod 770 /srv/samba/compartida
```
<img width="549" height="70" alt="Captura de pantalla 2026-05-18 165502" src="https://github.com/user-attachments/assets/174adbc5-c4cc-4eea-a913-3c6b6f7777db" />

**Significat del permís `770`:**

| Qui | Valor | Permisos |
|---|---|---|
| Propietari (root) | 7 | Lectura + Escriptura + Execució |
| Grup (sambashare) | 7 | Lectura + Escriptura + Execució |
| Altres | 0 | Cap permís |

> 🔐 Els usuaris que no pertanyin al grup `sambashare` no podran accedir als directoris.

---

### 2.6 Creació d'usuaris del sistema

Creem tres usuaris sense accés a shell (únicament per a SAMBA):

```bash
sudo useradd -m -s /sbin/nologin -G sambashare samba1
sudo useradd -m -s /sbin/nologin -G sambashare samba2
sudo useradd -m -s /sbin/nologin -G sambashare samba3
```

| Paràmetre | Significat |
|---|---|
| `-m` | Crea el directori personal de l'usuari |
| `-s /sbin/nologin` | Impedeix l'accés interactiu per shell |
| `-G sambashare` | Afegeix l'usuari al grup `sambashare` |

---



><img width="654" height="137" alt="Captura de pantalla 2026-05-18 170620" src="https://github.com/user-attachments/assets/e5a0c092-e8b5-4f2c-adf0-87df7300baec" />


### 2.7 Assignació de contrasenyes SAMBA

Els usuaris del sistema necessiten una contrasenya específica per a SAMBA, independent de la del sistema:

```bash
sudo smbpasswd -a samba1
sudo smbpasswd -a samba2
sudo smbpasswd -a samba3
```

> 💡 Introduïu una contrasenya segura per a cada usuari quan us ho demani. Recordeu-les per a les proves posteriors.

---



> <img width="450" height="274" alt="Captura de pantalla 2026-05-19 160552" src="https://github.com/user-attachments/assets/deac715e-b526-45e0-9e77-1de8391ba893" />

---

## 3. Exercici 2: Accés a Carpeta Pública en Mode Anònim

### Objectiu

Configurar un recurs accessible per a qualsevol usuari de la xarxa sense necessitat d'autenticació, però només amb permís de lectura.

---

### Configuració del fitxer `smb.conf`

Editeu el fitxer de configuració:

```bash
sudo nano /etc/samba/smb.conf
```
<img width="629" height="44" alt="Captura de pantalla 2026-05-19 161203" src="https://github.com/user-attachments/assets/fa813ebe-0bf8-4349-b793-5d3db0dc2f11" />

Afegiu o modifiqueu les seccions següents:

```ini
[global]
   workgroup = WORKGROUP
   server string = Servidor Samba
   security = user
   map to guest = bad user

[publica]
   comment = Carpeta pública de només lectura
   path = /srv/samba/publica
   browsable = yes
   guest ok = yes
   read only = yes
```

> ℹ️ `map to guest = bad user` permet que els usuaris no reconeguts accedeixin com a convidats.
<img width="596" height="417" alt="Captura de pantalla 2026-05-19 162026" src="https://github.com/user-attachments/assets/9cea583d-dff0-4fe3-995a-7344f34248db" />
<img width="447" height="262" alt="Captura de pantalla 2026-05-19 161528" src="https://github.com/user-attachments/assets/ded6b281-33a8-4ab9-b22d-dcf76155eb53" />

---

### Validació de la configuració

Comproveu que no hi ha errors de sintaxi:

```bash
sudo testparm
```

---

#### 📸 Evidència 9 — Validació amb `testparm`

> **Captureu la sortida de `testparm` sense errors. Ha de mostrar "Loaded services file OK".**


```<img width="614" height="578" alt="Captura de pantalla 2026-05-19 162611" src="https://github.com/user-attachments/assets/d554168a-2c41-42ef-a360-13535dc512a5" />

````

### Creació d'un fitxer de prova i accés des de Windows

Creeu un fitxer de prova al servidor:

```bash
sudo touch /srv/samba/publica/test1.txt
```

Des de Windows 11, obriu l'Explorador de fitxers i accediu a:

```
\\IP_DEL_SERVIDOR\publica
```

**Resultat esperat:**

| Acció | Resultat |
|---|---|
| ✅ Visualitzar fitxers | Permès |
| ❌ Crear fitxers | Denegat |
| ❌ Modificar fitxers | Denegat |
| ❌ Eliminar fitxers | Denegat |

---

#### 📸 Evidència

<img width="497" height="54" alt="Captura de pantalla 2026-05-19 162651" src="https://github.com/user-attachments/assets/3339e16d-c115-41f1-91a1-439c82c503e1" />



<img width="1300" height="800" alt="captura" src="https://github.com/user-attachments/assets/14e61662-0067-4443-8603-175e78d19827" />

<img width="1714" height="624" alt="captura2" src="https://github.com/user-attachments/assets/774f0c3d-f35e-43b8-88e9-2b947a1f11eb" />
<img width="1180" height="896" alt="captura3" src="https://github.com/user-attachments/assets/d922afa9-f37d-4922-a2f2-87894c6f0be5" />
<img width="1542" height="672" alt="captura 4" src="https://github.com/user-attachments/assets/066c321d-92e4-4f34-8884-6f00780fef31" />



