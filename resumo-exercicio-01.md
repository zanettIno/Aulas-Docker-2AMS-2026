# 📋 Resumo — Exercício 01 de Docker
**Guilherme Xavier Zanetti | 2º ADS AMS | 23/04/2026**

---

## Exercício 1 — Shell Script em container com volume

### O que foi pedido
Criar um script Bash contador (`contador.sh`) e executá-lo dentro de um container Ubuntu usando bind mount, observando a saída em tempo real.

### Resolução

**`contador.sh`**
```bash
#!/bin/bash
i=0
while true; do
  echo $i
  i=$((i+1))
  sleep 1
done
```

```bash
# Subir o container em background montando o script via volume
docker run -d --name contador -v "$PWD/contador.sh:/contador.sh" ubuntu:22.04 bash /contador.sh

# Acompanhar a saída em tempo real
docker logs -f contador
```

### Por quê funciona assim?

- **`-d`** — O container roda em background (detached), liberando o terminal.
- **`-v "$PWD/contador.sh:/contador.sh"`** — Bind mount: espelha o arquivo local dentro do container no path `/contador.sh`. O container lê o arquivo diretamente do host, sem precisar rebuild da imagem.
- **`bash /contador.sh`** — Comando executado ao iniciar o container; o Bash interpreta o script.
- **`docker logs -f`** — A flag `-f` (follow) mantém o stream de logs aberto, mostrando cada número sendo impresso em tempo real, igual a um `tail -f`.

> ⚠️ Se o container já existia com o mesmo nome, o Docker retorna erro de conflito. Solução: `docker rm -f contador` antes de recriar.

---

## Exercício 2 — Container efêmero com `--rm`

### O que foi pedido
Criar um container que imprime o nome no terminal e se auto-remove após a execução.

### Resolução

```bash
docker run --rm ubuntu:22.04 echo "Guilherme Xavier Zanetti"
```

### Por quê funciona assim?

- **`--rm`** — Remove o container automaticamente assim que o processo principal encerra. Sem essa flag, o container ficaria em estado `Exited` visível no `docker ps -a`, acumulando resíduos desnecessários.
- O `echo` é o processo principal: assim que termina, o container morre e é deletado.
- Ideal para tarefas pontuais (one-shot) onde não precisamos manter histórico do container.

---

## Exercício 3 — Comunicação entre containers na mesma rede

### O que foi pedido
Criar uma rede Docker, subir dois containers Nginx nela, criar um container Ubuntu interativo na mesma rede, instalar `curl` e `ping`, e verificar a conectividade pelo nome do container.

### Resolução

```bash
# 1. Criar a rede
docker network create rede_alfa

# 2. Subir dois containers Nginx na rede
docker run -d --name web1 --network rede_alfa nginx
docker run -d --name web2 --network rede_alfa nginx

# 3. Subir container interativo na mesma rede
docker run -it --name testador --network rede_alfa ubuntu:22.04 bash

# 4. Dentro do container — instalar ferramentas
apt update && apt install -y curl iputils-ping

# 5. Testar conectividade por nome
ping -c 3 web2
curl web2
```

### Por quê funciona assim?

- **Redes customizadas têm DNS interno**: containers se localizam pelo **nome** (hostname). Na rede padrão `bridge`, isso não funciona — só por IP.
- **`ping web2`** resolve o nome `web2` para o IP interno que o Docker atribuiu ao container, comprovando alcançabilidade na camada de rede (ICMP).
- **`curl web2`** faz uma requisição HTTP na porta 80, retornando o HTML padrão do Nginx — prova que o serviço está acessível dentro da rede.
- O container `testador` precisa estar **na mesma rede** (`rede_alfa`) para enxergar os outros; do contrário, o DNS interno não resolve.

---

## Exercício 4 — Isolamento entre redes distintas

### O que foi pedido
Criar uma **segunda rede** (`rede_beta`), subir um Nginx nela, subir um container interativo na `rede_beta`, e verificar que **não é possível** acessar containers da `rede_alfa`.

### Resolução

```bash
# 1. Criar segunda rede
docker network create rede_beta

# 2. Subir Nginx na nova rede
docker run -d --name web3 --network rede_beta nginx

# 3. Container interativo na rede_beta
docker run -it --name testador_beta --network rede_beta ubuntu:22.04 bash

# 4. Dentro do container
apt update && apt install -y curl iputils-ping

# Funciona — web3 está na mesma rede
ping -c 3 web3
curl web3

# NÃO funciona — web1 está em outra rede
curl web1
# curl: (6) Could not resolve host: web1
```

### Por quê funciona assim?

- Redes Docker são **namespaces de rede isolados**. Containers em redes diferentes não se enxergam — nem por nome, nem por IP — a menos que sejam explicitamente conectados às duas redes com `docker network connect`.
- O erro `Could not resolve host: web1` acontece porque o DNS interno da `rede_beta` não tem registro do container `web1`, que existe apenas na `rede_alfa`.
- Esse isolamento é um **mecanismo de segurança**: serviços de diferentes aplicações não se comunicam acidentalmente.

