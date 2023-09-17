# 1
## O que é um container?
São recursos da máquina isolados (uma jaula) como:  
- chroot  
- jails  
- solariszone  
- lxc  
- openvz  

## Tipos de Isolamento
### Lógico
**Namespace**: isola recursos lógicos da máquina, como:  
- processos  
- network  
- mount pont  
- users  

### "Físico"
**CGroup**: isola recursos de hardware da máquina, como:  
- CPU  
- memória  
- IO de rede  
- IO de bloco  

---

# 2
## O que é copy on write?
Ao obter uma imagem de container (que carrega os utilitários de um sistema operacional),
tem-se várias camadas e nelas várias tarefas rodando, como:  
- debian  
- apache  
- php  

> Apenas a camada superior é **Read and write**  

## Copy and Write: exemplo
Uma imagem está pronta e uma container está rodando, deseja-se
editar algo que esta num camada abaixo (camada read-only), como fazer?  

### Copy and Write: resolução do exemplo
O Docker realiza uma cópia exata do que se deseja alterar, na camada superior (read and write)  

## Motivação
Caso uma imagem esteja rodando e se deseje subir outra com a mesma base,
o Docker irá usar a imagem original (sem cópia, apenas amesma imagem).  
 
Isso facilita a utilização de recursos de armazenamento, 
pois não será necessário usar espaço a mais para alocar a base do container,
apenas o da última camada (read and write).

---
 
# 3
## Namespace, Netfilter CGroup
O container utiliza o kernel da máquina local  

### Netfilter
Módulo para o funcionamento de rede dentro do kernel, quem vêm dentro do kernel.  

### Namespace
Módulo para o funcionamento de isolamentos específicos, quem vêm dentro do kernel.    

#### PID Namespace
Isolamento de processos.  

#### NET Namespace
Isolamento de redes.  

#### MT Namespace
Isolamento de mounted points (pontos de montagem).  

### CGroup
Responsável por fazer isolamento de:  
- CPU  
- Memória  
- Network  
- Disco  

---

# 4
## Client
Comando para acesso ao Docker.  

## Engine
Demon (Server) do Docker.  

---

# 5 e 6
## Comandos
O Docker Client possui comandos que capacitam seu uso em relação a cada segmento.  

### Container
```
  docker container
```  

### Images
```
  docker image
```  

### Network
```
  docker network
```  
 
### Volume
```
  docker volume
```  

## Processo de execução do Docker
Nesta sessão, um container será rodado.  
 
A imagem escolhida foi a **hello-world**.  

### Rode o hello-world
> -it == entre no modo iterativo através do terminal  
```
  docker container run -it hello-world
```  

### Passos que o Docker realiza
1. Docker Client contata o Docker Daemon  
2. Docker Daemon faz o download (pulled) da imagem no Docker Hub, caso a imagem não exista no host  
3. Docker Daemon cria um novo container, baseado na imagem requisitada  
4. Docker Daemon exibe a saída para o Docker Client  

### Liste os containeres em execução
```
    docker container ls
```  

A saída é parecida com:  
```
    CONTAINER ID   IMAGE          COMMAND                   CREATED         STATUS         PORTS     NAMES
    a66138454608   hello-world    "/docker-entrypoint."     7 seconds ago   Up 5 seconds             hello-world
```  
---

# 7
## Configuração de memória e CPU
Pode-se configurar o uso de recursos da máquina por parte de um container,
através da linha de comando.  

> -d == rode como um daemon  
> -m || --memory == configuração de memória  
```
  docker container run -d -m 128M nginx
```  

**OBS**  
Com container, não é necessário usar SWAP (Kubernets não permite usar SWAP)  


> 19c == forma concisa (três primeiras letras) de referenciar um container  
```
  docker container inspect 19c
```  

Após encontrar o número referente à memória, converta-o para M.
```
  bc << "134217728/1024/1024"
  128
```  

> -d == rode como um daemon  
> -m || --memory == configuração de memória  
> --cpus == configuração de uso máximo de cpus  
```
  docker container run -d -m 128M --cpus 0.5 nginx
```  

---

# 8
# Primeiro Dockerfile  
Cada um desses termos, forma uma pilha de camadas:  
- **FROM**    == Imagem base  
- **WORKDIR** == Diretório de trabalho dentro do container  
- **COPY**    == Copia coisas de fora para dentro da imagem  
- **RUN**     == Comando a serem executados dentro da imagem no momento de sua construção (build)  
- **ENV**     == Variáveis de ambiente  
- **CMD**     == Qual comando será executado ao aplicar RUN, quando o container já tiver sido construido (buildado)  

```sh
FROM debian                      # De qual imagem 

LABEL app="nome_da_app"          # Metadata
ENV VARIAVEL_DE_AMBIENTE="PORRA"

RUN apt update && apt install -y stress && apt clean # Rode estes comandos

CMD stress --cpu 1 --vm-bytes 64M --vm 1  # Execute alguma ação (apenas 1 CMD por Dockerfile)
```  

## Construa (build) um Dockerfile
No mesmo diretório no qual o Dockerfile foi criado:  
> -t == tag para uma imagem  
```sh
  docker image build -t toskeira1.0
```  

## Execute (rode) um container
A partir da imagem criada do Dockerfile acina, rode um container:  
> -d == rode o container em segundo plano (como um daemon)  
```sh
  docker container run -d toskeira1.0
```  

---

# 9
**Fim do dia 1**  
 
---

# 10, 11 e 12
## O que é um volume para um container?
Volume é um "pedaço" do File System (Sistema de Arquivos), isolado
dentro de um container a fim de armazenar as modificações realizadas
dentro daquele espaço  
 
## Tipos de Volume
Ao subir um Dockerfile sem nenhuma configuração de "run",
por padrão, o Docker criará um volume automaticamente
do tipo "**volume**".  
 
**OBS 1**
Ao não declarar o tipo de um **volume**, o container sobe com um nome aletório.  
  
**OBS 2**
Ao matar um container e subir outro com as mesmas opções (identificações diferentes), 
o **volume** poderá ser acessado normalmente.  
 
**OBS 3**
Por padrão, o Docker não replica os dados de um volume para outros nodes num cluster.  

### Bind
Quando já se tem um diretório e deseja montá-lo dentro do container.  

Criação de diretório:  
```
  mkdir -p /opt/para_dentro_do_container
```  
 
Comando para montar o diretório dentro do container:  
> -it                == entre no modo iterativo através do terminal  
> type               == declara o tipo do volume  
> source || src      == diretório local montado dentro do volume  
> destination || dst == diretório montado no volume  
> ro                 == read-only  
```
  docker container run -it --mount type=bind,src=/opt/para_dentro_do_container,dst=/diretorio_dentro_do_container,ro debian
```  

**OBS**
--volume-from == antiga declaração  
--mount       == nova declaração  
 
#### Data Only
Exemplo de criação de container com volume tipo bind: declaração antiga.  
> --name {CONTAINER_NAME}           == nome do container  
> --volumes-from {VOLUME_NAME}      == pega todos os volume em {CONTAINER_NAME} e monte com este container   
```
   docker container run --name CONTAINER_NAME --volumes-from VOLUME_NAME
```  

Observe com um exemplo de volume especidficado abaixo:  
> -v                                == declaração de volume da maneira antiga  
> /opt/para_dentro_do_container     == diretório local onde fica armazenado o volume  
> /data                             == diretorio dentro do container -> Postgres monta dados do banco em /data, por isso esse nome  
> --name {VOLUME_NAME}              == nome do volume  
```
  docker container create -v /opt/para_dentro_do_container:/data --name VOLUME_NAME centos
  docker container create -v /data --name VOLUME_NAME centos
```  
 
Execução de containeres com PostgreSQL:  
> -d                                == roda como daemon
> -p HOST_PORT:CONTAINER_PORT       == docker redireciona de HOST_PORT para CONTAINER_PORT, pela regras do IPtables etc.
> --name {CONTAINER_NAME}           == nome do container
> --volumes-from {VOLUME_NAME}      == pega todos os volume em {CONTAINER_NAME} e monta aqui
```
  docker container run -d -p 5432:5432 --name CONTAINER_NAME --volumes-from VOLUME_NAME
  -e POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker -e POSTGRESQL_DB=docker {IMAGE_NAME}
   
  docker container run -d -p 5433:5432 --name CONTAINER_NAME --volumes-from VOLUME_NAME
  -e POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker -e POSTGRESQL_DB=docker {IMAGE_NAME}
```  
 
