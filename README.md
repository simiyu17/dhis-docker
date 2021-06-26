### DHIS 2 INSTALLATION VIA DOCKERIZATION

This document does not contain docker installation instruction. So it's assumed that you've your server with docker v19+ installed.

## Necessary Modification
The only thing to do is to modify database username/password that you wish to use. Also for safety of our data in case the server goes down, it's important to point the database docker container to a directory on the server so that it saves data on the server. This is because data in the running docker container has the lifespan of that container which means for any reason the container goes down, all it's data is also lost.

## Database
DHIS 2 in this context uses the postgres database which will run as a docker container.
Open `./config/dhis2_home/dhis.conf` using any editor and you'll see the following:


connection.dialect = org.hibernate.dialect.PostgreSQLDialect
connection.driver_class = org.postgresql.Driver

# "db" maps to service name defined in Docker Compose
# "dhis2" maps to POSTGRES_DB environment variable defined in Docker Compose
connection.url = jdbc:postgresql://db/dhis2

# maps to POSTGRES_USER environment variable defined in Docker Compose.
connection.username = dhis

# maps to POSTGRES_PASSWORD environment variable in Docker Compose.
connection.password = dhis


Change the values for `connection.username=` (username) and `connection.password =` (password). Save the file and close.

## Save Data on docker host (The server)
As mention earlier, data of a docker container has the lifespan of the container. This means that you will loose your data if the container was killed in any of many circumstances. To solve this problem we map the postgres docker data to a directory on the server. When we re-run the docker container, it will continues reading/writing to this directory.
To do this, open `docker-compose.yml` in any editor. You will get the following contents:


version: '3'
services:
  db:
    image: mdillon/postgis:10-alpine
    command: postgres -c max_locks_per_transaction=100
    networks:
      - dhis
    environment:
      POSTGRES_USER: dhis
      POSTGRES_DB: dhis2
      POSTGRES_PASSWORD: dhis
    volumes:
    - /root/dhis2_home/dhisdata:/var/lib/postgresql/data
  web:
    image: dhis2/core:2.33.0
    volumes:
    - ./config/dhis2_home/dhis.conf:/DHIS2_home/dhis.conf
    environment:
    - WAIT_FOR_DB_CONTAINER=db:5432 -t 0
    networks:
      - dhis
    ports:
    - "8082:8080"
    depends_on:
    - db
    
networks:
  dhis:
    driver: bridge
    
    
    
Under services, we have 2 services (db and web). The `db` represents the database and the `web` represents the DHIS2 web application. Under the `db` service find the volumes section. In the above configurations, it's on line 29 `- /root/dhis2_home/dhisdata:/var/lib/postgresql/data`. The first section before the full colon represents the directory on the server where you intend to save the data. So in this context it's `/root/dhis2_home/dhisdata`. So this is the section you change to match your directory.

## Port
In the above same `docker-compose.yml` go to ports section under the `web` service section. I have instructed the host server to serve this web application on port `8082` as shown above. If you wish to use a different port, this is what you modify.


### Running the DHIS2
At this point, you've made all updates as mentioned above or you are okay with what is already there. Let's now bring up the DHIS2 application. On terminal, change directory to the directory that contains this file(At time of writing this it was dhis-docker).
Run `docker-compose up --build -d` to start the application. Go to `http://<hostname or IP>:PORT(if not 80)` and you should see the DHIS2 login page. Default username is `admin` and password `district`.
To stop the web application, run `docker-compose down`


NOTE: Once you've started using the application , don't make the changes we did earlier and you may loose some data. But you can always change the PORT.


































