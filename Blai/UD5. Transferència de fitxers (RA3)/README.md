# T03: Serveis de Transferència de Fitxers 🚀

Benvinguts al mòdul de configuració de serveis de transferència de fitxers. Després d'haver dominat la infraestructura base a **EverPia** (DHCP, DNS i administració remota), és hora de fer un pas més enllà en la gestió de dades i la seguretat en xarxa.

L'objectiu d'aquesta unitat és dominar els protocols que permeten el moviment d'informació entre sistemes, entenent la diferència crítica entre la funcionalitat tradicional i els estàndards de seguretat moderns.

---

## 🎯 Objectius de la Formació

En finalitzar aquest bloc, seràs capaç de resoldre escenaris reals de transferència de dades responent a:

* **Funcionament intern:** Com opera realment el protocol FTP?
* **Modes de connexió:** Quina diferència hi ha entre el **mode actiu** i el **mode passiu**?
* **Seguretat:** Com implementar un servidor FTP segur.
* **Alternatives modernes:** El protocol **sFTP** com a substitut robust.
* **Aïllament:** Tècniques d'**engabiat (chroot)** d'usuaris en connexions SFTP.
* **Estat de l'art:** Altres mètodes alternatius per a la transferència de fitxers.

---

## 🛠️ Pla de Treball

La formació es divideix en dues fases clarament diferenciades:

### 1. Fonaments Teòrics i Seguretat
Estudi exhaustiu de la pila de protocols i els ports implicats. Pararem especial atenció a la seguretat de les dades i l'aïllament d'usuaris.



### 2. Laboratori Pràctic (The "Real World")
Configuració d'entorns reals mitjançant màquines virtuals en dos escenaris:

* **Activitat A - Servidor FTP:** * Configuració d'un servidor estàndard.
    * Creació d'usuaris i permisos de lectura/escriptura.
    * Implementació de l'engabialament (**chroot**).
    * *Anàlisi:* Verificació de com les dades viatgen "en clar" (sense xifrar).
* **Activitat B - Servidor sFTP (Secure FTP):**
    * Implementació de la transferència sobre el protocol **SSH**.
    * Configuració avançada d'engabiat per evitar fuites de seguretat del sistema.

> **La seguretat no és una opció, és una obligació.** Com a administradors, entendre la diferència entre aquests protocols és vital per al vostre futur professional.

---

## 📊 Sistema d'Avaluació

Per superar aquesta formació, cal demostrar competència en dos àmbits:

| Prova | Descripció | Pes (%) |
| :--- | :--- | :---: |
| **Prova Escrita** | Teoria sobre protocols, ports, modes i seguretat. | **40%** |
| **Examen Pràctic** | Repte cronometrat: Aixecar servidors FTP/sFTP des de zero. | **60%** |

---

## 📚 Materials i Links de Suport

* 📂 **Plataforma virtual:** [Moodle Serveis de Xarxa]
* 💻 **Eines:** VirtualBox/VMware, clients FTP (FileZilla, WinSCP).

---

**Prepareu les vostres màquines virtuals... Comencem a transferir dades!** 📂🌐

---

