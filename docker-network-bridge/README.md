# Descrizione e configurazione dei Bridge Networks in Docker

Un **bridge network** in Docker è una rete virtuale privata che permette ai container di comunicare tra loro e con l'host Docker. Quando Docker avvia un nuovo container senza specificare un network, di default lo collega a un bridge network. Ogni bridge network funziona come un segmento di rete isolato all'interno dell'host Docker, con il suo range di indirizzi IP. I container all'interno dello stesso bridge network possono comunicare tra loro tramite indirizzi IP, indipendentemente dalle porte esposte dai container. Questo tipo di rete è ideale per la comunicazione interna tra i servizi di un'applicazione senza dover esporre le porte dei container all'host o alla rete esterna.

In pratica, i bridge network in Docker permettono di far comunicare i container con tutta la nostra rete locale attraverso il meccanismo di bridge, che gestisce l'interazione tra i container e l'host Docker. Questo facilita l'integrazione delle applicazioni containerizzate con altri servizi e risorse della rete locale, mantenendo nel contempo l'isolamento e la sicurezza dei container stessi.

Il bridge network può essere configurato tramite Docker Compose per definire come i container devono essere collegati e come devono comunicare tra loro e con il mondo esterno.

## Configurazione facile di Bridge Network in Docker
Prima di tutto, ferma il servizio Docker:

```bash
sudo systemctl stop docker.socket
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
sudo systemctl stop docker.socket
sudo systemctl stop docker
```

Rimuovi l'interfaccia docker0 esistente:

```bash
sudo ip link set docker0 down
sudo brctl delbr docker0
```

Crea un nuovo bridge chiamato br0 e assegnagli un indirizzo IP:

```bash
sudo brctl addbr br0
sudo ip addr add 192.168.1.91/24 dev br0
sudo brctl addif br0 ens32
sudo ip link set br0 up
```

Aggiungi la tua interfaccia di rete (ad esempio, ens33) al bridge:

```bash
ip a
sudo brctl addif br0 ens33
```

Modifica la configurazione di Docker per utilizzare br0 invece di docker0. Crea o modifica il file `/etc/docker/daemon.json`:

```bash
sudo vim /etc/docker/daemon.json
```

Aggiungi o modifica il contenuto come segue:

```json
{
  "bridge": "br0",
  "fixed-cidr": "192.168.1.100/27"
}
```

Questo assegnerà ai container Docker indirizzi IP nel range 192.168.1.100 - 192.168.1.223.

Riavvia il servizio Docker:

```bash
sudo systemctl start docker
```

Modifica il tuo file `docker-compose.yml` per usare la rete del bridge:

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
    network_mode: bridge

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
    network_mode: bridge

volumes:
  db-data:
```

Riavvia i tuoi container:

```bash
sudo docker-compose down
sudo docker-compose up -d
```

Ora i tuoi container Docker utilizzeranno il bridge network `br0` che hai configurato.

Per verificare gli indirizzi IP assegnati ai container:

```bash
sudo docker-compose ps

sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' docker-compose-mysql-phpmyadmin-db-1

sudo docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' docker-compose-mysql-phpmyadmin-phpmyadmin-1
```

Questo ti permetterà di vedere gli indirizzi IP dei container `db` e `phpmyadmin` assegnati dalla rete del bridge `br0`.

---



[![Screenshot-20240702-001820.png](https://i.postimg.cc/NfjsZCD0/Screenshot-20240702-001820.png)](https://postimg.cc/kDLPbckr)

---

## Rimozione del Bridge br0 dalla Rete Locale

Per rimuovere il bridge br0 dalla tua rete locale, segui questi passaggi:

### Operazioni nel Docker
```
sudo systemctl stop docker
sudo docker network ls
sudo docker network rm ID my-network
```

1. **Disattiva il Bridge:**

   ```bash
   sudo ip link set br0 down
   ```

2. **Rimuovi le Interfacce dal Bridge:**

   Se hai aggiunto `ens33` al bridge, rimuovila:

   ```bash
   sudo brctl delif br0 ens33
   ```

3. **Elimina il Bridge:**

   ```bash
   sudo brctl delbr br0
   ```

4. **Ripristina la Configurazione di Rete:**

   Se hai modificato la configurazione di rete per usare `br0`, dovrai ripristinare la configurazione originale. Edita il file di configurazione dell'interfaccia (es. `/etc/network/interfaces`).

5. **Ripristina la Configurazione di Docker (Se Applicabile):**

   Se hai modificato la configurazione di Docker per usare `br0`, modifica o rimuovi la configurazione nel file `/etc/docker/daemon.json`.

6. **Riavvia il Servizio di Rete:**

   - Per sistemi che usano il comando systemctl:

     ```bash
     sudo systemctl restart networking
     ```

   - Per sistemi che usano Netplan:

     ```bash
     sudo netplan apply
     ```

7. **Riavvia il Servizio Docker:**

   ```bash
   sudo systemctl restart docker
   ```

8. **Verifica:**

   Infine, verifica che `br0` sia stato rimosso e che la tua configurazione di rete sia tornata alla normalità:

   ```bash
   ip a
   ```
