# Installazione di Docker Compose V2 su Linux

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

#### Procedura per la configurazione del Gruppo
```bash
sudo apt-get install docker.io
sudo groupadd docker
sudo usermod -aG docker <nome_utente>
sudo chown $USER:$USER /var/run/docker.sock
sudo systemctl restart docke
```

Verifichiamo che funzioni tutto correttamente:
```bash
docker info
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

#### Procedura per la configurazione del Gruppo
```bash
sudo apt-get install docker.io
sudo groupadd docker
sudo usermod -aG docker <nome_utente>
sudo chown $USER:$USER /var/run/docker.sock
sudo systemctl restart docke
```

Verifichiamo che funzioni tutto correttamente:
```bash
docker info
```

### Verificare l'installazione
```bash
docker compose version
```

Nota: Assicurati di utilizzare la versione più recente di Docker Compose V2 sostituendo `v2.23.3` con l'ultima versione disponibile al momento dell'installazione.
