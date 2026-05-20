# Memòria Tècnica: Desplegament Multidomini i Seguretat per al Projecte Nexus

**Objectiu:** Infraestructura web per a l'agència de disseny i l'acadèmia Nexus.  
**Entorn:** Servidor Apache sobre Ubuntu Server.  

---

## 1. Introducció al Projecte

Aquesta memòria recull tots els passos que he seguit per muntar el servidor web de l'empresa Nexus. La idea era configurar un entorn professional que pogués allotjar dos portals diferents (una acadèmia i una agència de disseny) a la mateixa màquina, aplicant-hi certificats de seguretat SSL/TLS, personalització d'errors i optimització de velocitat amb HTTP/2.

---

## 2. Preparació i Instal·lació d'Apache

Per començar, vaig assegurar-me de tenir el sistema actualitzat i vaig instal·lar el servei web Apache:

```bash
sudo apt update && sudo apt install apache2
```
<img width="962" height="1025" alt="Captura de pantalla 2026-04-17 151239" src="https://github.com/user-attachments/assets/aa1b1150-12fd-4734-bdf2-56e5a9c27750" />

Un cop instal·lat, vaig comprovar dues coses fonamentals:
1. Que el servei estigués funcionant: `sudo systemctl status apache2`
2. Que l'usuari encarregat del web s'hagués creat correctament: `grep "www-data" /etc/passwd`

Aquesta comprovació de l'usuari `www-data` és important perquè és qui s'encarrega de llegir el contingut de `/var/www`. D'aquesta manera evitem donar permisos innecessaris a la resta del sistema.

<img width="934" height="482" alt="Captura de pantalla 2026-04-17 151309" src="https://github.com/user-attachments/assets/5c0db505-0e0b-4d39-9c3f-7ad4780d585b" />

---

## 3. Configuració Multidomini (VirtualHosts)

Com que havia de servir dos dominis (`projectenexus.test` i `academia.test`) amb una sola IP, vaig preparar l'estructura de directoris per a cadascun:

```bash
sudo mkdir -p /var/www/projectenexus /var/www/academia
```

A continuació, vaig agafar l'arxiu de configuració que ve per defecte i en vaig fer una còpia per a cada projecte:

```bash
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/projectenexus.conf
sudo cp /etc/apache2/sites-available/000-default.conf /etc/apache2/sites-available/academia.conf
```

Dins de cada arxiu (fent servir `nano`), vaig definir el `ServerName` corresponent (ex: `www.projectenexus.test`) i el `DocumentRoot` apuntant a la seva carpeta. 

Per aplicar els canvis, vaig habilitar els llocs i vaig reiniciar el servei:

```bash
sudo a2ensite projectenexus.conf
sudo a2ensite academia.conf
sudo systemctl reload apache2
```

Finalment, per poder fer proves locals des del meu Windows, vaig editar el fitxer `C:\Windows\System32\drivers\etc\hosts` i hi vaig afegir la IP de la meva màquina:

```text
192.168.56.202  www.projectenexus.test www.academia.test
```
<img width="957" height="989" alt="Captura de pantalla 2026-04-17 151924" src="https://github.com/user-attachments/assets/daffb867-7109-4837-80d9-ebc0f6c00347" />
<img width="954" height="1018" alt="Captura de pantalla 2026-04-17 151611" src="https://github.com/user-attachments/assets/b55b1cc0-563d-47fc-a4f1-795e46fd72da" />
<img width="941" height="170" alt="Captura de pantalla 2026-04-17 151438" src="https://github.com/user-attachments/assets/4f049fe1-1e1c-4333-9e63-dbbd031fc180" />
<img width="684" height="367" alt="Captura de pantalla 2026-04-17 152005" src="https://github.com/user-attachments/assets/25b21ea5-61be-45fe-ae68-0093a8a7a845" />
<img width="512" height="89" alt="Captura de pantalla 2026-04-17 151330" src="https://github.com/user-attachments/assets/0564cfdc-7207-418f-9745-c2729e9a3010" />


---

## 4. Pàgina d'Error (404) Personalitzada

Per donar una imatge més corporativa i evitar la pantalla genèrica d'Apache quan no es troba una pàgina, vaig dissenyar un error personalitzat.

Primer, vaig crear l'arxiu HTML:
```bash
sudo nano /var/www/academia/error404.html
```

Després, vaig anar a l'arxiu `/etc/apache2/sites-available/academia.conf` i vaig afegir-hi la següent línia:
```apache
ErrorDocument 404 /error404.html
```

Amb això, qualsevol URL inexistent redirigeix al meu disseny mantenint la identitat visual del projecte.

