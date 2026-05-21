# Informe Tècnic: Implementació d'un Proxy Web amb Filtratge d'URL a IPFire

---

## 1. Presentació

Aquest document recull, de forma pràctica i detallada, tot el procés necessari per posar en marxa el servei de **Proxy Web (Squid)** i el seu complement de **Filtratge de Contingut (URL Filter)** en un entorn basat en el tallafocs IPFire.

La finalitat principal és aplicar un conjunt de polítiques de control sobre la navegació de la xarxa interna (interfície Green). Entre les funcionalitats que es cobriran hi figuren:

- Restricció d'accés per **categories temàtiques**.
- Administració de **llistes negres i blanques** de dominis.
- Bloqueig de **rutes (URLs) concretes** dins d'un domini.
- Filtratge mitjançant **expressions regulars** (paraules clau).
- Denegació d'accés basada en **franges horàries** i **rangs de xarxa**.

---

## 2. Preparació de l'Entorn

### 2.1. Posada en Marxa del Proxy a IPFire

Per defecte, IPFire no intercepta el trànsit web de la xarxa local. Abans de definir cap política de filtratge, cal activar manualment el servei de proxy.

**Passos a seguir:**

1. Obrir la consola web d'IPFire i dirigir-se a **Red → Web Proxy**.
2. A l'apartat *Configuraciones comunes*, marcar l'opció **Activado en Green** per posar en funcionament el proxy a la xarxa interna.
3. Comprovar que el **port del proxy** es manté al seu valor per defecte: `800`.
4. Dins del mateix panell, buscar el bloc **URL filter** i assegurar-se d'activar la seva casella. Sense aquest pas, el proxy funcionarà però **no aplicarà cap regla de filtratge**.
5. Guardar els canvis. L'indicador d'estat del servei hauria de passar a mostrar-se com a actiu.


---

### 2.2. Ajustos de Proxy al Client

En aquest escenari s'utilitza un model de **proxy explícit (no transparent)**, cosa que implica que cada equip client ha de ser configurat manualment perquè dirigeixi el seu trànsit cap al proxy.

**Procediment:**

1. A la màquina client, accedir a la configuració de xarxa del navegador o del sistema operatiu.
2. Establir el mode de proxy com a **Manual**.
3. Indicar la IP de la interfície Green del servidor IPFire (`192.169.6.254`) i el port `800` tant per a HTTP com per a HTTPS.

Amb aquesta configuració, qualsevol petició web del navegador passarà obligatòriament pel tallafocs.


---

## 3. Configuració de les Polítiques de Filtratge

A partir d'aquest punt, totes les accions es realitzen des del menú **Red → Filtro de contenido** d'IPFire.

---

### 🔹 Activitat 1 — Descàrrega i Instal·lació de les Llistes Negres

El sistema de filtratge per categories necessita una base de dades que classifiqui els llocs web per temàtica. Sense aquesta descàrrega prèvia, les categories no estaran disponibles.

1. Anar a la secció *Mantenimiento de URL Filter*.
2. Dins del bloc *Actualización automática de lista negra*, habilitar l'opció **Activar actualización automática**.
3. Configurar la periodicitat a **mensualmente** al desplegable corresponent.
4. Escollir la font **Univ. Toulouse** com a proveïdor de la base de dades.
5. Prémer *Guardar configuraciones de actualización* i seguidament clicar **Actualizar ahora** per forçar la primera descàrrega.

<img width="870" height="367" alt="Captura de pantalla 2026-05-20 203905" src="https://github.com/user-attachments/assets/bcb839ad-5d86-486f-88d9-aef0628bd5ff" />

---

### 🔹 Activitat 2 — Bloqueig de Categories: Bancs i Ràdio

Amb les llistes ja carregades, és possible restringir l'accés a categories senceres de contingut.

1. Dins de *Configuraciones de URL filter*, desplegar l'àrea de **Categorías bloqueadas**.
2. Localitzar i activar les caselles de les categories **bank** i **radio**.
3. Guardar i reiniciar el servei des del final de la pàgina.

<img width="1100" height="770" alt="Captura de pantalla 2026-05-20 204256" src="https://github.com/user-attachments/assets/0192ccf6-10d8-4927-9280-ed7f202db330" />


#### Comprovació i Explicació Tècnica

Si des del client intentem visitar un domini bancari com `ing.es`, el navegador mostrarà l'error **`ERR_TUNNEL_CONNECTION_FAILED`** en lloc de la típica pàgina vermella de bloqueig d'IPFire.

**Per què passa això?** El trànsit HTTPS viatja xifrat. El proxy només pot llegir el nom del domini gràcies al camp **SNI (Server Name Indication)** de la negociació TLS. En identificar que el domini pertany a una categoria bloquejada, l'única opció que té és **tallar el túnel de connexió** de forma abrupta, ja que no pot injectar cap pàgina HTML d'avís dins d'un canal encriptat.

<img width="1171" height="860" alt="Captura de pantalla 2026-05-20 210038" src="https://github.com/user-attachments/assets/3a30b27f-535a-49d1-9a7b-e87c53da64da" />

<img width="1172" height="852" alt="Captura de pantalla 2026-05-20 210110" src="https://github.com/user-attachments/assets/82ad214a-6b7f-496f-ab91-0268e9751d6c" />

---

### 🔹 Activitat 3 — Denegació de Dominis Individuals

Independentment de les categories, podem bloquejar dominis específics de manera manual.

1. Dirigir-se a l'apartat **Lista Negra personalizada**.
2. Al camp de text de l'esquerra (*Dominios bloqueados*), escriure els dominis desitjats, un per línia:
   ```
   elnacional.cat
   tecnocampus.cat
   ```
