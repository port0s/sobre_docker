# inicie container com volume
```sh
  docker run -d \
  --name devtest \
  --mount source=myvol2,target=/app \
  nginx:latest
```

```sh
docker run -d \
  --name devtest \
  -v myvol2:/app \
  nginx:latest
```

# Popule um volume usando um container
```sh
docker run -d \
  --name=nginxtest \
  --mount source=nginx-vol,destination=/usr/share/nginx/html \
  nginx:latest
```

```sh
docker run -d \
  --name=nginxtest \
  -v nginx-vol:/usr/share/nginx/html \
  nginx:latest
```


# backup de volume
```sh
docker run -v /dbdata --name dbstore ubuntu /bin/bash
```

```sh
docker run --rm --volumes-from dbstore -v $(pwd):/backup ubuntu tar cvf /backup/backup.tar /dbdata
```


# retaurar volume de backup
```sh
docker run -v /dbdata --name dbstore2 ubuntu /bin/bash
```

```sh
docker run --rm --volumes-from dbstore2 -v $(pwd):/backup ubuntu bash -c "cd /dbdata && tar xvf /backup/backup.tar --strip 1"
```

# Exemplo de comandos
```sh
# docker container run -ti --mount type=bind,src=/volume,dst=/volume ubuntu
# docker container run -ti --mount type=bind,src=/root/primeiro_container,dst=/volume ubuntu
# docker container run -ti --mount type=bind,src=/root/primeiro_container,dst=/volume,ro ubuntu
# docker volume create giropops
# docker volume rm giropops
# docker volume inspect giropops
# docker volume prune
# docker container run -d --mount type=volume,source=giropops,destination=/var/opa  nginx
# docker container create -v /data --name dbdados centos
# docker run -d -p 5432:5432 --name pgsql1 --volumes-from dbdados -e POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker -e POSTGRESQL_DB=docker kamui/postgresql
# docker run -d -p 5433:5432 --name pgsql2 --volumes-from dbdados -e  POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker -e POSTGRESQL_DB=docker kamui/postgresql
# docker run -ti --volumes-from dbdados -v $(pwd):/backup debian tar -cvf /backup/backup.tar /data
```

# Exemplo de Dockerfile
```sh
FROM debian

RUN apt-get update && apt-get install -y apache2 && apt-get clean
ENV APACHE_LOCK_DIR="/var/lock"
ENV APACHE_PID_FILE="/var/run/apache2.pid"
ENV APACHE_RUN_USER="www-data"
ENV APACHE_RUN_GROUP="www-data"
ENV APACHE_LOG_DIR="/var/log/apache2"

LABEL description="Webserver"

VOLUME /var/www/html/
EXPOSE 80
```

```sh
FROM debian

RUN apt-get update && apt-get install -y apache2 && apt-get clean
ENV APACHE_LOCK_DIR="/var/lock"
ENV APACHE_PID_FILE="/var/run/apache2/apache2.pid"
ENV APACHE_RUN_USER="www-data"
ENV APACHE_RUN_DIR="/var/run/apache2"
ENV APACHE_RUN_GROUP="www-data"
ENV APACHE_LOG_DIR="/var/log/apache2"

LABEL description="Webserver"

VOLUME /var/www/html/
EXPOSE 80

ENTRYPOINT ["/usr/sbin/apachectl"]
CMD ["-D", "FOREGROUND"]
```

```sh
package main
import "fmt"

func main() {
	fmt.Println("GIROPOPS STRIGUS GIRUS - LINUXTIPS")
}

FROM golang

WORKDIR /app
ADD . /app
RUN go build -o goapp
ENTRYPOINT ./goapp

FROM golang AS buildando

ADD . /src
WORKDIR /src
RUN go build -o goapp


FROM alpine:3.1

WORKDIR /app
COPY --from=buildando /src/goapp /app
ENTRYPOINT ./goapp
```