---

## Exercício 5 — Banco de dados MariaDB em rede dedicada

### O que foi pedido
Criar uma rede, subir um container MariaDB nela, conectar um container Debian interativo na mesma rede, instalar o cliente MariaDB e executar comandos SQL (CREATE, INSERT, SELECT).

### Resolução

```bash
# 1. Criar rede dedicada ao banco
docker network create rede_banco

# 2. Subir o MariaDB com senha de root via variável de ambiente
docker run -d --name meu_db --network rede_banco \
  -e MARIADB_ROOT_PASSWORD=root \
  mariadb

# 3. Subir container cliente interativo na mesma rede
docker run -it --name cliente_db --network rede_banco debian bash

# 4. Dentro do container Debian
apt update && apt install -y mariadb-client

# 5. Conectar ao servidor (pelo nome do container como host)
mariadb -h meu_db -u root -proot
```

**Comandos SQL executados:**
```sql
CREATE DATABASE fatec;
USE fatec;

CREATE TABLE alunos (
  id   INT AUTO_INCREMENT PRIMARY KEY,
  nome VARCHAR(50)
);

INSERT INTO alunos (nome) VALUES ('Guilherme Zanetti');

SELECT * FROM alunos;
-- Resultado:
-- +----+-------------------+
-- | id | nome              |
-- +----+-------------------+
-- |  1 | Guilherme Zanetti |
-- +----+-------------------+

exit
```

### Por quê funciona assim?

- **`-e MARIADB_ROOT_PASSWORD=root`** — Variáveis de ambiente configuram o container na inicialização. O MariaDB exige essa variável para definir a senha do root; sem ela, o container não sobe corretamente.
- **`-h meu_db`** no cliente mariadb — O hostname `meu_db` é resolvido pelo DNS interno da `rede_banco`, conectando ao servidor sem precisar saber o IP.
- A rede dedicada isola o banco de outros serviços, uma boa prática de segurança: apenas containers explicitamente na `rede_banco` conseguem acessar o MariaDB.
- O `AUTO_INCREMENT` no campo `id` garante que cada insert gera um identificador único automaticamente.

> ⚠️ Se houver containers com o mesmo nome de tentativas anteriores, use `docker rm -f <nome>` para remover o conflito antes de recriar.

---

## Exercício 6 — Docker Compose com Nginx e volume HTML

### O que foi pedido
Criar um `docker-compose.yml` que sobe um Nginx (`nginx:trixie`) com porta `8123:80` e monta uma pasta `html/` local como conteúdo do servidor.

### Resolução

**Estrutura de arquivos:**
```
ex06/
├── docker-compose.yml
└── html/
    └── index.html
```

**`html/index.html`**
```html
<html><body><h1>Trabalho do Zanetti</h1></body></html>
```

**`docker-compose.yml`**
```yaml
services:
  web-guilherme:
    image: nginx:trixie
    container_name: web-guilherme
    ports:
      - "8123:80"
    volumes:
      - ./html:/usr/share/nginx/html
```

```bash
# Subir o ambiente
docker compose up -d

# Verificar no navegador: http://localhost:8123

# Derrubar e limpar
docker compose down
```

### Por quê funciona assim?

- **`docker-compose.yml`** é uma forma **declarativa** de definir a infraestrutura. Em vez de um longo `docker run` com flags, descrevemos o estado desejado em YAML — mais legível, versionável e reproduzível.
- **`ports: "8123:80"`** — Mapeia a porta 8123 do host para a 80 do container. Acessar `localhost:8123` no navegador chega ao Nginx dentro do container.
- **`volumes: ./html:/usr/share/nginx/html`** — Bind mount da pasta local. O Nginx serve os arquivos de `/usr/share/nginx/html` por padrão; ao montar nossa pasta ali, qualquer arquivo `.html` local é servido automaticamente — sem rebuild.
- **`docker compose up -d`** cria a rede padrão, o container e os volumes de uma vez. **`docker compose down`** destrói container e rede, mas mantém os arquivos locais intactos.
- O aviso sobre `version: '3'` sendo obsoleto indica que versões modernas do Docker Compose não requerem mais o campo `version` no arquivo.

---

## 🗺️ Conceitos consolidados neste exercício

| Conceito | Onde apareceu | Ponto-chave |
|----------|--------------|-------------|
| Bind mount (`-v`) | Ex. 1, 6 | Espelha arquivo/pasta do host no container em tempo real |
| `--rm` | Ex. 2 | Container se auto-remove ao encerrar — ideal para tarefas pontuais |
| `docker logs -f` | Ex. 1 | Stream de saída do container em tempo real |
| Redes customizadas | Ex. 3, 4, 5 | Habilitam DNS interno por nome de container |
| Isolamento de rede | Ex. 4 | Containers em redes diferentes não se comunicam |
| Variáveis de ambiente (`-e`) | Ex. 5 | Configuram serviços na inicialização sem alterar a imagem |
| Docker Compose | Ex. 6 | Orquestra múltiplos serviços de forma declarativa via YAML |
