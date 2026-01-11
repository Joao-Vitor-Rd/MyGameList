# Trabalho GC - REST API com Spring Boot

API REST para gerenciamento de jogos (Games) com suporte a duas versões: V1 e V2.

## Requisitos

- Docker e Docker Compose instalados
- Porta 8080 disponível (aplicação)
- Porta 3307 disponível (MySQL)

## Como Iniciar o Projeto

### 1. Usando Docker Compose (Recomendado)

Clone o repositório e acesse o diretório:

```bash
cd trabalho-gc
```

**Primeira vez (build + start):**

```powershell
docker-compose build --no-cache
docker-compose up
```

**Próximas vezes (sem rebuild):**

```powershell
docker-compose up
```

**Parar a aplicação:**

```powershell
docker-compose down
```

### 2. Outputs Esperados

Após executar `docker-compose up`, você verá:

- **MySQL**: `HikariPool-1 - Start completed`
- **Aplicação**: `Tomcat initialized with port 8080`
- **Migrations**: `Schema 'game_db' is up to date`

A aplicação estará pronta quando ver:

```
o.s.b.w.e.tomcat.TomcatWebServer : Tomcat started on port(s): 8080
```

## Documentação da API

Acesse a documentação interativa (Swagger UI):

```
http://localhost:8080/swagger-ui.html
```

Ou via OpenAPI:

```
http://localhost:8080/v3/api-docs
```

### Endpoints Disponíveis

#### **V1 - `/api/game/v1`** (Versão Original)

- `GET /api/game/v1` - Listar todos os jogos (com paginação)
- `GET /api/game/v1/{id}` - Obter jogo por ID
- `GET /api/game/v1/findGameByName/{name}` - Buscar jogos por nome
- `POST /api/game/v1` - Criar novo jogo
- `POST /api/game/v1/massCreation` - Criar múltiplos jogos (CSV/XLSX)
- `GET /api/game/v1/exportPage` - Exportar página de jogos (CSV/XLSX)
- `PUT /api/game/v1` - Atualizar jogo
- `PATCH /api/game/v1/{id}` - Marcar jogo como finalizado
- `DELETE /api/game/v1/{id}` - Deletar jogo

#### **V2 - `/api/game/v2`** (Nova Versão com Star Rating + HATEOAS)

- `GET /api/game/v2` - Listar todos os jogos com rating
- `GET /api/game/v2/{id}` - Obter jogo com rating por ID
- `GET /api/game/v2/findGameByName/{name}` - Buscar jogos com rating por nome
- `POST /api/game/v2` - Criar novo jogo com rating

### Parâmetros de Paginação

Aplicável a endpoints GET com listagem:

```
?page=0          # Página (padrão: 0)
&size=12         # Quantidade por página (padrão: 12)
&direction=asc   # Ordenação: asc ou desc (padrão: asc)
```

**Exemplo:**

```
GET /api/game/v2?page=1&size=20&direction=desc
```

## Diferenças V1 vs V2

| Recurso | V1 | V2 |
|---------|----|----|
| **Star Rating** | Não | Sim |
| **HATEOAS Links** | Sim | Sim |
| **Campos** | id, name, developer, year | id, name, developer, year, star_rating |
| **Formato** | Padrão | Camel Case JSON |

### Exemplo V2 Response

```json
{
  "id": 1,
  "name": "The Legend of Zelda",
  "developer": "Nintendo",
  "year": 1986,
  "star_rating": 5,
  "_links": {
    "self": { "href": "http://localhost:8080/api/game/v2/1" },
    "findAll": { "href": "http://localhost:8080/api/game/v2" },
    "findByName": { "href": "http://localhost:8080/api/game/v2/findGameByName/" },
    "create": { "href": "http://localhost:8080/api/game/v2" }
  }
}
```

## Banco de Dados

- **Banco**: MySQL 8.0
- **Database**: `game_db`
- **Usuário**: `gameuser`
- **Senha**: `gamepass`
- **Host**: `mysql:3306` (Docker)

As tabelas são criadas automaticamente via Flyway Migrations.

## Estrutura do Projeto

```
trabalho-gc/
├── src/
│   ├── main/java/br/com/jovirds/
│   │   ├── controllers/
│   │   │   ├── GameController.java (V1)
│   │   │   ├── GameControllerV2.java (V2)
│   │   │   └── docs/ (Documentação Swagger)
│   │   ├── service/
│   │   │   └── GameService.java
│   │   ├── model/
│   │   │   └── Game.java (Entidade JPA)
│   │   ├── data/dto/
│   │   │   ├── V1/ (GameDTO)
│   │   │   └── V2/ (GameDTOV2)
│   │   └── ...
│   └── resources/
│       ├── application.yml (Configuração)
│       └── db/migration/ (Scripts SQL Flyway)
├── Dockerfile
├── docker-compose.yml
└── pom.xml
```

## Configurações

### application.yml

```yaml
spring:
  datasource:
    url: jdbc:mysql://mysql:3306/game_db
    username: gameuser
    password: gamepass
  
  jpa:
    hibernate:
      ddl-auto: update
```

## Checklist de Verificação

- Docker Desktop está em execução
- Executou `docker-compose up`
- Aplicação iniciou na porta 8080
- Pode acessar `http://localhost:8080/swagger-ui.html`
- Banco de dados conectou (ver logs)

## Troubleshooting

### Porta 8080 já em uso

```powershell
netstat -ano | findstr :8080
taskkill /PID <PID> /F
```

### Porta 3307 já em uso

```powershell
netstat -ano | findstr :3307
taskkill /PID <PID> /F
```

Ou altere em `docker-compose.yml`:

```yaml
ports:
  - "3308:3306"  # Mude 3307 para 3308
```

### Contêineres não iniciam

```powershell
docker-compose down
docker system prune -a
docker-compose up --build
```

## Exemplo de Requisições

### Listar Jogos V2

```bash
curl -X GET "http://localhost:8080/api/game/v2?page=0&size=10" \
  -H "Accept: application/json"
```

### Criar Jogo V2

```bash
curl -X POST "http://localhost:8080/api/game/v2" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "The Last of Us",
    "developer": "Naughty Dog",
    "year": 2013,
    "star_rating": 5
  }'
```

### Buscar por Nome V2

```bash
curl -X GET "http://localhost:8080/api/game/v2/findGameByName/Zelda" \
  -H "Accept: application/json"
```

## Documentação Adicional

- **Swagger UI**: http://localhost:8080/swagger-ui.html
- **OpenAPI Spec**: http://localhost:8080/v3/api-docs
- **Spring Boot**: https://spring.io/projects/spring-boot
- **Flyway**: https://flywaydb.org/

## Autor

Trabalho académico - Gerenciamento de Conteúdo com Java e Spring Boot

---

**Última atualização**: Janeiro 2026
