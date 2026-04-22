## DOCKER!! :)

# Versao do cliente do Docker
    docker --version 

# Versao do cliente E do servidor do Docker (+ info)
    (sudo) docker version

# Tags em Docker
    Versionamento e organizacao de imagens do Docker. Um hash unico 
    que podemos referenciar de diferentes maneiras, criando tags: 
    para ficar mais legivel, entrar em um padrao etc.
    Podemos usar tanto letras quanto numeros. Eh uma alias:

    e.g.
    3.23.3 <- 3.23 <- 3 <- latest

# Ver quais containeres estao rodando na maquina com parametros
    docker ps

    docker ps -a (TUDO, inclusive containeres que morreram)

# Puxar imagens do DockerHun
    docker pull 

# Executar algo (imagem, container)
    docker run

    # e.g. (rodamos este container SOMENTE para dar um echo no terminal, dps ele morre)
    docker run alpine echo "ola mundo"

    temos o log dessa operacao em docker ps -a

    # e.g.2 (acessamos o root do container no proprio terminal)
    docker run -it alpine sh

    -it  -> interativo
    sh -> rodar em shell

# Buildar imagem customizada
    docker build .
    Builda o Dockerfile do diretorio que estamos atualmente

# Adicionar tag a imagem do Docker
    docker tag <hash> <tag>
    ou
    docker tag <hash> <tag>:<tag>

# Adicionar nome a imagem do Docker
    docker run --name <nome>

# Particulas

-d -> O container roda independente do terminal (detach ou desanexar do terminal)

rm -> remove containeres

rmi -> remove imagens 

stop -> para containeres 

-p -> mapear porta: <host>:<container>

# Expondo porta de um nginx
    docker run --name web1 -d -p 8080:80 nginx:1.29

    -v -> volume
    :ro -> read only
    :rw -> reand and write 

    e.g. (expondo um arquivo html)
    docker run --name web2 -v <caminho do arquivo>:<arquivo dentro do container>:ro -d -p 8081:80 nginx:1.29