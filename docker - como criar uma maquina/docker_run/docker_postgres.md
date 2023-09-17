# COMANDO PARA RODAR O POSTGRES SQL
  * run = rode este container
  * --name = pg (flag) = nome atribuído ao container
  * --restart = always (flag) = quando o docker for iniciado (servidor cair, crash no sistema etc.) este container necessita ser reiniciado
  * -e = POSTGRES_PASSWORD='123' (flag) = variável de ambiente com senha de acesso
  * -P = 5432:5432 (flag) = porta exposta (aplicação Flask acessa): porta dentro do container (Postegres) 
  * -d = postgres:10 (flag) = rode em backgroud e baixe a imagem do container:versão
```sh
	docker run --name pg --restart always -e POSTGRES_PASSWORD='' -P 5432:5432 -d postgres:10
```

# EXECUÇÂO DO PSQL ATRAVÉS DO MODO CONSOLE INTERATIVO DO DOCKER
  * exec = execute o comando
  * -it  = modo interativo (entre no shell)
  * pg   = nome do container
  * psql = nome do Postegres no container
  * -U   = postegres (flag) = acesse o usuário postegres
```sh
  docker exec -it pg psql -U postregres
```