<img width="959" height="1077" alt="Captura de pantalla 2026-04-17 153200" src="https://github.com/user-attachments/assets/e93d842d-9f02-482c-a96a-4b6cedf15c4a" /><img width="1102" height="604" alt="Captura de pantalla 2026-04-17 154159" src="https://github.com/user-attachments/assets/8b795e94-f86c-413b-9111-db344d2728ee" />
<img width="1102" height="629" alt="Captura de pantalla 2026-04-17 154519" src="https://github.com/user-attachments/assets/dca22dc2-2cc9-4dc4-b0d4-26ed7b25b596" />
<img width="1103" height="616" alt="Captura de pantalla 2026-04-17 154600" src="https://github.com/user-attachments/assets/c76e6b5a-79d6-4825-899e-3fc55b88c6cf" />
<img width="1289" height="886" alt="Captura de pantalla 2026-04-17 154633" src="https://github.com/user-attachments/assets/fc24e5e8-caea-429d-b397-03298f88fa29" />
<img width="1275" height="876" alt="Captura de pantalla 2026-04-17 154420" src="https://github.com/user-attachments/assets/afd404d8-9127-43b9-9805-e3f6ee54d143" />
<img width="1268" height="800" alt="Captura de pantalla 2026-04-17 154206" src="https://github.com/user-attachments/assets/a1c5557e-51da-4ef2-ad51-20fa2b0d9c0b" />
<img width="1102" height="604" alt="Captura de pantalla 2026-04-17 154159" src="https://github.com/user-attachments/assets/33272118-1e12-49c2-bd10-4a6bcb6e895c" />


---

## 5. Implementació de Seguretat HTTPS (SSL)

Un dels requisits era que la navegació fos segura. El primer pas va ser activar el mòdul necessari a Apache:

```bash
sudo a2enmod ssl
```

Després, vaig crear les carpetes específiques per desar els certificats de forma endreçada:

```bash
sudo mkdir -p /var/www/academia/cert /var/www/academia/private
sudo mkdir -p /var/www/projectenexus/cert /var/www/projectenexus/private
ls /var/www/academia/
```

Vaig generar un certificat autosignat (de 2048 bits i vàlid per a 365 dies) amb aquesta comanda:

```bash
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /var/www/academia/private/academia.key -out /var/www/academia/cert/academia.crt
```

Per configurar els dominis pel port segur (443), vaig copiar la plantilla per defecte de SSL:

```bash
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/academia-ssl.conf
sudo cp /etc/apache2/sites-available/default-ssl.conf /etc/apache2/sites-available/projectenexus-ssl.conf
```

A dins de `academia-ssl.conf`, vaig configurar aquests paràmetres clau:
- `SSLEngine on`
- `SSLCertificateFile /var/www/academia/cert/academia.crt`
- `SSLCertificateKeyFile /var/www/academia/private/academia.key`

> **Nota important:** Vaig haver de comentar la línia `SSLCertificateChainFile` al fitxer de configuració, ja que estic fent servir un certificat autosignat i no tinc cap cadena de certificació externa.

Per rematar la seguretat, al VirtualHost del port 80 vaig forçar la redirecció cap a HTTPS afegint-hi:
```apache
Redirect permanent / https://www.academia.test/
```
<img width="954" height="1013" alt="Captura de pantalla 2026-04-17 155053" src="https://github.com/user-attachments/assets/f6e15818-5993-4d44-b3d0-b95075690497" />
<img width="949" height="136" alt="Captura de pantalla 2026-04-17 155152" src="https://github.com/user-attachments/assets/da654351-101d-493a-a789-fbfaae9c9e72" />
<img width="955" height="1025" alt="Captura de pantalla 2026-04-17 155405" src="https://github.com/user-attachments/assets/da12af85-04b0-4dfa-b104-d705e9f18082" />



---

## 6. Activació i Test de HTTP/2

Per millorar els temps de càrrega i la gestió de múltiples peticions simultànies, vaig habilitar el protocol HTTP/2.

Vaig activar el mòdul:
```bash
sudo a2enmod http2
```

I dins dels arxius de configuració del port 443 (VirtualHosts segurs) vaig afegir-hi:
```apache
Protocols h2 http/1.1
```

Per demostrar que estava funcionant correctament, un cop guardat tot i reiniciat l'Apache, vaig fer la prova amb la comanda:
```bash
curl -I --http2 -k https://www.academia.test
```
El resultat de la petició va retornar un `HTTP/2 200`, confirmant que l'optimització està completament activa i funcional.

<img width="798" height="143" alt="Captura de pantalla 2026-04-17 155624" src="https://github.com/user-attachments/assets/31abc4f7-50ec-4187-a469-acfe675df44f" /><img width="956" height="1079" alt="Captura de pantalla 2026-04-17 155535" src="https://github.com/user-attachments/assets/c8149746-89d9-4f48-933a-e85c713494bd" />
<img width="955" height="1025" alt="Captura de pantalla 2026-04-17 155808" src="https://github.com/user-attachments/assets/756ba8db-9145-4cf1-b2d0-4743a714184e" />


---

## 7. Conclusions

El projecte ha quedat muntat amb èxit i compleix tots els requisits. He pogut desplegar dos webs al mateix servidor, aplicant restriccions i mesures de seguretat (HTTPS) adequades, amb una pàgina d'error personalitzada i fent ús d'HTTP/2 per garantir un alt rendiment.

[Tornar enrere](README.md)
```