3. Activar la casella **Activar Lista Negra personalizada** per fer efectiva la regla.

<img width="1274" height="880" alt="Captura de pantalla 2026-05-20 210300" src="https://github.com/user-attachments/assets/dc7d59fc-5441-4cb1-bec7-23450ee03a77" />
<img width="1135" height="791" alt="Captura de pantalla 2026-05-20 210347" src="https://github.com/user-attachments/assets/e7920439-f741-4b22-880c-919a6599dd91" />
<img width="1195" height="832" alt="Captura de pantalla 2026-05-20 210401" src="https://github.com/user-attachments/assets/ec90f9cd-335c-4732-a7b1-c7b4e7c31b8d" />


---

### 🔹 Activitat 4 — Bloqueig d'una Ruta Concreta Dins d'un Domini

De vegades no volem bloquejar un domini sencer, sinó únicament un directori o ruta específica. Això **només és viable amb trànsit HTTP**, ja que amb HTTPS el proxy no pot veure la ruta completa (queda oculta pel xifratge).

1. A la mateixa secció de *Lista Negra personalizada*, al camp dret (*URLs bloqueadas*), introduir la ruta:
   ```
   www.textfiles.com/jason/
   ```
2. Desar la configuració.

#### Verificació

| Prova | Resultat |
|---|---|
| Accedir a `textfiles.com` (arrel) | ✅ La pàgina carrega amb normalitat |
| Accedir a `textfiles.com/jason/` | ❌ El proxy intercepta la petició HTTP, llegeix la ruta i mostra **ACCESS DENIED** |

<img width="966" height="838" alt="Captura de pantalla 2026-05-21 172534" src="https://github.com/user-attachments/assets/fa4bf76f-fed5-43c0-986b-b609795445f5" />
<img width="957" height="822" alt="Captura de pantalla 2026-05-21 172512" src="https://github.com/user-attachments/assets/43bc557a-5b22-4172-8d7c-250d81147ff6" />


---

### 🔹 Activitat 5 — Filtratge per Paraula Clau amb Excepció via Llista Blanca

Aquesta pràctica demostra la **jerarquia de prioritats** del filtre: la Llista Blanca sempre té preferència sobre qualsevol altra restricció.

**Configuració en dos passos:**

1. **Restricció:** A l'apartat *Lista de frases personalizadas*, introduir el terme `anime` i activar la casella corresponent. Això bloqueja qualsevol URL que contingui aquesta paraula.
2. **Excepció:** A la secció *Lista Blanca personalizada* → *Dominios permitidos*, afegir el domini `animenewsnetwork.com` i habilitar la casella.

<img width="918" height="355" alt="Captura de pantalla 2026-05-21 172936" src="https://github.com/user-attachments/assets/d343e9fc-f532-497e-a04f-53bad8441851" />


#### Proves de Funcionament

| Cas | Domini | Resultat | Motiu |
|---|---|---|---|
| Bloqueig | `www4.animeflv.net` | ❌ Connexió tallada | El proxy detecta "anime" al SNI i interromp el túnel TLS |
| Excepció | `animenewsnetwork.com` | ✅ Accés complet | El domini consta a la Llista Blanca, que preval sobre el filtre |

<img width="946" height="894" alt="Captura de pantalla 2026-05-21 172958" src="https://github.com/user-attachments/assets/57aa0964-c0b6-4481-8bd7-db01217f5917" />
<img width="970" height="850" alt="Captura de pantalla 2026-05-21 173021" src="https://github.com/user-attachments/assets/1cae7b0f-b773-427b-9995-9322c19d4af7" />


---

### 🔹 Activitat 6 — Denegació Total per Horari i Subxarxa

Finalment, es configura una regla que impedeix completament la navegació a una subxarxa concreta dins d'una franja horària definida.

1. A l'àrea *Añadir nueva regla de restricción de tiempo*, seleccionar **tots els dies** de la setmana (Dilluns a Diumenge).
2. Establir l'horari de **00:00 a 24:00** (bloqueig permanent).
3. Completar els camps restants:

| Paràmetre | Valor |
|---|---|
| **Host o red(es) de origen** | `192.169.11.0/24` |
| **Destino** | `cualquier` |
| **Acceso** | `bloquear` |

4. Prémer **Agregar** per activar la regla.

<img width="797" height="480" alt="Captura de pantalla 2026-05-21 173753" src="https://github.com/user-attachments/assets/cfe9686f-6d73-4188-bc06-40d40130b624" />


#### Verificació

En provar d'accedir des del client a qualsevol lloc web (per exemple, `httpforever.com` via HTTP), IPFire retorna directament la pantalla **ACCESS DENIED**, confirmant que l'equip es troba dins de la subxarxa i la franja horària afectades per la regla.

<img width="1307" height="852" alt="Captura de pantalla 2026-05-21 173857" src="https://github.com/user-attachments/assets/aca00a17-ca6e-4bbf-8681-e520c2763776" />

---

## 4. Resum de Resultats

| # | Política aplicada | Resultat obtingut |
|---|---|---|
| 1 | Descàrrega de blacklists (Univ. Toulouse) | Base de dades de categories disponible |
| 2 | Bloqueig categories *bank* i *radio* | Túnel TLS tallat en accedir a dominis bancaris/ràdio |
| 3 | Llista negra manual de dominis | Dominis individuals denegats correctament |
| 4 | Bloqueig de ruta específica (HTTP) | Ruta bloquejada, resta del domini accessible |
| 5 | Paraula clau + excepció Llista Blanca | Filtre actiu amb excepció funcional |
| 6 | Restricció horària per subxarxa | Denegació total dins la franja i rang configurats |

---

[Tornar enrere](README.md)
