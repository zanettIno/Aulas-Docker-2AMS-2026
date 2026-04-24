# 🐳 Docker — Aula 1: Fundamentos

Docker é uma plataforma de **containerização** que permite empacotar aplicações e suas dependências em unidades isoladas chamadas **containers**. Isso garante que a aplicação rode da mesma forma em qualquer ambiente (desenvolvimento, staging, produção).

---

## 📌 Verificando a instalação

### Versão do cliente Docker
```bash
docker --version
```
> Exibe apenas a versão do CLI (cliente). Útil para checar se o Docker está instalado.

### Versão completa (cliente + servidor/daemon)
```bash
# sudo pode ser necessário dependendo do sistema
sudo docker version
```
> Exibe informações detalhadas tanto do **client** quanto do **server** (Docker Daemon). Se o server não aparecer, o daemon não está rodando.

---

## 🏷️ Tags em Docker

Tags são um sistema de **versionamento e organização de imagens**. Internamente, cada imagem possui um hash único (SHA256), mas as tags funcionam como **aliases legíveis** para esses hashes.

```
sha256:a1b2c3... ← identificador real
       ↑
   3.23.3  ←  3.23  ←  3  ←  latest
```

- Podemos usar letras e números nas tags
- `latest` é a tag padrão quando nenhuma é especificada
- A mesma imagem pode ter múltiplas tags apontando para ela
- Útil para manter padrões de versionamento semântico (ex: `1.0.0`, `1.0`, `1`, `stable`)

---

## 📋 Listando containers

### Ver containers em execução
```bash
docker ps
```

### Ver todos os containers (incluindo parados)
```bash
docker ps -a
```
> A flag `-a` (ou `--all`) mostra containers que já encerraram, com seu status, tempo de execução e código de saída. Útil para debugar containers que falharam ou já terminaram.

---

## 📥 Baixando imagens

```bash
docker pull <imagem>:<tag>

# Exemplos
docker pull alpine
docker pull nginx:1.29
docker pull ubuntu:22.04
```
> Puxa a imagem do **Docker Hub** (ou outro registry configurado) para sua máquina local. Se a tag for omitida, usa `latest`.

---

## ▶️ Executando containers

```bash
docker run <imagem>
```
O `docker run` combina `docker pull` + `docker create` + `docker start` em um único comando.

### Exemplo 1 — Container efêmero (executa e morre)
```bash
docker run alpine echo "ola mundo"
```
> Sobe um container Alpine Linux, executa o `echo` e o container encerra. Pode ser visto depois com `docker ps -a`.

### Exemplo 2 — Modo interativo (acesso ao shell)
```bash
docker run -it alpine sh
```
| Flag | Significado |
|------|-------------|
| `-i` | Mantém o STDIN aberto (interativo) |
| `-t` | Aloca um pseudo-TTY (terminal) |
| `sh` | Comando executado no container |

> Você entra direto no shell do container Alpine. Digite `exit` para sair (e o container encerrará).

---

## 🔨 Buildando imagens customizadas

```bash
docker build .
```
> Lê o `Dockerfile` do diretório atual e constrói uma nova imagem. O `.` indica o **build context** (arquivos disponíveis durante o build).

### Com tag já definida no build
```bash
docker build -t minha-app:1.0 .
```

---

## 🏷️ Gerenciando tags e nomes

### Adicionar tag a uma imagem existente
```bash
# Por hash
docker tag <hash> <nova-tag>

# Com tag composta
docker tag <hash> <nome>:<versao>

# Exemplo
docker tag a1b2c3d4 minha-app:latest
docker tag a1b2c3d4 minha-app:1.0.0
```

### Nomear o container ao executar
```bash
docker run --name <nome> <imagem>

# Exemplo
docker run --name meu-nginx nginx
```
> Sem `--name`, o Docker gera nomes aleatórios (ex: `happy_curie`). Nomear facilita o gerenciamento posterior.

---

## 🔧 Flags e partículas importantes

| Flag / Comando | Descrição |
|----------------|-----------|
| `-d` | **Detach** — roda o container em background, independente do terminal |
| `-it` | **Interativo + TTY** — acesso ao terminal do container |
| `--name` | Define um nome legível para o container |
| `-p <host>:<container>` | **Port mapping** — mapeia porta do host para porta do container |
| `-v <host>:<container>` | **Volume** — monta diretório/arquivo do host no container |
| `:ro` | Modificador de volume: **read-only** (somente leitura) |
| `:rw` | Modificador de volume: **read-write** (leitura e escrita, padrão) |
| `rm` | Remove containers parados |
| `rmi` | Remove imagens |
| `stop` | Para um container em execução (envia SIGTERM, depois SIGKILL) |

---

## 🌐 Expondo portas — Exemplo com Nginx

```bash
docker run --name web1 -d -p 8080:80 nginx:1.29
```

| Parte | Significado |
|-------|-------------|
| `--name web1` | Nome do container |
| `-d` | Roda em background |
| `-p 8080:80` | Porta 8080 do host → porta 80 do container |
| `nginx:1.29` | Imagem e tag |

> Acesse `http://localhost:8080` no navegador para ver o Nginx rodando.

---

## 📁 Volumes — Exemplo servindo HTML estático

```bash
docker run --name web2 \
  -v /caminho/local/index.html:/usr/share/nginx/html/index.html:ro \
  -d \
  -p 8081:80 \
  nginx:1.29
```

| Parte | Significado |
|-------|-------------|
| `-v <caminho-host>:<caminho-container>` | Monta o arquivo do host dentro do container |
| `:ro` | O container só pode **ler** o arquivo, não modificar |
| `-p 8081:80` | Porta diferente para não conflitar com `web1` |

> Qualquer alteração no arquivo local reflete imediatamente no container (sem rebuild).