### Volume
Nesta sessão, será abordada a criação de volumes.  
  
#### Local dos Volumes
Todas os volumes criados, estão no diretótio abaixo:  
```
 "/var/lib/docker/volumes/"
```  
 
> ${VOLUME_NAME} == nome do volume  
```
 "/var/lib/docker/volumes/${VOLUME_NAME}/_data"
```  

#### Criação de arquivo dentro do volume
Utilizando o mesmo local acima, pode-se criar um arquivo dentro de um volume:  
> ${VOLUME_NAME} == nome do volume  
```
 cd /var/lib/docker/volumes/${VOLUME_NAME}/_data
 touch arq_0
 touch arq_1
```  

##### Suba um container e verifique os arquivos
Agora, um container irá acessar os dados contidos dentro do volume.  
```
 docker container run -it --mount type=volume,src=${VOLUME_NAME},dst=/${DIR_IN_CONTAINER} ${IMAGE_NAME}
```  

**OBS**
Ao declarar um volume do tipo "volume" seguindo o comando run,
mesmo que o volume não tenha sido anteiormente criado,
terá sua criação realizada.  

Uma vez dentro do container, entre no diretório cujo nome está em **dst** e liste os arquivos:  
```
 cd ${DIR_IN_CONTAINER}
 ls
```  
 
Agora, de dentro do container, crie um arquivo que ficará armazenado dentro do volume:  
```
 touch arq_2
```  
 
Verifique o arquivo recém criado:  
```
 ls /var/lib/docker/volumes/${VOLUME_NAME}/_data
```  

#### Create
Cria um volume:  
```
  docker volume create meu_volume
```  
 
#### Inspect
Inspecione volumes:  
```
  docker volume inspect meu_volume
```  
 
Comando para montar o diretório dentro do container:  
> -it                == entre no modo iterativo através do terminal  
> type               == declara o tipo do volume  
> source || src      == volume  
> destination || dst == diretório montado no volume  
> ro                 == read-only  
```
  docker container run -it --mount type=volume,src=meu_volume,dst=/diretorio_dentro_do_container,ro debian
```  
 
#### Rm
Deleção (remoção) de volumes:  
```
  docker volume rm meu_volume
```  
 
#### Prume
Deleção (remoção) de volumes que não estiverem sendo utilizados por algum container:  
```
  docker volume prune
```  

---

# 13 e 14
## Desafio: proposição
Crie um volume e suba dois containeres com a imagem do PostgreSQL  

## Desafio: resposta
Criação de volume:  
```
    docker volume create db_dados
```  

Rode o primeiro container do PostgreSQL, usando o volume __db_dados__:  
> -d                                == roda como daemon
> -p HOST_PORT:CONTAINER_PORT       == docker redireciona de HOST_PORT para CONTAINER_PORT, pela regras do IPtables etc.
> --name {CONTAINER_NAME}           == nome do container
> --mount            == nova declaração  
> type               == declara o tipo do volume  
> source || src      == volume  
> destination || dst == diretório montado no volume  
```
  docker container run -d -p 5432:5432 --name postgres_1 --mount type=volume,src=db_dados,dst=/data
  -e POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker -e POSTGRESQL_DB=docker kamui/postgresql
```  
   
Rode o segundo container do PostgreSQL, usando o volume __db_dados__:  
> -d                                == roda como daemon
> -p HOST_PORT:CONTAINER_PORT       == docker redireciona de HOST_PORT para CONTAINER_PORT, pela regras do IPtables etc.
> --name {CONTAINER_NAME}           == nome do container
> --mount            == nova declaração  
> type               == declara o tipo do volume  
> source || src      == volume  
> destination || dst == diretório montado no volume  
```
  docker container run -d -p 5433:5432 --name postgres_2 --mount type=volume,src=db_dados,dst=/data
  -e POSTGRESQL_USER=docker -e POSTGRESQL_PASS=docker -e POSTGRESQL_DB=docker kamui/postgresql
```  

Exiba as informações em __db_dados__:  
```
    docker volume ls
```  
 
Exiba informações sobre o Volume:   
```
    docker volume inspect db_dados
```  
 
Exiba as informações sobre o banco de dados criado:  
```
 ls /var/lib/docker/volumes/db_dados/_data
```  

---

# 15
## Backup
Realização de backup dos dados contido dentro do volume.  
Será criado um arquivo cuja extensão é **.tar**.  

**OBS**
Um arquivo **.tar**, significa que arquivos foram empacotados (estão juntos).  
Isso é diferente de compactação, onde haveria aplicação de algoritmos de compactação.  

O backup será realizado dentro de outro container, utilizando um diretório ja existente.  

### Criação do volume: tipo bind
```
    mkdir -p /opt/backup
```  

### Comando para realização de backup
```
    docker container run -it 
    --mount type=volume,src=db_dados,dst=/data
    --mount type=bind,src=/opt/backup,dst=/backup
    debian
    tar -cvf /backup/meu_backup.tar /data
```  

### Exiba os dados armazenados
Mova-se para o diretótio onde os dados estão armazenados:  
```
    cd /opt/backup
```  
 
Desempacote o arquivo **.tar**:  
```
    tar -xvf meu_backup.tar
```  

Liste os arquivos:  
```
    ls
```  

### Comentário de cada linha
Rode o container no modo iterativo  
> -it == entre no modo iterativo  
```
    docker container run -it
```  

Montagem do volume do qual os dados serão extraídos:  
> --mount            == nova declaração  
> type               == declara o tipo do volume  
> source || src      == volume  
> destination || dst == diretório montado no volume  
```
    --mount type=volume,src=db_dados,dst=/data
```  

Montagem do volume dentro do qual os dados serão armazenados:  
> --mount            == nova declaração  
> type               == declara o tipo do volume  
> source || src      == volume  
> destination || dst == diretório montado no volume  
```
    --mount type=bind,src=/opt/backup,dst=/backup
```  

Nome da imagem:  
```
    debian
```  

Declaração do comando **tar**, dentro do container rodando imagem **debian**:  
> -c || --create   == cria um novo pacote  **.tar**
> -v || --verbose  == modo verboso, exibe na tela as informações  
> -f || --file     == arquivo que será criado
```
    tar -cvf /backup/meu_backup.tar /data
```  

---

# 16, 17, 18, 19 e 22
## Dockerfile
Arquivo para configuração de imagens.  
* FROM                         == Imagem base.  
* FROM <IMAGE> AS <ALIAS>      == Imagem base.  
* WORKDIR                      == Diretório de trabalho dentro do container.  
* COPY                         == Copia coisas de fora para dentro da imagem.  
* ADD                          == Copia coisas de fora para dentro da imagem: .tar -> copia apenas o que estiver dentro do .tar && arquvo remoto: https://arquivo.com/arquivo.  
* RUN                          == Comandos a se executar dentro da imagem, durante a execução do container. Cada RUN é uma camada.  
* ARG <VARIABLE_NAME>          == Argumento. Diferente do ENV, pode ser mudado na hora do build.
* ENV                          == Variáveis de ambiente.  
* ENV <VARIABLE_NAME> <VALUE>  == Variáveis de ambiente, ex: ENV app_dir /opt/app; WORKDIR ${app_dir}  
* HEALTHCHECK                  == Permite ao container fazer a checagem ao longo do tempo.
* CMD                          == Qual comando será executado ao aplicar RUN.  
* LABEL                        == Descrição referente a: chave=valor.  
* USER                         == Usuário específico
* VOLUME                       == Declaração de (local do) volume (tipo volume).  
* EXPOSE                       == Esponha numa porta.  
* ENTRYPOINT                   == Principal processo do container. Se morrer, máquina morre junto. Similar ao init, dentro do Linux.  

