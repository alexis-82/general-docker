# Descrizione e configurazione dei Bridge Networks in Docker

Un **bridge network** in Docker è una rete virtuale privata che permette ai container di comunicare tra loro e con l'host Docker. Quando Docker avvia un nuovo container senza specificare un network, di default lo collega a un bridge network. Ogni bridge network funziona come un segmento di rete isolato all'interno dell'host Docker, con il suo range di indirizzi IP. I container all'interno dello stesso bridge network possono comunicare tra loro tramite indirizzi IP, indipendentemente dalle porte esposte dai container. Questo tipo di rete è ideale per la comunicazione interna tra i servizi di un'applicazione senza dover esporre le porte dei container all'host o alla rete esterna.

In pratica, i bridge network in Docker permettono di far comunicare i container con tutta la nostra rete locale attraverso il meccanismo di bridge, che gestisce l'interazione tra i container e l'host Docker. Questo facilita l'integrazione delle applicazioni containerizzate con altri servizi e risorse della rete locale, mantenendo nel contempo l'isolamento e la sicurezza dei container stessi.

Il bridge network può essere configurato tramite Docker Compose per definire come i container devono essere collegati e come devono comunicare tra loro e con il mondo esterno.

## Configurazione Bridge Network in Docker
Prima di tutto crea la rete `macvlan`:

```bash
docker network create -d macvlan \
  --subnet=192.168.1.0/24 \
  --gateway=192.168.1.1 \
  -o parent=ens32 \
  my_macvlan_network
```

`-o parent=en32`: inserire il nome della vostra interfaccia fisica

### Esempio di file docker-compose.yml

```yaml
services:
  db:
    image: mysql:latest
    container_name: mysql_db
    restart: always
    env_file:
      - mysql.env
    volumes:
      - ./database/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./database/data:/var/lib/mysql
    networks:
<<<<<<< HEAD
      my_macvlan_network:
        ipv4_address: 192.168.1.200  # Cambia l'indirizzo IP in base alla tua rete
    healthcheck:
      test: ["CMD", "mysqladmin", "ping", "-h", "localhost"]
      interval: 10s
      timeout: 5s
      retries: 5
=======
      - db_network
>>>>>>> 3e3223a5767909683db790f3e5f414a447151a74

  phpmyadmin:
    image: phpmyadmin:latest
    container_name: phpmyadmin
    restart: always
    depends_on:
      db:
        condition: service_healthy
    env_file:
      - phpmyadmin.env
    ports:
      - "8080:80"
    networks:
<<<<<<< HEAD
      my_macvlan_network:
        ipv4_address: 192.168.1.201  # Cambia l'indirizzo IP in base alla tua rete
=======
      - db_network
>>>>>>> 3e3223a5767909683db790f3e5f414a447151a74

networks:
  my_macvlan_network:
    external: true
    name: my_macvlan_network
```

Configurazione del file `phpmyadmin.env` e di `mysql.env`:

phpmyadmin.env:
```
PMA_HOST=192.168.1.200
PMA_PORT=3306
UPLOAD_LIMIT=300M
MEMORY_LIMIT=512M
```

mysql.env:
```
MYSQL_USER=user
MYSQL_PASSWORD=userpassword
MYSQL_DATABASE=mydb

# Opzioni aggiuntive (opzionali)
MYSQL_ALLOW_EMPTY_PASSWORD=no
MYSQL_RANDOM_ROOT_PASSWORD=no
MYSQL_ONETIME_PASSWORD=no

# Configurazione del server (opzionale)
MYSQL_CHARACTER_SET_SERVER=utf8mb4
MYSQL_COLLATION_SERVER=utf8mb4_unicode_ci
```

Avvia il container:
```bash
docker compose up -d
```

Ora verifica se i container possono pingarsi tra di loro:
```bash
docker exec -it phpmyadmin apt-get update
docker exec -it phpmyadmin apt-get install -y iputils-ping
docker exec -it phpmyadmin ping 192.168.1.200

```

### Configurazione della Password di Root

Arresta il container MySQL:

```bash
docker compose stop db
```
Avvia il container con l'opzione per ignorare i privilegi:

Modifica temporaneamente il file docker-compose.yml con il flag `--skip-grant-tables` per poter accedere senza password:

```yaml
services:
  db:
    image: mysql:latest
    command: --skip-grant-tables
    ...
```

Avvia nuovamente il container:

```bash
docker compose up -d db
```

Accedi al database MySQL:

```bash
docker exec -it mysql_db mysql -u root
```

Una volta all'interno del client MySQL:

```sql
FLUSH PRIVILEGES;
ALTER USER 'root'@'localhost' IDENTIFIED BY 'nuova_password';
ALTER USER 'root'@'%' IDENTIFIED BY 'nuova_password';
```

Riavvia il container senza il flag --skip-grant-tables:

Ripristina il file docker-compose.yml e riavvia:

```bash
docker compose down
docker compose up -d
```
---


[![Screenshot-20240702-001820.png](https://i.postimg.cc/NfjsZCD0/Screenshot-20240702-001820.png)](https://postimg.cc/kDLPbckr)

---