```sh
ADD => Copia novos arquivos, diretórios, arquivos TAR ou arquivos remotos e os adicionam ao filesystem do container;

CMD => Executa um comando, diferente do RUN que executa o comando no momento em que está "buildando" a imagem, o CMD executa no início da execução do container;

LABEL => Adiciona metadados a imagem como versão, descrição e fabricante;

COPY => Copia novos arquivos e diretórios e os adicionam ao filesystem do container;

ENTRYPOINT => Permite você configurar um container para rodar um executável, e quando esse executável for finalizado, o container também será;

ENV => Informa variáveis de ambiente ao container;

EXPOSE => Informa qual porta o container estará ouvindo;

FROM => Indica qual imagem será utilizada como base, ela precisa ser a primeira linha do Dockerfile;

MAINTAINER => Autor da imagem; 

RUN => Executa qualquer comando em uma nova camada no topo da imagem e "commita" as alterações. Essas alterações você poderá utilizar nas próximas instruções de seu Dockerfile;

USER => Determina qual o usuário será utilizado na imagem. Por default é o root;

VOLUME => Permite a criação de um ponto de montagem no container;

WORKDIR => Responsável por mudar do diretório / (raiz) para o especificado nele;
```

# Exemplo de DockerHub e Registry
```sh
# docker image inspect debian
# docker history linuxtips/apache:1.0
# docker login
# docker login registry.suaempresa.com
# docker push linuxtips/apache:1.0
# docker pull linuxtips/apache:1.0
# docker image ls
# docker container run -d -p 5000:5000 --restart=always --name registry registry:2
# docker tag IMAGEMID localhost:5000/apache
```

# Exemplo de Docker Machine
```sh
Para fazer a instalação do Docker Machine no Linux, faça:

# curl -L https://github.com/docker/machine/releases/download/v0.16.1
/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine
# chmod +x /tmp/docker-machine 
# sudo cp /tmp/docker-machine /usr/local/bin/docker-machine


Para seguir com a instalação no macOS:

# curl -L https://github.com/docker/machine/releases/download/v0.16.1
/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine 
# chmod +x /usr/local/bin/docker-machine

Para seguir com a instalação no Windows caso esteja usando o Git bash:

# if [[ ! -d "$HOME/bin" ]]; then mkdir -p "$HOME/bin"; fi
# curl -L https://github.com/docker/machine/releases/download/v0.16.1
/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe"
# chmod +x "$HOME/bin/docker-machine.exe"


Para verificar se ele foi instalado e qual a sua versão, faça:

# docker-machine version

# docker-machine create --driver virtualbox linuxtips

# docker-machine ls

# docker-machine env linuxtips

# eval "$(docker-machine env linuxtips)"

# docker container ls

# docker container run busybox echo "LINUXTIPS, VAIIII"

# docker-machine ip linuxtips

# docker-machine ssh linuxtips

# docker-machine inspect linuxtips

# docker-machine stop linuxtips

# docker-machine ls 

# docker-machine start linuxtips

# docker-machine rm linuxtips

# eval $(docker-machine env -u)Para fazer a instalação do Docker Machine no Linux, faça:

# curl -L https://github.com/docker/machine/releases/download/v0.16.1
/docker-machine-`uname -s`-`uname -m` >/tmp/docker-machine
# chmod +x /tmp/docker-machine 
# sudo cp /tmp/docker-machine /usr/local/bin/docker-machine


Para seguir com a instalação no macOS:

# curl -L https://github.com/docker/machine/releases/download/v0.16.1
/docker-machine-`uname -s`-`uname -m` >/usr/local/bin/docker-machine 
# chmod +x /usr/local/bin/docker-machine

Para seguir com a instalação no Windows caso esteja usando o Git bash:

# if [[ ! -d "$HOME/bin" ]]; then mkdir -p "$HOME/bin"; fi
# curl -L https://github.com/docker/machine/releases/download/v0.16.1
/docker-machine-Windows-x86_64.exe > "$HOME/bin/docker-machine.exe"
# chmod +x "$HOME/bin/docker-machine.exe"


Para verificar se ele foi instalado e qual a sua versão, faça:

# docker-machine version

# docker-machine create --driver virtualbox linuxtips

# docker-machine ls

# docker-machine env linuxtips

# eval "$(docker-machine env linuxtips)"

# docker container ls

# docker container run busybox echo "LINUXTIPS, VAIIII"

# docker-machine ip linuxtips

# docker-machine ssh linuxtips

# docker-machine inspect linuxtips

# docker-machine stop linuxtips

# docker-machine ls 

# docker-machine start linuxtips

# docker-machine rm linuxtips

# eval $(docker-machine env -u)
```

