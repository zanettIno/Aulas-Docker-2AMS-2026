# 🐳 Docker — Aula 2: Volumes, Redes e Docker Compose

---

## 💾 Volumes

Volumes são o mecanismo oficial do Docker para **persistência de dados**. Diferente de mounts de arquivos (`-v /caminho/local`), volumes são gerenciados pelo próprio Docker Daemon e ficam armazenados em uma área controlada do servidor (`/var/lib/docker/volumes/` no Linux).

> **Por que usar volumes?**  
> Containers são efêmeros — quando removidos, tudo dentro deles é perdido. Volumes garantem que dados importantes (bancos de dados, uploads, configurações) sobrevivam ao ciclo de vida do container.

---

### 📋 Comandos de Volume

#### Help / subcomandos disponíveis
```bash
docker volume
```
> Lista todos os subcomandos disponíveis para gerenciamento de volumes.

#### Listar volumes
```bash
docker volume ls
```
> Exibe todos os volumes presentes no servidor Docker, com nome e driver.

#### Criar um volume
```bash
docker volume create "nome-do-volume"

# Exemplo
docker volume create dados_banco
```
> Cria um volume nomeado gerenciado pelo Docker. Ideal para bancos de dados (PostgreSQL, MySQL, MongoDB) onde os dados precisam persistir entre reinicializações.

#### Inspecionar um volume
```bash
docker volume inspect <nomeDoVolume>

# Exemplo
docker volume inspect dados_banco
```
> Retorna um JSON com detalhes como:
> - `Mountpoint`: caminho físico no host onde os dados estão armazenados
> - `Driver`: driver usado (geralmente `local`)
> - `CreatedAt`: data de criação

#### Usando o volume em um container
```bash
docker run -d \
  --name meu-banco \
  -v dados_banco:/var/lib/postgresql/data \
  postgres:16
```
> O volume `dados_banco` é montado no path do PostgreSQL. Mesmo que o container seja removido e recriado, os dados permanecem.

#### Remover volumes não utilizados
```bash
docker volume prune
```
> Remove todos os volumes que não estão sendo usados por nenhum container. **Use com cuidado** — dados são perdidos.

---

## 🌐 Redes (Networks)

Redes Docker permitem que containers se **comuniquem entre si de forma isolada**. Por padrão, containers na mesma rede conseguem se encontrar pelo **nome do container** (DNS interno), sem precisar expor portas ao host.

> **Por que criar redes customizadas?**  
> A rede padrão (`bridge`) não oferece resolução DNS por nome. Redes customizadas permitem que `app` acesse `banco` simplesmente pelo hostname `banco`.

---

### 📋 Comandos de Rede

#### Help / subcomandos disponíveis
```bash
docker network
```

#### Criar uma rede
```bash
docker network create rede_fatec

# Com driver explícito (bridge é o padrão para redes locais)
docker network create --driver bridge rede_fatec
```
> Tipos de driver:
> | Driver | Uso |
> |--------|-----|
> | `bridge` | Comunicação entre containers no mesmo host (padrão) |
> | `host` | Container compartilha a rede do host diretamente |
> | `none` | Container sem acesso à rede |
> | `overlay` | Comunicação entre hosts diferentes (Docker Swarm) |

#### Conectar um container à rede
```bash
docker run -it --network rede_fatec alpine sh

# Exemplo com dois containers na mesma rede se comunicando
docker run -d --name app --network rede_fatec minha-app
docker run -d --name banco --network rede_fatec postgres:16
# "app" pode acessar "banco" pelo hostname "banco"
```

#### Inspecionar uma rede
```bash
docker network inspect <nomeDaRede>

# Exemplo
docker network inspect rede_fatec
```
> Retorna JSON com containers conectados, faixa de IPs, gateway e configurações do driver. Útil para debugar problemas de conectividade entre containers.

#### Listar redes
```bash
docker network ls
```

#### Remover uma rede
```bash
docker network rm rede_fatec
```
> Só é possível remover redes sem containers conectados.

---

## 🐙 Docker Compose

Docker Compose é uma ferramenta para **definir e orquestrar múltiplos containers** através de um único arquivo YAML (`docker-compose.yml` ou `compose.yml`). Em vez de executar vários comandos `docker run` manualmente, você descreve toda a infraestrutura da aplicação de forma declarativa.

> **Quando usar Compose?**  
> Sempre que sua aplicação tiver mais de um serviço (ex: backend + banco de dados + cache). Ele garante que todos os containers subam na ordem certa, na mesma rede, com as configurações corretas.

### Exemplo de `docker-compose.yml`
```yaml
services:
  app:
    image: node:20-alpine
    ports:
      - "3000:3000"
    volumes:
      - .:/app
    depends_on:
      - banco
    networks:
      - rede_app

  banco:
    image: postgres:16
    environment:
      POSTGRES_PASSWORD: senha123
      POSTGRES_DB: meu_banco
    volumes:
      - dados_postgres:/var/lib/postgresql/data
    networks:
      - rede_app

volumes:
  dados_postgres:

networks:
  rede_app:
```

---

### 📋 Comandos do Docker Compose

#### Subir todos os serviços
```bash
docker compose up

# Em background (detached)
docker compose up -d

# Forçar rebuild das imagens antes de subir
docker compose up --build
```
> Lê o `docker-compose.yml` do diretório atual, cria as redes e volumes definidos, e sobe todos os containers. Com `-d`, libera o terminal.

#### Derrubar e limpar (containers + rede)
```bash
docker compose down
```
> Para e remove os containers e as redes criadas pelo Compose. **Por padrão, volumes NÃO são removidos** (para não perder dados).

```bash
# Remover também os volumes (CUIDADO: apaga dados persistidos)
docker compose down -v
```

#### Outros comandos úteis do Compose
```bash
# Ver logs de todos os serviços
docker compose logs -f

# Ver status dos serviços
docker compose ps

# Executar comando em um serviço rodando
docker compose exec app sh

# Parar sem remover
docker compose stop

# Rebuild de um serviço específico
docker compose build app
```

---

## 🗺️ Resumo: Volume vs Bind Mount vs Tmpfs

| Tipo | Onde os dados ficam | Gerenciado pelo Docker | Uso típico |
|------|--------------------|-----------------------|------------|
| **Volume** | `/var/lib/docker/volumes/` | ✅ Sim | Bancos de dados, dados persistentes |
| **Bind Mount** | Qualquer path do host (`-v /meu/path:/container`) | ❌ Não | Desenvolvimento, hot-reload de código |
| **Tmpfs** | Memória RAM (não persiste) | ✅ Sim | Dados sensíveis temporários |