### Exemplo de Dockerfile
```sh
FROM debian                                     # Image utilizada

RUN apt-get update &&\
apt-get install -y apache2 curl &&\
apt-get clean                                   # apt já remove os .deb
# RUN chown www-data:www-data /var/lock && chown www-data:www-data /var/run && chown www-data:www-data /var/log # permite USER diferente do root
ENV APACHE_LOCK_DIR="/var/lock"                 # impede dois processos do Apache de rodar simultâneamente
ENV APACHE_PID_FILE="/var/run/apache2.pid"      # identificação do processo (PID -> Process IDentification)
ENV APACHE_RUN_USER="www-data"                  # Usuário responsável pelo Apache
ENV APACHE_RUN_GROUP="www-data"                 # Grupo responsável pelo Apache
ENV APACHE_LOG_DIR="/var/log/apache2"           # Salva logs do Apache

ADD index.html /var/www/html                    # Copia o arquivo "index.html", (neste caso, "index.html" deve estar no mesmo diretório que Dockerfile) para dentro do container

HEALTHCHECK --interval=1m --timeout=3s \        # A cada 1 minuto, espere 3 segundos e aplique o CMD (comando) curl a localhost
    CMD curl -f http://localhost/ || exit 1

LABEL description="Webserver"                   # Descrição da imagem
LABEL version="1.0.0"                           # Descrição da versão

# USER www-data                                 # Usuário a ser usado inicialmente. Só permite ligar (realizar o bind) em porta alta colocada EXPOSE, ex: EXPOSE 8080
USER root                                       # Usuário a ser usado inicialmente

WORKDIR /var/www/html                           # Diretório de trabalho dentro do container

VOLUME /var/www/html/                           # Local no qual ficará o volume
EXPOSE 80                                       # Declaração de qual porta roda o (server) Apache

ENTRYPOINT ["/usr/sbin/apachectl"]              # modo exec: Comando principal do container
CMD ["-D", "FOREGROUND"]                        # modo exec: Chama comandos baseados no ENTRYPOINT (o principal processo) e 
                                                # diz ao Apache para rodar em primeiro plano

# CMD /usr/sbin/apachectl -D FOREGROUND         # modo shell: similar ao acima, mas roda no shell principal da imagem
```  

**OBS**
> -P == verifica o Dockerfile, buscando EXPOSE. Caso exista, rode (bind) o valor em EXPOSE numa porta aleatória do host.
```
    docker container run -it -P
```  

### Build de Dockerfile
-t == nome da imagem
```
    docker image build --tag=meu_apache:1.0.0 .
    docker image build -t meu_apache:1.0.0 .
```  


### Rode o container contruído (buildado) com Dockerfile
> -d                           == rode o container em segundo plano (como um daemon)  
> -p HOST_PORT:CONTAINER_PORT  == docker redireciona de HOST_PORT para CONTAINER_PORT, pela regras do IPtables etc.
```
    docker container run -d -p 8080:80 meu_apache:1.0.0
```  

### Inspecione o container contruído (buildado) com Dockerfile
> 19c == forma concisa (três primeiras letras) de referenciar um container  
```
  docker container inspect <CONTAINER_ID>
```  

### Verifique os logs do container contruído (buildado) com Dockerfile
> -f == siga a saída do log
```
  docker container logs -f <CONTAINER_ID>
```  

---

# 20 e 21
## Multi Stage
Possibilita a interação entre as fases (camadas) da geração da imagem.  
A imagem que permanece, é a mais abaixo no arquivo **Dockerfile**.  

### Arquivo go
```
    package main
    import "fmt"

    func main() {
        fmt.Println("GIROPOPS STRIGUS GIRUS - LINUXTIPS")
    }
```

### Dockerfile Multi Stage
Faz muito sentido para aplicações fechadas, como as que rodam com um binário.  
```
FROM golang AS buildando                # Imagem "golang", sendo apelidada de "buildando"


WORKDIR /app                            # Diretório dentro do container
ADD . /app                              # Adiciona o que estiver no mesmo diretório do Dockerfile, para /app
RUN go build -o goapp                   # Ao subir o CONTAINER, rode este comando
ENTRYPOINT ./goapp                      # Principal processo do container


FROM alpine:3.1                                # Imagem

WORKDIR /app                                   # Diretório dentro do container
COPY --from=buildando /app/goapp /meu_app_go   # Copia da camada acima: "/app/goapp" (em "/app", o binário "goapp"), para: "/meu_app_go"
ENTRYPOINT ./meu_app_go                        # Principal processo do container
```  

### Build de Dockerfile
-t == nome da imagem
```
    docker image build --tag=go_app:1.0.0 .
    docker image build -t go_app:1.0.0 .
```  


### Rode o container contruído (buildado) com Dockerfile
> -it == entre no modo iterativo através do terminal  
```
    docker container run -it go_app:1.0.0
```  

### Inspecione o container contruído (buildado) com Dockerfile
> 19c == forma concisa (três primeiras letras) de referenciar um container  
```
  docker container inspect <CONTAINER_ID>
```  

### Verifique os logs do container contruído (buildado) com Dockerfile
> -f == siga a saída do log
```
  docker container logs -f <CONTAINER_ID>
```  

---

# 22
## Docker commit
Criação de uma imagem a partir de outra imagem.  

**OSB**
Não é a melhor maneira de ser feita (que é com Dockerfile).  

### 1º Passo - Execute um container
```
    docker container run -it debian bash
```  

### 2º Passo - Edite este container
```
    apt install curl nvim
```  

### 3º Passo - Saia do container sem matá-lo
```
    CTRL+p q
```  

### 4º Passo - Pegue o ID do container
```
    docker ps -a
```  

### 5º Passo - Dê o comando com o commit
> -m == escreva uma mensagem
```
    docker commit -m "Debian com curl e nvim" <CONTAINER_ID>
```  

### 6º Passo - Coloque um nome para esta nova imagem
> -m == escreva uma mensagem
```
    # docker image tag <CONTAINER_ID> <CONTAINER_TAG>:1.0
    docker image tag <CONTAINER_ID> debian_curl_nvim:1.0
```  

### 7º Passo - Coloque seu nome de usuário do DockerHub
```
    # docker image tag <CONTAINER_ID> <DOCKER_HUB_USER>/<CONTAINER_TAG>:1.0
    docker image tag <CONTAINER_ID> boguinha/debian_curl_nvim:1.0
```  

---

# 23 e 24
## DockeHub
Repositório (registry) que armazena imagens oficiais docker.  

### Faça login com o DockerHub
```
    docker login
```  

### Suba a imagem para o DockerHub
```
    # docker imagem push <DOCKER_HUB_USER>/<CONTAINER_TAG>:<TAG_NUMBER>
    docker imagem push <CONTAINER_ID> boguinha/debian_curl_nvim:1.0
```  

---

# 25 e 26
## Docker Registry
Repositório que armazena as imagens do docker.  
 
Aqui, um container registry armazenará as imagens localmente.  

### Rode o Registry
> -d == rode como um daemon  
> -p HOST_PORT:CONTAINER_PORT       == docker redireciona de HOST_PORT para CONTAINER_PORT, pela regras do IPtables etc.
> --restart {DEFUALT}               == nome do padrão  
> --name {CONTAINER_NAME}           == nome do container  
```
    docker container run -d -p 5000:5000 --restart=always --name registry registry:2
```  

### Suba a imagem para o Registry Local
```
    # docker imagem push <DOCKER_HUB_USER>/<CONTAINER_TAG>:<TAG_NUMBER>
    docker imagem push localhost:5000/debian_curl_nvim:1.0
```  

### Rode a imagem
> -p HOST_PORT:CONTAINER_PORT       == docker redireciona de HOST_PORT para CONTAINER_PORT, pela regras do IPtables etc.
```
    docker container run -d localhost:5000/debian_curl_nvim:1.0
```  

### Exiba as imagens
```
    curl localhost:50001/v2/_catalog
```  

### Exiba as versões de uma imagem
```
    # curl localhost:50001/v2/<CONTAINER_TAG>/tags/list
    curl localhost:50001/v2/debian_curl_nvim/tags/list
```  

---

# 27
## Docker Machine
Docker com máquinas virtuais.  

