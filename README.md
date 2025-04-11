# Docker mail

Comandos server

**Instalación de Docker:**

# Add Docker's official GPG key:

```bash
sudo apt-get update
```

```bash
sudo apt-get install ca-certificates curl
```

```bash
sudo install -m 0755 -d /etc/apt/keyrings
```

```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
```

```bash
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

# Add the repository to Apt sources:

```bash
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

```bash
sudo apt-get update
```

```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

```bash
sudo systemctl status docker
```

```bash
sudo systemctl start docker
```

**Instalación:**

```bash
mkdir -p ~/docker-mailserver
```

```bash
cd ~/docker-mailserver
```

```bash
curl -o setup.sh https://raw.githubusercontent.com/docker-mailserver/docker-mailserver/master/setup.sh
```

```bash
chmod a+x setup.sh
```

```bash
DMS_GITHUB_URL="https://raw.githubusercontent.com/docker-mailserver/docker-mailserver/master" 
```

```bash
wget "${DMS_GITHUB_URL}/compose.yaml"
```

```yaml
wget "${DMS_GITHUB_URL}/mailserver.env"
```

**Configuración de compose para docker-mailserver y atmoz/sftp**

```bash
sudo nano compose.yaml
```

```yaml
services:
  mailserver:
    image: ghcr.io/docker-mailserver/docker-mailserver:latest
    container_name: mailserver
    # Provide the FQDN of your mail server here (Your DNS MX record should point to this value)
    hostname: mail.example.com
    env_file: mailserver.env
    # More information about the mail-server ports:
    # https://docker-mailserver.github.io/docker-mailserver/latest/config/security/understanding-the>
    ports:
      - "25:25"    # SMTP  (explicit TLS => STARTTLS, Authentication is DISABLED => use port 465/587>
      - "143:143"  # IMAP4 (explicit TLS => STARTTLS)
      - "465:465"  # ESMTP (implicit TLS)
      - "587:587"  # ESMTP (explicit TLS => STARTTLS)
      - "993:993"  # IMAP4 (implicit TLS)
    volumes:
      - ./docker-data/dms/mail-data/:/var/mail/
      - ./docker-data/dms/mail-state/:/var/mail-state/
      - ./docker-data/dms/mail-logs/:/var/log/mail/
      - ./docker-data/dms/config/:/tmp/docker-mailserver/
      - /etc/localtime:/etc/localtime:ro
    environment:
      - PERMIT_DOCKER=network
      - ONE_DIR=1
      - DMS_DEBUG=0
    restart: always
    stop_grace_period: 1m
    # Uncomment if using `ENABLE_FAIL2BAN=1`:
    cap_add:
      - NET_ADMIN
    healthcheck:
      test: "ss --listening --tcp | grep -P 'LISTEN.+:smtp' || exit 1"
      timeout: 3s
      retries: 0

  roundcube:
    image: roundcube/roundcubemail:latest
    container_name: roundcube
    depends_on:
      - mailserver
    ports:
      - "8080:80"
    environment:
      ROUNDCUBEMAIL_DEFAULT_HOST: mail.example.com
      ROUNDCUBEMAIL_SMTP_SERVER: mail.example.com
      ROUNDCUBEMAIL_SMTP_PORT: 587
    volumes:
      - roundcube_data:/var/roundcube
    restart: unless-stopped

  sftp:
    image: atmoz/sftp:latest
    container_name: sftp
    ports:
      - "2222:22"
    volumes:
      - /srv/sftp_data:/home/foo/upload
    command: foo:pass:1001
    restart: unless-stopped

volumes:
  maildata:
  mailstate:
  maillogs:
  roundcube_data:
  sftp_data:

```

```bash
sudo ./setup.sh config dkim
```


```yaml
 sudo docker compose up -d
```
```yaml
sudo ./setup.sh email add emilio@example.com 12345
```
```bash
docker-compose -f mailserver/docker-compose.yml up -d
docker-compose -f mailserver/docker-compose.yml down
docker-compose -f mailserver/docker-compose.yml exec sftp ls /home/foo/upload
```