# Exemplo de Docker Swarm
```sh
# docker swarm init

# docker swarm join --token \ SWMTKN-1-100_SEU_TOKEN SEU_IP_MASTER:2377

# docker node ls

# docker swarm join-token manager

# docker swarm join-token worker

# docker node inspect LINUXtips-02

# docker node promote LINUXtips-03

# docker node ls

# docker node demote LINUXtips-03

# docker swarm leave

# docker swarm leave --force

# docker node rm LINUXtips-03

# docker service create --name webserver --replicas 5 -p 8080:80  nginx

# curl QUALQUER_IP_NODES_CLUSTER:8080

# docker service ls

# docker service ps webserver

# docker service inspect webserver

# docker service logs -f webserver

# docker service rm webserver

# docker service create --name webserver --replicas 5 -p 8080:80 --mount type=volume,src=teste,dst=/app  nginx

# docker network create -d overlay giropops

# docker network ls

# docker network inspect giropops

# docker service scale giropops=5

# docker network rm giropops

# docker service create --name webserver --network giropops --replicas 5 -p 8080:80 --mount type=volume,src=teste,dst=/app  nginx

# docker service update <OPCOES> <Nome_Service>
```

# Exemplo de Docker Secrets
```sh
echo 'minha secret' | docker secret create 
docker secret create minha_secret minha_secret.txt
docker secret inspect minha_secret
docker secret ls
docker secret rm minha_secret
docker service create --name app --detach=false --secret db_pass  minha_app:1.0
docker service create --detach=false --name app --secret source=db_pass,target=password,uid=2000,gid=3000,mode=0400 minha_app:1.0
ls -lhart /run/secrets/
docker service update --secret-rm db_pass --detach=false --secret-add source=db_pass_1,target=password app
```

