# Installazione di Docker, Docker Compose e Docker Compose V2 su Linux

## 1. Installazione di Docker

### Aggiornare il sistema
```bash
sudo apt update
sudo apt upgrade
```

### Installare i pacchetti necessari
```bash
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

### Aggiungere la chiave GPG ufficiale di Docker
```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

### Aggiungere il repository Docker
```bash
echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

### Installare Docker
```bash
sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io
```

### Verificare l'installazione
```bash
sudo docker run hello-world
```

---

## 2. Installazione di Docker Compose

### Scaricare Docker Compose
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

### Rendere eseguibile Docker Compose
```bash
sudo chmod +x /usr/local/bin/docker-compose
```

### Verificare l'installazione
```bash
docker-compose --version
```

---

## 3. Installazione di Docker Compose V2

### Scaricare Docker Compose V2
```bash
curl -SL https://github.com/docker/compose/releases/download/v2.23.3/docker-compose-linux-x86_64 -o /usr/local/bin/docker-compose
```

[Guida Ufficiale](https://docs.docker.com/engine/install/ "Link")

### Rendere eseguibile Docker Compose V2

#### Debian
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

#### Installazione pacchetti
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Verificare l'installazione
```bash
docker compose version
```

#### Ubuntu
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt-get update
```

#### Installazione pacchetti
```bash
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

### Verificare l'installazione
```bash
docker compose version
```

Nota: Assicurati di utilizzare la versione più recente di Docker Compose V2 sostituendo `v2.23.3` con l'ultima versione disponibile al momento dell'installazione.
