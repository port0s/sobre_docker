## Cria container: com alias
```
alias mix='docker container run -it elixir mix'
mix new atol
```
## Cria container: sem alias
```
docker container run -it elixir mix mix new atol
```

## Inicia container
```
docker container start ${CONTAINER_ID}
```

## Executa comando e acessa container
```
docker container exec -it ${CONTAINER_ID} /bin/bash
```
