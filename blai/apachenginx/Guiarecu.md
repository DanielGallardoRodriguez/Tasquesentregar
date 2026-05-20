# Pràctica Nginx – Migració del Projecte Nexus i Acadèmia

**Data**: 17/04/2026
**Sistema**: Ubuntu Server 24.04

---

## Objectiu

Passar el servidor web d'Apache a Nginx.
---

## Pas 1 – Instal·lar Nginx

```bash
sudo apt update && sudo apt install nginx -y
sudo systemctl status nginx
```
<img width="795" height="428" alt="Captura de pantalla 2026-04-20 154340" src="https://github.com/user-attachments/assets/8fc118cf-0b09-411a-b654-8391cded370e" />
<img width="949" height="444" alt="Captura de pantalla 2026-04-20 161127" src="https://github.com/user-attachments/assets/18d57c09-a79c-4e03-9c66-dc82f7c9be76" />
!

---

## Pas 2 – Preparar les carpetes dels webs

Cada projecte el seu lloc. He creat:


```bash
sudo mkdir -p /usr/share/nginx/academia
sudo mkdir -p /usr/share/nginx/projectenexus
```
<img width="942" height="128" alt="Captura de pantalla 2026-04-20 161428" src="https://github.com/user-attachments/assets/7b0f77cf-aad9-403e-afc3-0b03d563dd39" />

!

---

## Pas 3 – Configurar els virtual hosts

A Nginx se'n diuen Server Blocks. He duplicat el fitxer `default` i l'he enllaçat a `sites-enabled`:

```bash
cd /etc/nginx/sites-available/
sudo cp default academia
sudo cp default projectenexus
sudo ln -s /etc/nginx/sites-available/academia /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/projectenexus /etc/nginx/sites-enabled/
```

I a `nginx.conf` he pujat el hash bucket a 64 per si m'afegeixo més dominis endavant:

```
server_names_hash_bucket_size 64;
```
<img width="940" height="1018" alt="Captura de pantalla 2026-04-20 161619" src="https://github.com/user-attachments/assets/1c3511cb-88b3-466a-ab21-0f6657006a09" />
<img width="939" height="280" alt="Captura de pantalla 2026-04-20 161903" src="https://github.com/user-attachments/assets/7fad969e-76e0-4c7c-913b-93b234dfe9c6" />

---

## Pas 4 – Pàgina 404 pròpia

```nginx
error_page 404 /404.html;
location = /404.html {
    internal;
}
```

He posat `internal` perquè no es pugui obrir la pàgina 404 ficant la URL al navegador. Només la mostra Nginx quan realment no troba res.

<img width="945" height="982" alt="Captura de pantalla 2026-04-20 162028" src="https://github.com/user-attachments/assets/e8244bd2-6556-425a-a489-e54755b8b870" />


---

## Pas 5 – SSL

Certificat autofirmat amb RSA 2048 bits, vàlid 365 dies:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout .../private/academia.key \
  -out .../cert/academia.crt
```

<img width="572" height="141" alt="Captura de pantalla 2026-04-20 162054" src="https://github.com/user-attachments/assets/ea30941c-9824-4c62-b81e-19705e32d2bb" />

---

## Pas 6 – Protegir la carpeta private

```nginx
location /private {
    deny all;
    return 403;
}
```

<img width="834" height="39" alt="Captura de pantalla 2026-04-20 162238" src="https://github.com/user-attachments/assets/bd93317e-3f92-4e38-a443-8e651f7c0c79" />

---

## Pas 7 – Forçar HTTPS i activar HTTP/2

Al bloc de l'escolta del port 80 he posat la redirecció 301 cap al 443, i al del 443 hi he afegit `http2`:

```
listen 443 ssl http2;
```

Així tot va xifrat i més ràpid amb multiplexació.
<img width="943" height="802" alt="Captura de pantalla 2026-04-20 163343" src="https://github.com/user-attachments/assets/10eef206-f20c-46f1-b420-455b9941d2ae" />


---

## Pas 8 – Comprovar que funciona

| Comanda | Què fa |
|---|---|
| `sudo nginx -t` | Revisa que la config no té errors |
| Editar `/etc/hosts` | Fa que el domini apunti a la meva VM |
| `curl -I -k --http2 https://www.academia.test` | Confirma que HTTP/2 està actiu |

```bash
curl -I -k --http2 https://www.academia.test
```