O Docker Machine deve ser instalado na **MÀQUINA LOCAL**,
depois dever ser configurado conforme 
o local no qual se deseja criar a máquina virtual.  

## Docker Machine: instalação
```
    curl -L https://github.com/docker/machine/releases/download/<SET_VERISON>/docker-machine-`uname -s`-`uname -m` > /tmp/docker-machine
```  

## Docker Machine: comando inicial
```
    docker-machine --help
```  

## Docker Machine: create
```
    # docker-machine --driver <DRIVER_NAME> <VIRTUAL_MACHINE_NAME>
    docker-machine --driver virtualbox jongas
```  

## Docker Machine: list as máquinas
```
    docker-machine ls
```  

## Docker Machine: ip da máquina virtual
```
    # docker-machine ip <VIRTUAL_MACHINE_NAME
    docker-machine ip jongas
```  

## Docker Machine: conectar-se a máquina virtual
```
    # docker-machine ssh <VIRTUAL_MACHINE_NAME
    docker-machine ssh jongas
```  

## Docker Machine: exiba as variáveis de ambiente
```
    # docker-machine env <VIRTUAL_MACHINE_NAME
    docker-machine env jongas
```  

### Docker Machine: estabeleça uma conexão entre a máquina loca e a virtual
```
    # eval $(docker-machine env <VIRTUAL_MACHINE_NAME)
    eval $(docker-machine env jongas)
```  

---

# 28, 30 e 31
## Docker Swarm
Realiza a gerencia de container através de um orquestrador built-in.  

No momento de realizar a integração entre seu container e outras ferramentas,
orquestrar é essencial.  

Isso exige uma ferramenta epecífica para isso, como: docker-swarm, kubernets etc.  

## Portas cluster Swarm
2377   |  TCP      |   Comunicação do gerenciamento do cluster
7946   |  TCP/UDP  |   Comunicação entre os nós do cluster
4789   |  UDP      |   Comunicação entre as redes overlay

### Papéis dentro do Swarm
#### Manager
A função do Manager é conhecer todos os detalhes do cluster.  
É no Manager que esta toda a administração do container.  
Dentro do cluster, estão todos os container de todos os serviços.  

#### Worker
A função do Worker é ter um container rodando.  

#### Eleição
Quando um manager cai, acontecerá uma eleição entre os outros managers,
em busca de estabelecer o próximo a estar ativo.  

##### Down Time
Como ocorre uma eleição entre os managers, isso quer dizer,
cada manager será avaliado para a posição de eleito (em atividade),
quando se têm muitos concorrendo, pode levar um tempo muito logo.  

E disso, pode decorrer um down time.  

Por isso, é sempre importante ponderar sobre o número de managers
rodando num cluster.  

### Conta de funcionamento do cluster Manager
Para um cluster Swarm seguir funcionando, faz-se necessário 51% de Manager.  

#### Exemplos de cálculo para cluster
Os cálculos abaixo são realizados para exemplificar o mantenimento da atividade do cluster.  

##### Exemplo 1
Manager | Porcentagem de cluster Manager up | Quantos caem |  Cluster continua vivo?
   1    |                 100%              |      1       |           Não

##### Exemplo 2
Manager | Porcentagem de cluster Manager up | Quantos caem |  Cluster continua vivo?
   1    |                 100%              |      0       |           Sim

##### Exemplo 3
Manager | Porcentagem de cluster Manager up | Quantos caem |  Cluster continua vivo?
   2    |                 100%              |      1       |           Não

##### Exemplo 4
Manager | Porcentagem de cluster Manager up | Quantos caem |  Cluster continua vivo?
   3    |                 100%              |      1       |           Sim

##### Exemplo 5
Manager | Porcentagem de cluster Manager up | Quantos caem |  Cluster continua vivo?
   3    |                 100%              |      2       |           Não

##### Exemplo 6
Manager | Porcentagem de cluster Manager up | Quantos caem |  Cluster continua vivo?
   4    |                 100%              |      2       |           Não

##### Exemplo 7
Manager | Porcentagem de cluster Manager up | Quantos caem |  Cluster continua vivo?
   4    |                 100%              |      3       |           Não

##### Exemplo 8
Manager | Porcentagem de cluster Manager up | Quantos caem |  Cluster continua vivo?
   5    |                 100%              |      2       |           Sim

##### Exemplo 9
Manager | Porcentagem de cluster Manager up | Quantos caem |  Cluster continua vivo?
   5    |                 100%              |      3       |           Não

### Docker Swarm: comandos
```
    docker swarm --help

      ca          Exiba e rode os certificados
      init        Inicialize a swarm
      join        Junte a swarm a um node (worker) e/ou a um manager, entre no cluster
      join-token  Gerencie a união entre tokens, gerencia os tokens
      leave       Deixe a swarm, saia do cluster
      unlock      Desbloqueie swarm
      unlock-key  Gerencie a chave de desbloqueio
      update      Atualize o swarm
```  

## Docker Swarm: inicie um cluster.
```
    docker swarm init
```  

## Docker Swarm: inicie um cluster com um IP.
> caso deseje espeificar outra interface de rede.  
```
    # docker swarm init --advertise-addr <IP>
    docker swarm init --advertise-addr 192.168.0.1
```  

Ao iniciar um cluster (docker swarm init), uma linha de comando é exibida.  
Esta linha, deve ser adicionada a **OUTRAS MÁQUINAS**,
que serão reconhecidas como **WORKERS**.  
```
    docker swarm join --token SWMTKN-1-4g1wky72ktbs7fsu8stkov3dceag53bz0tczmn35vz12y0j2ql-dsrs9ph15rob7yr3s4ctt6po2 192.168.0.1:2377
```  

## Docker Swarm Node: exiba os nodes
Dentro da máquina principal (leader), exiba os node.  
```
    docker nodes ls
```

## Docker Swarm Node: promova um worker
Dentro da máquina principal (leader),
promova um worker ao status **rechable** (alcançável a ser lider)
```
    docker node promote <WORKER_HOSTNAME>
```  

## Docker Swarm Node: despromova um worker
Dentro da máquina principal (leader),
despromova um worker ao status **rechable** (alcançável a ser lider)
```
    docker node demote <WORKER_HOSTNAME>
```  

## Docker Swarm: deixe o (saia do) cluster worker
Dentro da máquina (worker), que se deseja sair do cluster:
```
    docker swarm leave
```  
> Só é possível sair da forma acima, caso o node não seja um "leader" ou "reachable"

**OBS**
O node Manager só pode ser liberado forçadamente:  

## Docker Swarm: deixe o (saia do) cluster manager
Dentro da máquina (worker), que se deseja sair do cluster:
```
    docker swarm leave --force
```  

## Docker Swarm: pegue o token referente ao manager
Dentro da máquina principal (leader):
```
    docker swarm join-token manager
```  

## Docker Swarm: pegue o token referente ao worker
Dentro da máquina principal (leader):
```
    docker swarm join-token worker
```  

## Docker Swarm: crie (rode) e pegue o token referente ao manager
Dentro da máquina principal (leader):
```
    docker swarm join-token --rotate manager
```  

## Docker Swarm: crie (rode) e pegue o token referente ao worker
Dentro da máquina principal (leader):
```
    docker swarm join-token --rotate worker
```  

## Docker Swarm Node: pause um node
Node não pode receber mais nenhum container
```
    docker node update --availability pause <WORKER_HOSTNAME>
```  

## Docker Swarm Node: ative um node
Node voltou a poder receber containeres
```
    docker node update --availability active <WORKER_HOSTNAME>
```  

## Docker Swarm Node: impessa um node de receber container
Node não pode conter nenhum container
```
    docker node update --availability drain <WORKER_HOSTNAME>
```  

---

# 32
## Docker (Swarm) Service
O Swarm, enquanto cluster, trás uma forma de se ter reziliência
em relação ao nós, da onde esses containeres serão executados.  

No caso do **service**, essa é uma forma de ter reziliência
nos containeres.  

## Docker (Swarm) Service, o que é?
É uma forma de se ter diversos containeres,
respondedo a um determinado serviço.  

