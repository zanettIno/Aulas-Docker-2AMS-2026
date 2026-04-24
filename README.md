# 🐳 Docker — Notas de Aula & Exercícios

Repositório de estudos da disciplina de **Plataformas Multiplataforma** do curso de **Análise e Desenvolvimento de Sistemas** na FATEC São Caetano do Sul (AMS).

Contém anotações das aulas, complementadas com explicações aprofundadas, e relatórios de exercícios práticos.

---

## 👤 Autor

**Guilherme Xavier Zanetti**
2º ADS — FATEC São Caetano do Sul (AMS)

---

## 📁 Estrutura do repositório

```
📦 docker-aulas/
├── 📄 primeira-aula.md        # Fundamentos: comandos, imagens, containers, volumes e portas
├── 📄 segunda-aula.md         # Volumes nomeados, redes e Docker Compose
├── 📄 resumo-exercicio-01.md  # Resolução comentada do Exercício 01 (6 atividades práticas)
└── 📄 README.md               # Este arquivo
```

---

## 📚 Conteúdo das aulas

### Aula 1 — [`primeira-aula.md`](./primeira-aula.md)
Introdução ao Docker e seus conceitos fundamentais:

- Verificando versão do cliente e servidor (`docker version`)
- Sistema de **tags** e versionamento de imagens
- Listando containers (`docker ps`, `docker ps -a`)
- Baixando imagens do Docker Hub (`docker pull`)
- Executando containers (`docker run`) — modo efêmero e modo interativo (`-it`)
- Buildando imagens customizadas (`docker build`)
- Gerenciando tags e nomes de containers
- Flags essenciais: `-d`, `-p`, `-v`, `--name`, `--rm`
- Expondo portas e servindo arquivos estáticos com Nginx

### Aula 2 — [`segunda-aula.md`](./segunda-aula.md)
Persistência, comunicação e orquestração:

- **Volumes** nomeados: criação, listagem, inspeção e uso
- **Redes** customizadas: criação, inspeção e DNS interno entre containers
- **Docker Compose**: definição declarativa de serviços via `docker-compose.yml`
- Subindo e derrubando ambientes com `docker compose up/down`

---

## 🛠️ Exercícios Práticos

### Exercício 01 — [`resumo-exercicio-01.md`](./resumo-exercicio-01.md)

Resolução comentada com explicação do **porquê** de cada comando e decisão técnica:

| # | Atividade | Conceitos envolvidos |
|---|-----------|---------------------|
| 1 | Script Bash contador em container Ubuntu | Bind mount, `docker logs -f`, detached mode |
| 2 | Container efêmero printando nome | Flag `--rm`, containers one-shot |
| 3 | Comunicação entre containers via rede | Redes customizadas, DNS interno, `curl`, `ping` |
| 4 | Isolamento entre redes distintas | Namespaces de rede, falha intencional de resolução DNS |
| 5 | MariaDB + cliente SQL em rede dedicada | Variáveis de ambiente, cliente/servidor em containers |
| 6 | Nginx via Docker Compose com HTML local | Compose declarativo, bind mount, mapeamento de portas |

---

## 🧠 Conceitos-chave abordados

- **Container vs Imagem** — imagem é o template; container é a instância em execução
- **Bind Mount vs Volume nomeado** — bind para dev/hot-reload; volume para persistência gerenciada
- **Redes customizadas** — DNS interno por nome de container; isolamento entre redes
- **Docker Compose** — infraestrutura como código, orquestração de múltiplos serviços
- **Flags essenciais** — `-d`, `-it`, `--rm`, `-p`, `-v`, `-e`, `--name`, `--network`

---

## 🚀 Comandos de referência rápida

```bash
# Containers
docker run -d --name <nome> -p <host>:<container> <imagem>
docker ps -a
docker logs -f <nome>
docker rm -f <nome>

# Imagens
docker pull <imagem>:<tag>
docker build -t <nome>:<tag> .
docker rmi <imagem>

# Redes
docker network create <rede>
docker network inspect <rede>

# Volumes
docker volume create <volume>
docker volume inspect <volume>

# Compose
docker compose up -d
docker compose down
docker compose logs -f
```

---

## ⚙️ Pré-requisitos

- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (Windows/macOS) ou Docker Engine (Linux)
- Terminal / CMD / PowerShell

---

![Docker em ação](URL_DO_GIF_AQUI)

> 💡 *Substitua `URL_DO_GIF_AQUI` pela URL de um GIF do Docker — sugestão: buscar em [GIPHY](https://giphy.com/search/docker) ou [tenor.com](https://tenor.com/search/docker-gifs)*
