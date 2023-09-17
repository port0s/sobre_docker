# Elixir
  * -it       = entre no modo interativo (-i) e acesse o um emulador de terminal (-t)
  * --rm      = remova o container automaticamente, caso já exista
  * --name    = assine um nome ao container
  * --volume  = /<HOST_DIR>:/<CONTAINER_DIR>
  * --workdir = diretório no qual irá rodar o app dentro do container

## Rode o Elixir e já adentre ao IEx
```sh
	docker run -it --rm elixir
```
## Adentre ao IEx (explicitamente), através do Docker
```sh
	docker run -it --rm elixir iex
```
## Execute um script Elixir (run elixir) através do Docker
```sh
	docker run -it --rm --name elixir-inst1 -v "$PWD":/usr/src/myapp -w /usr/src/myapp elixir elixir <SCRIPT>.exs
```
## Compile um script Elixir (compile elixir) através do Docker
```sh
	docker run -it --rm --name elixir-inst1 -v "$PWD":/usr/src/myapp -w /usr/src/myapp elixir elixirc <SCRIPT>.ex
```
## Adentre ao IEx, com o carregamento do script enviado, através do Docker
```sh
	docker run -it --rm --name elixir-inst1 -v "$PWD":/usr/src/myapp -w /usr/src/myapp elixir iex <SCRIPT>.ex
```
## Crie um projeto mix, com o usuário de id 1000 e grupo 1000. O diretório no container conterá leitura e escrita
```sh
    docker run -it --rm --user 1000:1000 --name mix -v "$PWD":/usr/src/myapp:rw -w /usr/src/myapp elixir mix new '
```

# REFERÊNCIAS
[Mix Project](https://www.linkedin.com/pulse/non-root-docker-containers-linas-zvirblis)