Realiza todo balancemanto de carga de dentro ou de fora do cluster,
para os containeres.  

### Docker Swarm: comandos
```
    docker service --help

      create      Cria um novo service
      inspect     Exiba informações detalhadas de um ou mais services
      logs        Exiba os logs dos services ou tasks
      ls          Liste os services
      ps          Liste as tasks de um ou mais services
      rm          Remova um or mais services
      rollback    Reverta as mudanças nas configurações dos services
      scale       Escale um ou multiplas réplicas dos services
      update      Atualize o service

    docker service <COMMAND> --help
```  


### Docker (Swarm) Service: create
Comando que permite criar um novo serviço.  
> --name     == nome do serviço.  
> --replicas == número de réplicas, containeres utilizados.  
> --publish || -p HOST_PORT:CONTAINER_PORT  == qualquer IP do cluster redireciona ao container.  
```
    docker service create --name <SERVICE_NAME> --replicas <NUMBER_REPLICAS> --publish <HOST_PORT:CONTAINER_PORT> <IMAGE>
```  

> --name         == nome do serviço.  
> --replicas     == número de réplicas, containeres utilizados.  
> --publish || -p HOST_PORT:CONTAINER_PORT  == qualquer IP do cluster redireciona ao container.  
> --mount        == ponto de montagem
> --hostname     == nome do host
> --limit-cpu    == configuração de uso máximo de cpus
> --limit-memory == configuração de uso máximo de memória
> --env          == variável de sistema
> --dns          == configuração de dns usada pela máquina
```
    docker service create 
    --name <SERVICE_NAME> 
    --replicas <NUMBER_REPLICAS> 
    --publish <HOST_PORT:CONTAINER_PORT> 
    --mount type=volume,src=<VOLUME_NAME>,dst=<DIR_IN_CONTAINER>
    --hostname <HOSTNAME>
    --limit-cpu <CPU_NUMBER>
    --limit-memory <MEMORY_NUMBER>
    --env <VARIABLE>=<VALUE>
    --dns <DNS>
    <IMAGE>
```  

### Docker (Swarm) Service: ls
Liste as informações dos seviços na máquina.  
```
    docker service ls
```  

### Docker (Swarm) Service: ps
Liste os locais onde roda esse serviço.  
```
    docker service ps <SERVICE_NAME>
```  

### Docker (Swarm) Service: inspect
Inspecione informações sobre service.  
> --pretty == Exiba a saída em formato diferente de JSON.  
```
    docker service inspect <SERVICE_NAME> --pretty
```  
**OBS**
PublishMode = ingress  
tipo de rede (network) que consegue comunicar,
entre os containeres da mesma rede.  

### Docker (Swarm) Service: scale
Escale a quantidade de containres, para mais (up) ou menos (down).  
```
    docker service scale <SERVICE_NAME>=<SERVICE_NUMBER>
```  

### Docker (Swarm) Service: logs
Exiba os logs do service (de cada container).  
> -f == mostra o final do logs.  
```
    docker service logs -f <SERVICE_NAME>
```  

### Docker (Swarm) Service: rm
Remova um service.  
```
    docker service rm <SERVICE_NAME>
```  

---

# 33 e 34
## Docker (Swarm) com volumes
Pode-se montar um diretório em mais de um nó dentro do cluster.  

Para isso, será instalado um servidor que conectará os nós do cluster.  

**OBS**
O Cluster já exite.  

### Nó Server (Principal)
Cria volume.  
```
    docker create volume <VOLUME_NAME>
```  

Apague o diretório referente ao volume.  
```
    rm -rf /var/lib/docker/volumes/<VOLUME_NAME>/_data/
```  

Agore, será instalado o **nfs-server**:  
```
    apt install nfs-server
```  

Em seguida, acesse o arquivo que estrutura as exportações.  
```
    vim /etc/exports
```  

Dentro do arquivo, declare o que será exportado.  
```
    /var/lib/docker/volumes/<VOLUME_NAME> *(rw,sync,subtree_check)
    /opt/site *(rw,sync,subtree_check)
```  

Dê o comando para a realização das exportações.  
```
    exportfs -ar
```  

Crie o diretótio para a exportação:  
```
    mkdir -p /opt/site/_data
```  

Crie o arquivo a ser exportado:  
```
    echo "COLUQE UMA INFORMAÇÂO" > /opt/site/_data/index.html
```  

Criação de link simbólico entre o diretório local e o dentro do container:  
```
    ln -s /opt/site/_data/ /var/lib/docker/volumes/<VOLUME_NAME>/
```  

Comando que permite criar um novo serviço.  

**OBS**
Deve ser executado após as configurações terem sido realizadas dentro do SERVER e do CLIENT.  

> --name     == nome do serviço.  
> --replicas == número de réplicas, containeres utilizados.  
> --publish || -p HOST_PORT:CONTAINER_PORT  == qualquer IP do cluster redireciona ao container.  
```
    docker service create 
    --name <SERVICE_NAME> 
    --replicas <NUMBER_REPLICAS> 
    --publish <HOST_PORT:CONTAINER_PORT>
    --mount type=volume,src=<VOLUME_NAME>,dst=<DIR_IN_CONTAINER>
    <IMAGE>
```  

#### Exemplo com Nginx
```
    docker service create 
    --name giropops
    --replicas 3
    --publish 8080:80
    --mount type=volume,src=giropops,dst=/usr/share/nginx/html
    nginx
```  

### Nó Client
Primeiro, será instalado o **nfs-common**:  
```
    apt install nfs-common
```  

Em seguida, verifique se consegue se conectar à máquina SERVER.  
```
    showmount -e <IP_SERVER>
```  

Monte o diretório do SERVER dentro CLIENT.  
```
    mount <IP_SERVER>:<DIR_IN_SERVER> <DIR_IN_CLIENT>
```  

#### Exemplo
```
    mount 172.31.19.54:/opt/site/ /var/lib/docker/volumes/<VOLUME_NAME>
        ou
    mount 172.31.19.54:/opt/site/ /var/lib/docker/volumes/giropops
```  

### Design acima
```
                 _______                ____________________________
    o           |       |              |                            |
   /|\ -------> |       | <----------> | PORT 8080  | Cluster Swarm |
   / \          |_______|              |____________________________|
  USER          NAVEGADOR                           │
                                                    │
                                                    │
                                          ┌─────────┘
     _____________________________________│_________________________________________
    |                                                                               |
    |                           GIROPOPS      SERVICE                               |
    |_______________________________________________________________________________|
                                          │
                                          │
          ┌───────────────────────────────│────────────────────────────────────┐
     _____|______                    _____|______                         _____|______
    |            |                  |            |                       |            |
    |  NGINX  1  |                  |  NGINX  2  |                       |  NGINX  3  |
    |____________|                  |____________|                       |____________|
          |                               |                                   |
          |                               └─────────────┐                     └──────────────┐
     _____|__________________________        ___________|_____________________     __________|______________________
    |nfs                             |      |nfs                              |   |nfs                              |
L-->|/var/lib/docker/volumes/giropops|      |/var/lib/docker/volumes/giropops |   |/var/lib/docker/volumes/giropops |
i   |________________________________|      |_________________________________|   |_________________________________|
n                   |                                        |                                      |
k                   |                                        |                                      |
  -s         _______|_______                          _______|_______                        _______|_______
S           |    HOSTNAME   |                        |    HOSTNAME   |                      |    HOSTNAME   |
i  n        |       1       |                        |       2       |                      |       3       |
m  l        |_______ _______|                        |_______ _______|                      |_______ _______|
b                   |                                        |                                      |
ó                   |                                       /                                      /
l                   |                                      /                                      /
i            _______|_______                              /                                      /
c           |nfs            | ____________NFS____________| ________________NFS__________________|
o-----------|/opt/site      |
            |_______________|
```  

---

