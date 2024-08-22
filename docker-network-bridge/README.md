# Descrizione e configurazione dei Bridge Networks in Docker

Un **bridge network** in Docker è una rete virtuale privata che permette ai container di comunicare tra loro e con l'host Docker. Quando Docker avvia un nuovo container senza specificare un network, di default lo collega a un bridge network. Ogni bridge network funziona come un segmento di rete isolato all'interno dell'host Docker, con il suo range di indirizzi IP. I container all'interno dello stesso bridge network possono comunicare tra loro tramite indirizzi IP, indipendentemente dalle porte esposte dai container. Questo tipo di rete è ideale per la comunicazione interna tra i servizi di un'applicazione senza dover esporre le porte dei container all'host o alla rete esterna.

In pratica, i bridge network in Docker permettono di far comunicare i container con tutta la nostra rete locale attraverso il meccanismo di bridge, che gestisce l'interazione tra i container e l'host Docker. Questo facilita l'integrazione delle applicazioni containerizzate con altri servizi e risorse della rete locale, mantenendo nel contempo l'isolamento e la sicurezza dei container stessi.

Il bridge network può essere configurato tramite Docker Compose per definire come i container devono essere collegati e come devono comunicare tra loro e con il mondo esterno.

## Configurazione facile di Bridge Network in Docker
Prima di tutto, ferma il servizio Docker:

```bash
sudo systemctl stop docker
```
Modifica la configurazione di Docker, modifica il file `/lib/systemd/system/docker.service`:

```bash
sudo vim /lib/systemd/system/docker.service
```

Modifica il contenuto come segue:

```json
ExecStart=/usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock --bip "192.168.1.100/27"
```

Questo assegnerà ai container Docker indirizzi IP nel range 192.168.1.100 - 192.168.1.223.

Riavvia il servizio Docker:

```bash
sudo systemctl daemon-reload
sudo systemctl start docker
```

Con questo metodo possiamo assegnare al network docker0 la stessa classe di indirizzo IP della nostra rete.
  
### Esempio di file docker-compose.yml

```yaml
services:
  db:
    image: mysql
    ports:
      - "3306:3306"
    restart: always
    env_file:
      - mysql.env
    volumes:
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./database/data:/var/lib/mysql
    networks: 
    	  - db_bridge

  phpmyadmin:
    image: phpmyadmin
    restart: always
    depends_on:
      - db
    env_file:
      - phpmyadmin.env
    ports:
      - "8080:80"
    environment:
      PMA_HOST: db
      PMA_PORT: 3306
      PMA_ARBITRARY: 1
    networks: 
    	  - db_bridge
    
networks:
  db_network:
    driver: bridge

volumes:
  db-data:
```
  
---  
  
## Configurazione avanzata di Bridge Network in Docker

Prima di tutto, ferma il servizio Docker:

```bash
sudo systemctl stop docker
```


Modificare la propria rete in /etc/netplan/01-network.yaml (Brdige Static):

```bash
network:
  version: 2
  renderer: networkd
  ethernets:
    ens32:
      dhcp4: no
  bridges:
    br0:
      dhcp4: no
      addresses: [192.168.1.100/24]
      nameservers:
        addresses: [8.8.8.8, 8.8.4.4]
      routes:
        - to: default
          via: 192.168.1.1
          on-link: true
      interfaces:
        - ens32
```

Ora con il comando ip a avrai un device br0.

Adesso verificare la presenza di un pacchetto:

```bash
sudo apt install python3-distutils-extra
```


---



[![Screenshot-20240702-001820.png](https://i.postimg.cc/NfjsZCD0/Screenshot-20240702-001820.png)](https://postimg.cc/kDLPbckr)

---
