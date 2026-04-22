# Comando "help" para volumes
    docker volume 

# Listar volumes
    docker volume ls

# Criar volme (dentro do seu servidor Docker)
    Utilizamos volume para armazenar dados de maneira permanente no Docker
    docker volume create "nome"

# Informacoes detalhadas do volume
    docker volume inspect <nomeDoVolume>

# Comando "help" para redes
    docker network

# Criar rede (dentro do seu servidor Docker)
    docker network create rede_fatec

    e.g.
    docker run -it --network rede_fatec alpine sh

# Analisar rede
    docker network inspect <nomeDaRede>

## DOCKER COMPOSE!! :)

# Criar configuracao de container com arquivo docker-compose
    docker compose up

# Delecao de rede e container (por padrao)
    docker compose down