# 35
## Docker (Swarm) service com rede
> Rede overlay, comunicação intra-pods (entre os containeres) dentro de uma rede.  
```
    docker swarm init --advertise-addr=<IP>   # Crie um cluster, caso não exista

    docker network create -d overlay <NETWORK_NAME>

    docker service create --name <SERVICE_NAME> --publish <CONTAINER_PORT_1:HOST_PORT> --network <NETWORK_NAME> <IMAGE>
    docker service create --name <SERVICE_NAME> --publish <CONTAINER_PORT_2:HOST_PORT> --network <NETWORK_NAME> <IMAGE>

    docker container exec -it <CONTAINER_ID>
    curl <SERVICE_NAME>
    CTRL+P
    docker service inspect <CONTAINER_NAME> --pretty

    # docker service update --network-add <NETWORK_NAME> <CONTAINER_NAME>
```  

---

# 36
## Docker secrets
Informações sensíveis guardadas dentro de um container.  

### Local dos Secrets
> O local dos secrets aqui citados, refere-se a um diretório dentro do container.  
```
    /run/secrets
```

### Docker secret: create
```
    docker secret create <SECRET_NAME>
    echo -n "<INFORMAÇÂO>" | docker secret create <SECRET_NAME> -
```  

### Docker secret: ls
```
    docker secret ls
```  

### Docker secret: inspect
```
    docker secret inspect <SECRET_NAME>
```  

### Docker secret: service + secret
```
    docker service create
    --name <SERVICE_NAME>
    --publish <HOST_PORT:CONTAINER_PORT>
    --secret <SECRET_NAME>
    <IMAGE>
```  

#### Exemplo de docker secret: service + secret
```
    docker service create
    --name nginx1
    --publish 8080:80
    --secret meu_arquivo
    nginx
```  

### Docker secret: service + secret
```
    docker service create
    --name <SERVICE_NAME>
    --publish <HOST_PORT:CONTAINER_PORT>
    --secret src=<SECRET_NAME>,target=<CONTAINER_SECRET_NAME>,uid=<USER_ID>,gid=<GROUP_ID>,mode=<PERMISSIOS>
    <IMAGE>
```  

#### Exemplo de docker secret: service + secret
```
    docker service create
    --name nginx2
    --publish 8081:80
    --secret src=meu_arquivo,target=meu_secret_dentro_do_container,uid=200,gid=200,mode=0400
    nginx
```  


### Docker secret: add in service
```
    docker service update --secret-add <SECRET_NAME> <IMAGE>
```  

### Docker secret: rm in service
```
    docker service update --secret-rm <SECRET_NAME> <IMAGE>
```  

---

# 40, 41 e 42

## Docker compose
Ferramenta que permite realizar o deploy de uma imagem pronta.  

### Documentação do compose
```
    https://docs.docker.com/compose/compose-file/
```  

### Docker compose: criação de arquivo
```
    vi docker-compose.yml
```  

### Docker compose: nginx
```yml
version: "3.7"                      # versão do compose utilizada
services:                           # definição dos serviços a serem criados
  web:                              # service a ser criado (informações sobre o container)
    image: nginx                    # imagem utilizada pelo service web
    deploy:                         # ao subir o service, faça
      replicas: 5                   # número de réplicas criadas
      resources:                    # atributos utilizados
        limits:                     # os limites especificados
          cpus: "0.1"               # quantidade de cpus utilizadas: 10% de 1 core (núcleo)
          memory: 50M               # quantidade de memória usada
      restart_policy:               # política de restart do container
        condition: on-failure       # condição para o restart: sempre em caso de falha
    ports:                          # porta
    - "8080:80"                     # quem bater na porta 8080, é redirecionado para a porta 80
    networks:                       # rede na qual este service rodará
    - webserver                     # nome da rede
networks:                           # definição das rede a serem criadas
  webserver:                        # rede a ser criada (informações sobre a rede) 
```  

### Docker compose: MySQL e WordPress
```yml
version: "3.7"                              # versão do compose utilizada
services:                                   # definição dos serviços a serem criados
  db:                                       # service a ser criado (informações sobre o container)
    image: mysql:5.7                        # imagem utilizada pelo service mysql
    volumes:                                # declaração do volume a ser utilizado
    - db_data:/var/lib/mysql                # o volume "db_data" será montado em "/var/lib/mysql"
    environment:                            # variáveis de sistema
      MYSQL_ROOT_PASSWORD: somewordpress    # password do usuário root do wordpress
      MYSQL_DATABASE: wordpress             # banco de dados usado
      MYSQL_USER: wordpress                 # usuário dentro do mysql
      MYSQL_PASSWORD: wordpress             # password do usuário declarado acima

  wordpress:                                # service a ser criado (informações sobre o container)
    depends_on:                             # declara que depende de outro service
    - db                                    # este service depende do service "db": primeiro sobe o "db" depois "wordpress"
    image: wordpress:latest                 # imagem utilizada pelo service wordpress
    ports:                                  # porta
    - "8080:80"                             # quem bater na porta 8080, é redirecionado para a porta 80
    environment:                            # variáveis de sistema
      WORDPRESS_DB_HOST: db:3306            # "db", nome do service; "3306", porta do mysql
      WORDPRESS_DB_USER: wordpress          # usuário dentro do mysql
      WORDPRESS_DB_PASSWORD: wordpress      # password do usuário declarado no service acima
volumes:                                    # definição dos volumes a serem criados
  db_data:                                  # volume a ser criado (informações sobre o volume) 
```  

### Docker compose: nginx e visualizer
```yml
version: "3.7"                                      # versão do compose utilizada
services:                                           # definição dos serviços a serem criados
  web:                                              # service a ser criado (informações sobre o container)
    image: nginx                                    # imagem utilizada pelo service web
    deploy:                                         # ao subir o service, faça
      placement:                                    # local no qual o service vai subir
        constraints:                                # restrição
        - node.labels.dc == UK                      # label chamado "dc" que dever ser igual a "UK"
      replicas: 5                                   # número de réplicas criadas
      resources:                                    # atributos utilizados
        limits:                                     # os limites especificados
          cpus: "0.1"                               # quantidade de cpus utilizadas: 10% de 1 core (núcleo)
          memory: 50M                               # quantidade de memória usada
      restart_policy:                               # política de restart do container
        condition: on-failure                       # condição para o restart: sempre em caso de falha
    ports:                                          # porta
    - "8080:80"                                     # quem bater na porta 8080, é redirecionado para a porta 80
    networks:                                       # rede na qual este service rodará
    - webserver                                     # nome da rede

  visualizer:                                       # service a ser criado (informações sobre o container)
    image: dockersample/visualizer:stable           # imagem utilizada pelo service web
    ports:                                          # porta
    - "8888:8080"                                   # quem bater na porta 8888, é redirecionado para a porta 8080
    volumes:                                        # definição dos volumes a serem criados
    - "/var/run/docker.sock:/var/run/docker.sock"   # monta o socket do docker "/var/run/docker.sock", dentro deste service
    deploy:                                         # ao subir o service, faça
      placement:                                    # local no qual o service vai subir
        constraints: [node.role == manager]         # restrição: rode apenas no manager
    networks:                                       # rede na qual este service rodará
    - webserver                                     # nome da rede

networks:                                           # definição das rede a serem criadas
  webserver:                                        # rede a ser criada (informações sobre a rede) 
```  


### Docker compose: deploy
```
    docker stack deploy -c <COMPOSE_FILE> <STACK_NAME>
```  

#### Exemplo de docker compose: deploy
```
    docker stack deploy -c docker-compose.yml giropops
```  

**OBS**
Caso já exista um node similar ao criado acima - como em "nginx e visualizer" -,
mas que não possua o label referido abaixo da "constraints":  
node.labels.dc == UK  
atualize o label.  

Veja abaixo:  
#### Exemplo de docker compose: deploy
```
    # docker node update --label-add <LABEL>=<VALUE> <MANAGER_NODE>
    docker node update --label-add dc=UK <MANAGER_NODE>
```  

## Docker stack
Docker stack, é o comando para a realização de deploy dos docker-compose.  

Quando se trata de stack, diz-se sobre todos os componentes para realizar
o deploy da aplicação.  

