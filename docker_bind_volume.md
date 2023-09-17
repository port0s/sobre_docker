# Criação de um projeto mix localmente ligado ao container
Usamos então a opção -v local_path:container_path, onde ambos os caminhos precisam ser absolutos. Vamos tentar criar um novo projeto de elixir usando `mix new` e ligar a montagem de um diretório local.

```sh
docker container run --rm -v $PWD:/data -w /data elixir:1.7.4 mix new crypto
```



# Extrutura do projeto mix criado localmente
Nós montamos o diretório atual no caminho de /data do container, usando a variável de ambiente `$PWD`. Também adicionamos uma opção -w /data que diz ao container para iniciar o comando no diretório de trabalho /data. Podemos ver que o diretório crypto está agora em nosso diretório atual. Mix.exs

```sh
└── crypto
    ├── lib
    │   └── crypto.ex
    ├── mix.exs
    ├── README.md
    └── test
        ├── crypto_test.exs
        └── test_helper.exs
```



# inserção de dependência localmente 
Vamos mover para dentro do diretório cripto 
```sh
cd crypto
```

e adicionar uma dependência em mistura. exs
```ex
defp deps do
  [ {:poison, "~> 4.0"} ]
end
```

## comando para inserção de dependência
```sh
docker container run --rm -v $PWD:/app -w /app -it elixir:1.7.4 mix deps.get
```

Não conseguiu encontrar Hex, que é necessário para construir dependência :poison
Devo instalar o Hex? (se estiver executando de forma não interativa, use "mix local.hex --force") [Yn] Y.



# crição de volume para
Depois que a dependência é baixada e construída, o container é destruído e todos os dados em /root/.mix está perdida. Se executarmos novamente os mesmos comandos teremos novamente de instalar o hex. Para resolver isso, um problema semelhante, podemos criar um volume local

```sh
docker volume create elixir-mix
```


## uso do volume criado
Para usá-lo, precisamos adicionar outra opção ao comando -v volume-name:container_mount_point.

```sh
docker container run --rm -v elixir-mix:/root/.mix -v $PWD:/app -w /app -it  elixir:1.7.4 mix deps.get # adição das dependências
docker container run --rm -v elixir-mix:/root/.mix -v $PWD:/app -w /app -it  elixir:1.7.4 mix deps.get # teste se as dependências foram adicionadas
```


# execução do iex com dependência instalada
A primeira vez, Hex precisa ser instalado. A segunda vez, o comando está funcionando corretamente porque o hex é encontrado no diretório /root/.mix, onde o volume elixir-mix é montado.

Agora podemos executar nosso iex carregando nosso projeto cripto

```sh
docker container run --rm -v elixir-mix:/root/.mix -v $PWD:/app -w /app -it  elixir:1.7.4 iex -S mix
iex(1)> Poison.encode! %{hello: :world}
```

As dependências são baixadas em deps e compiladas no diretório do projeto de build (compilação). Uma vez que essas mudanças são refletidas em nosso diretório de projeto através da montagem de vinculação, se executarmos o mesmo comando novamente não devemos ter nenhuma compilação de dependências


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

---

# REFERÊNCIAS
[Stack Overflow - Mount a Single File](https://stackoverflow.com/questions/42248198/how-to-mount-a-single-file-in-a-volume)
[Stack Overflow - Mount a Single File](https://stackoverflow.com/questions/54657370/mount-single-file-from-volume-using-docker-compose)