# Exemplo de Docker Compose
```sh
# mkdir /root/Composes
# mkdir /root/Composes/1
# cd /root/Composes/1

# vim docker-compose.yml

version: "3"
services:
  web:
    image: nginx
    deploy:
      replicas: 5
      resources:
        limits:	
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
    - "8080:80"
    networks:
    - webserver
networks:
  webserver:

# docker stack deploy -c docker-compose.yml primeiro
# curl 0:8080
# docker service ls
# docker service ps primeiro_web
# docker stack ls
# docker stack services primeiro
# docker stack ps primeiro
# docker stack rm primeiro

# vim docker-compose.yml

version: '3'
services:
   db:
     image: mysql:5.7
     volumes:
       - db_data:/var/lib/mysql
     environment:
       MYSQL_ROOT_PASSWORD: somewordpress
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: wordpress

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "8000:80"
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: wordpress
volumes:
  db_data:

# docker stack deploy -c docker-compose.yml segundo
# docker stack ls
# docker stack services segundo
# docker service ls
# docker service ps segundo_db
# docker service ps segundo_wordpress
# docker service logs segundo_wordpress


# mkdir 3
# cd 3
# vim docker-compose.yml

version: "3"
services:
  web:
    image: nginx
    deploy:
      replicas: 5
      resources:
        limits:
          cpus: "0.1"
          memory: 50M
      restart_policy:
        condition: on-failure
    ports:
      - "8080:80"
    networks:
      - webserver

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8888:8080"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
    networks:
      - webserver

networks:
  webserver:

# mkdir 4
# cd 4
# vim compose-file.yml

version: "3"
services:
  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]
  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure
  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - 5001:80
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure
  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]
  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]
networks:
  frontend:
  backend:
volumes:
  db-data:

# docker stack deploy -c docker-compose.yml quarto
# docker service ls

Visualizar a página de votação:
http://IP_CLUSTER:5000/

Visualizar a página de resultados:
http://IP_CLUSTER:5001/

Visualizar a página de com os containers e seus nodes:
http://IP_CLUSTER:8080/

# git clone https://github.com/badtuxx/giropops-monitoring.git
# cd giropops-monitoring

# cat docker-compose.yml

version: '3.3'

services:

  prometheus:
    image: linuxtips/prometheus_alpine
    volumes:
      - ./conf/prometheus/:/etc/prometheus/
      - prometheus_data:/var/lib/prometheus
    networks:
      - backend
    ports:
      - 9090:9090

  node-exporter:
    image: linuxtips/node-exporter_alpine
    hostname: '{{.Node.ID}}'
    volumes:
      - /proc:/usr/proc
      - /sys:/usr/sys
      - /:/rootfs
    deploy:
      mode: global
    networks:
      - backend
    ports:
      - 9100:9100

  alertmanager:
    image: linuxtips/alertmanager_alpine
    volumes:
      - ./conf/alertmanager/:/etc/alertmanager/
    networks:
      - backend
    ports:
      - 9093:9093

  cadvisor:
    image: google/cadvisor
    hostname: '{{.Node.ID}}'
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
      - /var/run/docker.sock:/var/run/docker.sock:ro
    networks:
      - backend
    deploy:
      mode: global
    ports:
      - 8080:8080

  grafana:
    image: grafana/grafana
    depends_on:
      - prometheus
    volumes:
      - grafana_data:/var/lib/grafana
    env_file:
      - grafana.config
    networks:
      - backend
      - frontend
    ports:
      - 3000:3000

# If you already have a RocketChat instance running, just comment the code of rocketchat, mongo and mongo-init-replica services bellow
  rocketchat:
    image: rocketchat/rocket.chat:latest
    volumes:
      - rocket_uploads:/app/uploads
    environment:
      - PORT=3080
      - ROOT_URL=http://YOUR_IP:3080
      - MONGO_URL=mongodb://mongo:27017/rocketchat
      - MONGO_OPLOG_URL=mongodb://mongo:27017/local
#      - MAIL_URL=smtp://user:pass@smtp.email
#      - HTTP_PROXY=http://proxy.domain.com
#      - HTTPS_PROXY=http://proxy.domain.com
    depends_on:
      - mongo
    ports:
      - 3080:3080

  mongo:
    image: mongo:3.2
    volumes:
     - mongodb_data:/data/db
     #- ./data/dump:/dump
    command: mongod --smallfiles --oplogSize 128 --replSet rs0

  mongo-init-replica:
    image: mongo:3.2
    command: 'mongo mongo/rocketchat --eval "rs.initiate({ _id: ''rs0'', members: [ { _id: 0, host: ''localhost:27017'' } ]})"'
    depends_on:
      - mongo

networks:
  frontend:
  backend:

volumes:
    prometheus_data:
    grafana_data:
    rocket_uploads:
    mongodb_data:



# docker stack deploy -c docker-compose.yml giropops
# docker service ls 
# docker stack ls

Prometheus:
http://SEU_IP:9090

AlertManager:
http://SEU_IP:9093

Grafana:
http://SEU_IP:3000

Node_Exporter:
http://SEU_IP:9100

Rocket.Chat:
http://SEU_IP:3080

cAdivisor:
http://SEU_IP:8080


# docker stack rm giropops

Lembrando, para conhecer mais sobre o giropops-monitoring acesse o repositório no GitHub e assista a série de vídeos em que falo detalhadamente como montei essa solução:
Repo: https://github.com/badtuxx/giropops-monitoring
Vídeos: https://www.youtube.com/playlist?list=PLf-O3X2-mxDls9uH8gyCQTnyXNMe10iml
```