**OBS**
Um stack, é a reunição de um ou vários services.  
Um service, é a reunição de um ou vários containeres.  
```
┌─────────────────────────────────────────────────────────────────┐
│                              STACK                              │
│  ┌───────────────────────────┐   ┌───────────────────────────┐  │
│  │          SERVICE          │   │          SERVICE          │  │
│  │ ┌─────────┐   ┌─────────┐ │   │ ┌─────────┐   ┌─────────┐ │  │
│  │ │CONTAINER│   │CONTAINER│ │   │ │CONTAINER│   │CONTAINER│ │  │
│  │ └─────────┘   └─────────┘ │   │ └─────────┘   └─────────┘ │  │
│  └───────────────────────────┘   └───────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

No exemplo acima, o arquivo possui um service e 5 containeres.  

### Docker (compose) stack: deploy
```
    docker stack deploy -c <COMPOSE_FILE> <STACK_NAME>
```  

### Docker (compose) stack: ls
> liste as stacks.  
```
    docker stack ls
```  

### Docker (compose) stack: ps
> liste as informações sobre as stacks.  
```
    docker stack ps <SERVICE_NAME>
```  

#### Exemplo de docker (compose) stack: ps
> liste as informações sobre as stacks.  
```
    docker stack ps giropops
```  

### Docker (compose) stack: services
> liste os services abaixo desta stack.  
```
    docker stack services <SERVICE_NAME>
```  

#### Exemplo de docker (compose) stack: services
> liste os services abaixo desta stack.  
```
    docker stack services giropops
```  

---

# 43

### Docker compose: redis, postgres, vote e visualizer
```yml
version: "3.7"                                              # versão do compose utilizada
services:                                                   # definição dos serviços a serem criados
  redis:                                                    # service a ser criado (informações sobre o container)
    image: redis:alpine                                     # imagem utilizada pelo service redis
    ports:                                                  # porta
    - "6379"                                                # 6379, cluster ip (uso interno do cluster)
    networks:                                               # rede na qual este service rodará
    - frontend                                              # nome da rede
    deploy:                                                 # ao subir o service, faça
      replicas: 2                                           # número de réplicas criadas
      update_config:                                        # como funcionará este deploy, caso ocorra alguma atualização
        parallelism: 1                                      # faça as atualizações em paralelo
        delay: 10s                                          # com 10 segundos de delay, entre cada réplica
        order: start-first                                  # start os containeres novos
      rollback_config:                                      # caso qualquer configuração dê errado
        parallelism: 1                                      # faça as atualizações em paralelo
        delay: 10                                           # com 10 segundos de delay, entre cada réplicas
        failure_action: continue                            # falhou rollback, continue o service
        monitor: 60s                                        # monitore o rollback
        order: stop-first                                   # stop os containeres atuais
      restart_policy:                                       # política de restart do container
        condition: on-failure                               # condição para o restart: sempre em caso de falha

  db:                                                       # service a ser criado (informações sobre o container)
    image: postgres:9.4                                     # imagem utilizada pelo service db
    volumes:                                                # definição dos volumes a serem criados
    - db-data:/var/lib/postgresql/data                      # monta o volume "db-data", dentro deste service
    networks:                                               # rede na qual este service rodará
    - backend                                               # nome da rede
    deploy:                                                 # ao subir o service, faça
      placement:                                            # local no qual o service vai subir
        constraints: [node.role == manager]                 # restrição: rode apenas no manager

  vote:                                                     # service a ser criado (informações sobre o container)
    image: dockersamples/examplevotingapp_vote:before       # imagem utilizada pelo service vote
    ports:                                                  # porta
    - 5000:80                                               # quem bater na porta 5000, é redirecionado para a porta 80
    networks:                                               # rede na qual este service rodará
    - frontend                                              # nome da rede
    depends_on:                                             # declara que depende de outro service
    - redis                                                 # este service depende do service "redis": primeiro sobe o "redis" depois "vote"
    depĺoy:                                                 # ao subir o service, faça
      replicas: 2                                           # número de réplicas criadas
      update_config:                                        # como funcionará este deploy, caso ocorra alguma atualização
        parallelism: 2                                      # faça as atualizações em paralelo
      restart_policy:                                       # política de restart do container
        condition: on-failure                               # condição para o restart: sempre em caso de falha

  result:                                                   # service a ser criado (informações sobre o container)
    image: dockersamples/examplevotingapp_result:before     # imagem utilizada pelo service result
    ports:                                                  # porta
    - 5001:80                                               # quem bater na porta 50001, é redirecionado para a porta 80
    networks:                                               # rede na qual este service rodará
    - backend                                               # nome da rede
    depends_on:                                             # declara que depende de outro service
    - db                                                    # este service depende do service "db": primeiro sobe o "db" depois "result"
    deploy:                                                 # ao subir o service, faça
      replicas: 1                                           # número de réplicas criadas
      update_config:                                        # como funcionará este deploy, caso ocorra alguma atualização
        parallelism: 2                                      # faça as atualizações em paralelo
        delay: 10s                                          # com 10 segundos de delay, entre cada réplicas
      restart_policy:                                       # política de restart do container
        condition: on-failure                               # condição para o restart: sempre em caso de falha

  worker:                                                   # service a ser criado (informações sobre o container)
    image: dockersamples/examplevotingapp_worker            # imagem utilizada pelo service worker
    networks:                                               # rede na qual este service rodará
    - frontend                                              # nome da rede
    - backend                                               # nome da rede
    deploy:                                                 # ao subir o service, faça
      mode: replicated                                      # modo do deploy, quantidade de réplicas de determinado service
      replicas: 1                                           # número de réplicas criadas
      labels: [APP=VOTING]                                  # label chamado "APP" que dever ser igual a "VOTING"
      restart_policy:                                       # política de restart do container
        condition: on-failure                               # condição para o restart: sempre em caso de falha
        delay: 10s                                          # com 10 segundos de delay, entre cada réplica
        max_attempts: 3                                     # tente no máximo 3 vezes realizar o restar
        window: 120s                                        # numa janela de 120 segundos
      placement:                                            # local no qual o service vai subir
        constraints: [node.role == manager]                 # restrição: rode apenas no manager

  visualizer:                                               # service a ser criado (informações sobre o container)
    image: dockersamples/visualizer:stable                  # imagem utilizada pelo service visualizer
    ports:                                                  # porta
    - "8080:8080"                                           # quem bater na porta 8080, é redirecionado para a porta 8080
    stop_grace_period: 1m30s                                # período de tempo para morrer, depois kill -9 e tcháu
    volumes:                                                # definição dos volumes a serem criados
    - "/var/run/docker.sock:/var/run/docker.sock"           # monta o socket do docker "/var/run/docker.sock", dentro deste service
    deploy:                                                 # ao subir o service, faça
      placement:                                            # local no qual o service vai subir
        constraints: [node.role == manager]                 # restrição: rode apenas no manager

networks:                                                   # definição das redes a serem criadas
  frontend:                                                 # rede a ser criada (informações sobre a rede)
  backend:                                                  # rede a ser criada (informações sobre a rede)

volumes:                                                    # definição dos volumes a serem criados
  db-data:                                                  # volume a ser criado (informações sobre o volume) 
```  

### Design acima
```
         ____________          ____________
        |            |        |            |
        | VOTING-APP |        | RESULT-APP |
        |   Python   |        |   Node.js  |
        |____________|        |____________|
             /                      \
            /                        \
           /                          \
    ______/__                        __\_________
   |         |                      |            |
   |  REDIS  |                      |     DB     |
   |  Redis  |                      | PostgreSQL |
   |_________|                      |____________|
              \                    /
               \                  /
                \                /
                 \______________/
                 |              |
                 |    WORKER    |
                 |     .net     |
                 |______________|
```  

### Fluxo do exposto acima
Vote:
* cachorro
* gato

Escreve do Redis.  

Worker: consome Redis e salva no PostgreSQL.  

Result: consome do PostgreSQL.  

```
                ____________  VOTING-APP  ___________
    o    Voto  |            |  salva no  |           |
   /|\ ────────| VOTING-APP |────────────|   REDIS   |
   / \         |   Python   |            |   redis   |
               |____________|            |___________|
                                              │
                                    Worker lê │
                                  ┌───────────┘
                         _________│__________
                        |                    |
                        |       WORKER       |
                        |        .net        |
                        |____________________|
                                  │ Worker salva no
                                  └────────────────┐
                                                   │
               ____________  RESULT-APP  __________│____
              |            | consome do |               |
              | RESULT-APP |────────────|      DB       |
              |   Node.js  |            |  PostgreeSQL  |
              |____________|            |_______________|
