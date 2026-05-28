<div align="center">
  <h1>🌐 Màquina WEB: Servidor Nginx + SFTP</h1>
  <p><i>Configuració inicial, accés per clau pública/privada i desplegament del servei web per a InnovateTech.</i></p>

  ![Nginx](https://img.shields.io/badge/Nginx-009639?style=for-the-badge&logo=nginx&logoColor=white)
  ![Ubuntu](https://img.shields.io/badge/Ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white)
  ![AWS EC2](https://img.shields.io/badge/AWS_EC2-FF9900?style=for-the-badge&logo=amazonaws&logoColor=white)
  ![SSH](https://img.shields.io/badge/SSH-4D4D4D?style=for-the-badge&logo=openssh&logoColor=white)
</div>

---

## 👥 Equip de Desenvolupament (pro-asixc1C-g1)
Aquest projecte ha estat desenvolupat i mantingut per:
* **Laia Coca**
* **Emilia Tikohonova**
* **Brenda Castro**
* **Mario Cabeza**

---

## 📖 Descripció de l'Apartat

Aquest document descriu la configuració completa de la màquina web d'InnovateTech, desplegada sobre una instància **Amazon EC2** amb Ubuntu. Inclou la creació d'usuaris, la configuració d'accés segur mitjançant clau pública/privada RSA i la instal·lació i verificació del servidor web **Nginx**.

---

# MÀQUINA WEB

## 🗺️ Apartat 2: Configuració de la Màquina Web

### 📋 Paràmetres de la Màquina

| Paràmetre | Valor |
| :--- | :--- |
| **Hostname** | `srv-web-ftps` |
| **Usuari creat** | `brenda` |
| **IP pública** | `98.83.3.235` |
| **IP privada** | `10.0.1.112` |
| **Sistema Operatiu** | Ubuntu 26.04 LTS (GNU/Linux 7.0.0-1004-aws x86\_64) |
| **Servei web** | Nginx |

---

### 🔧 2.1. Configuració Inicial

#### Creació d'Usuari i Hostname

El primer pas és crear un nou usuari al sistema i afegir-lo al grup `sudo` per disposar de privilegis d'administrador. A continuació, es canvia el hostname de la màquina per identificar-la correctament dins de la infraestructura.

```bash
# Crear el nou usuari
sudo adduser ubrenda

# Afegir l'usuari al grup sudo
sudo usermod -aG sudo brenda

# Tancar la sessió actual
exit

# Establir el nou hostname de la màquina
sudo hostnamectl set-hostname srv-web-ftps
```

> Després d'executar aquestes comandes, el prompt del sistema reflecteix el nou nom: **`brenda@srv-web-ftps:~$`**

---

### 🔑 2.2. Configuració d'Accés per Clau Pública/Privada

Per incrementar la seguretat, es desactiva l'accés per contrasenya i s'habilita l'autenticació mitjançant parell de claus RSA. D'aquesta manera es bloqueja qualsevol intent d'atac de força bruta.

**Problema inicial:** Amb la clau `.pem` d'AWS no era possible accedir directament amb l'usuari creat:

```
brenda@98.83.3.235: Permission denied (publickey).
```

**Solució:** Generació d'un nou parell de claus RSA des de la màquina local i còpia de la clau pública a la instància.

#### Pas 1 — Generar el parell de claus a la màquina local

```bash
# Generar el parell de claus RSA
ssh-keygen
# Fitxers generats:
# /home/brenda.castro.7e9/.ssh/id_rsa      → clau privada
# /home/brenda.castro.7e9/.ssh/id_rsa.pub  → clau pública

# Verificar els fitxers generats
ls ~/.ssh
# id_rsa  id_rsa.pub  known_hosts  known_hosts.old
```

#### Pas 2 — Copiar la clau pública generada

```bash
cat ~/.ssh/id_rsa.pub
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB... brenda.castro.7e9@HA207-12-WLD-006
```

#### Pas 3 — Crear el directori `.ssh` i afegir la clau a la instància

```bash
# Crear el directori .ssh a la instància
mkdir -p ~/.ssh

# Establir permisos correctes al directori
chmod 700 ~/.ssh

# Crear el fitxer authorized_keys
touch ~/.ssh/authorized_keys

# Establir permisos correctes al fitxer
chmod 600 ~/.ssh/authorized_keys

# Editar el fitxer i enganxar la clau pública copiada anteriorment
sudo nano ~/.ssh/authorized_keys
```

#### Pas 4 — Verificar que la clau s'ha afegit correctament

```bash
cat ~/.ssh/authorized_keys
# ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAAB... brenda.castro.7e9@HA207-12-WLD-006
```

#### Pas 5 — Connectar-se sense contrasenya

```bash
ssh brenda@98.83.3.235
# Welcome to Ubuntu 26.04 LTS (GNU/Linux 7.0.0-1004-aws x86_64)
```

---

> [!TIP]
> **Seguretat SSH:** Un cop configurada l'autenticació per clau pública, és recomanable desactivar explícitament l'autenticació per contrasenya editant `/etc/ssh/sshd_config` i establint `PasswordAuthentication no`, seguida d'un `sudo systemctl restart ssh`.

---

### 🌍 2.3. Configuració del Servei Web (Nginx)

#### Instal·lació d'Nginx

```bash
# 1. Actualitzar la llista de paquets
sudo apt update

# 2. Actualitzar els paquets instal·lats
sudo apt upgrade

# 3. Instal·lar el servidor web Nginx
sudo apt install nginx
```

#### Verificació de l'Estat del Servei

```bash
sudo systemctl status nginx
```

Resultat esperat:

```
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Thu 2026-05-21 07:17:50 UTC; 20s ago
    Main PID: 12671 (nginx)
      Tasks: 2 (limit: 657)
     Memory: 2.3M (peak: 5.1M)
        CPU: 35ms
```

#### Desplegament de Pàgina Web de Prova

Es modifica el fitxer `index.html` per defecte per verificar el funcionament del servidor amb contingut propi:

```bash
sudo nano /var/www/html/index.html
```

Contingut del fitxer:

```html
<!DOCTYPE html>
<html>
<head>
    <title>Prova</title>
</head>
<body>
    <h1>prova projecte blem:))</h1>
    <p>funcionament</p>
</body>
</html>
```

```bash
# Recarregar Nginx per aplicar els canvis
sudo systemctl reload nginx
```

---

> [!TIP]
> **Verificació:** Accedint a `http://98.83.3.235` des del navegador, primer apareix la pàgina de benvinguda **"Welcome to nginx!"** i, després de modificar l'`index.html` i recarregar el servei, es mostra el contingut personalitzat correctament.