```  

**OBS**
Worker é pertencente as duas redes (frontend e backend),
portanto, funciona nas duas partes do site.  

### Docker (compose) stack: deploy
```
    docker stack deploy -c <COMPOSE_FILE> <STACK_NAME>
```  

---

# 44

### Docker compose: redis, postgres, vote e visualizer
```yml
version: "3.7"                                               # versão do compose utilizada
services:                                                    # definição dos serviços a serem criados
  prometheus:                                                # service a ser criado (informações sobre o container)
    image: linuxtips/prometheus_alpine                       # imagem utilizada pelo service worker
    volumes:
      - ./conf/prometheus/:etc/prometheus                    # monta (esta configuração) este volume, dentro deste service
      - prometheus_data:/var/lib/prometheus                  # monta volume que guardará dados do prometheus
    networks:                                                # rede na qual este service rodará
      - backend                                              # nome da rede
    ports:                                                   # porta
      - 9090:9090                                            # quem bater na porta 9090, é redirecionado para a porta 9090

  node-exporter:                                             # service a ser criado (informações sobre o container)
    image: linuxtips/node-exporter_alpine                    # imagem utilizada pelo service worker
    hostname: '{{.NODE.ID}}'                                 # nome da máquina
    volumes:                                                 # definição dos volumes a serem criados
      - /proc:/usr/proc                                      # monta o volume "/proc", dentro deste service
      - /sys:/usr/sys                                        # monta o volume "/sys", dentro deste service
      - /:/rootfs                                            # monta o volume "/", dentro deste service
    deploy:                                                  # ao subir o service, faça
      mode: global                                           # modo do deploy, quantidade de réplicas de determinado service: global, exite um desse em cada nó
    networks:                                                # rede na qual este service rodará
      - backend                                              # nome da rede
    ports:                                                   # porta
      - 9100:9100                                            # quem bater na porta 9100, é redirecionado para a porta 9100

  alertmanager:                                              # service a ser criado (informações sobre o container)
    image: linuxtips/alertmanager_alpine                     # imagem utilizada pelo service worker
    volumes:                                                 # definição dos volumes a serem criados
      - ./conf/alertmanager/:/etc/alertmanager               # monta (esta configuração) este volume, dentro deste service
    networks:                                                # rede na qual este service rodará
      - backend                                              # nome da rede
    ports:                                                   # porta
      - 9093:9093                                            # quem bater na porta 9093, é redirecionado para a porta 9093

  cadvisor:                                                  # service a ser criado (informações sobre o container)
    image: google/cadvisor                                   # imagem utilizada pelo service worker
    hostname: '{{.NODE.ID}}'                                 #
    volumes:                                                 # definição dos volumes a serem criados
      - /:/rootfs:ro                                         # monta o volume "/" dentro deste service, apenas como read-only
      - /var/run/:/var/run:rw                                # monta o volume "/var/run" dentro deste service, read-write
      - /sys:/sys:ro                                         # monta o volume "/sys" dentro deste service, apenas read-only
      - /var/lib/docker/:/var/lib/docker.sock:ro             # monta o socket do docker "/var/run/docker.sock" dentro deste service, apenas como read-only
    networks:                                                # rede na qual este service rodará
      - backend                                              # nome da rede
    deploy:                                                  # ao subir o service, faça
      mode: global                                           # modo do deploy, quantidade de réplicas de determinado service: global, exite um desse em cada nó
      - backend                                              # nome da rede
    ports:                                                   # porta
      - 8080:8080                                            # quem bater na porta 8080, é redirecionado para a porta 8080

  grafana:                                                   # service a ser criado (informações sobre o container)
    image: google/cadvisor                                   # imagem utilizada pelo service worker
    depends_on:                                              # declara que depende de outro service
    - prometheus                                             # este service depende do service "prometheus": primeiro sobe o "db" depois "result"
    # volumes:                                                 # definição dos volumes a serem criados
    #   - ./conf/grafana/grafana.db/:/grafana/data/grafana.db  # monta (esta configuração) este volume, dentro deste service (não é uma boa prática)
    env_file:                                                # arquivo de configuração
      - grafana.config                                       # armazena senha do usuário
    networks:                                                # rede na qual este service rodará
      - frontend                                             # nome da rede
      - backend                                              # nome da rede
    ports:                                                   # porta
      - 3000:3000                                            # quem bater na porta 3000, é redirecionado para a porta 3000

networks:                                                   # definição das redes a serem criadas
  frontend:                                                 # rede a ser criada (informações sobre a rede)
  backend:                                                  # rede a ser criada (informações sobre a rede)

volumes:                                                    # definição dos volumes a serem criados
  prometheus_data:                                          # volume a ser criado (informações sobre o volume) 
```  

**OBS**
Quando se deseja monitorar o host (hostname) e esse host é baseado em container,
a montagem de volume acontecerá a partir do barra:  
```
    volumes:                                                 # definição dos volumes a serem criados
      - /proc:/usr/proc                                      # monta o volume "/proc", dentro deste service
      - /sys:/usr/sys                                        # monta o volume "/sys", dentro deste service
      - /:/rootfs                                            # monta o volume "/", dentro deste service
```  

#### Grafana config
No mesmo nível em que se encontra o **docker-compose.yml**,
crie o arquivo **grafana.config**:  
```
    touch grafana.config
```  
E dentro dele, insira o seguinte:  
```
GF_SECURITY_ADMIN_PASSWORD=giropops
GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-piechart-panel,comptocamp-prometheus-alertmanager-datasource,vonage-status-panel
```  

### Configs Files
Do mesmo diretório do nível de **docker-compose.yml**,
crie o seguinte diretório:  
```
mkdir -p conf
```  
Dentro do diretório recém criado, crie os seguintes diretórios:  
```
mkdir -p alertmanager prometheus # grafana 
```  

#### Configs Files: alertmanager
Dentro do diretório de nome **alertmanager**, recentemente criado,
crie o seguinte arquivo:  
```
touch config.yml
```  
Preencha-o com o seguinte:  
```
route:
  receiver: 'slack'

receivers:
  - name: 'slack'
    slack_configs:
      - send_resolved: true
        username: 'jeferson'
        channel: '#training'
        api_url: 'https://hooks.slack.com/services/<SIG>/<SIG>'
```  

#### Configs Files: prometheus
Dentro do diretório de nome **prometheus**, recentemente criado,
crie os seguintes arquivos:  
```
touch alert.rules prometheus.yml
```  
Preencha o arquivo **alert.rules** com o seguinte:  
```
groups:
- nome: exampĺe
  rules: 


  - alert: service_down
    expr: up == 0
    for: 2m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} down"
      description: "{{ $labels.instance }} of job {{ $labels.job}} has been down for more than 2 minutes."

  - alert: high_load
    expr: node_load1 > 0.5
    for: 2m
    labels:
      severity: page
    annotations:
      summary: "Instance {{ $labels.instance }} under high load"
      description: "{{ $labels.instance }} of job {{ $labels.job}} is under high load."
```  

Preencha o arquivo **prometheus.yml** com o seguinte:  
```
global:
  scrape_interval: 10s
  evaluation_interval: 10s
  external_labels:
    monitor: 'giropops-monitoring'
rules_files:
  - 'alert.rules'
alerting:
  alertmanagers:
  - scheme: http
    static_configs:
    - targets:
      - "alertmanager:9093"

scrape_configs:
  - job_name: 'node'
    scrape_interval: 5s
    static_configs:
      - targets: ['localhost:9090', 'cadvisor:8080', 'node-exporter:9100']
  - job_name: 'netdata'
    metrics_path: '/api/v1/allmetrics'
    params:
      format: [prometheus]
    hornor_labels: true
    scrape_interval: 5s
    static_configs:
      - targets: ['YOUR_NETDATA_IP:19999']
```  

### Docker (compose) stack: deploy
```
    docker stack deploy -c <COMPOSE_FILE> <STACK_NAME>
```  
