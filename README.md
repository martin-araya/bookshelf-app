# �� Bookshelf

> Biblioteca personal self-hosted, open source y 100% autogestionada en tu propio PC.

[
	[
		[
			[
				[
					[

						***

						## ¿Qué es Bookshelf?

						**Bookshelf** es una aplicación web full-stack para gestionar tu biblioteca personal de libros, completamente auto-hospedada en tu propio PC. Sin suscripciones, sin telemetría, sin dependencias de servicios en la nube propietarios.

						Con Bookshelf puedes catalogar tus libros físicos y digitales, hacer seguimiento de tu lectura, enriquecer metadatos automáticamente con un scraper inteligente y conversar con un asistente IA que conoce tu biblioteca y recuerda tus conversaciones anteriores.

						Todo corre con un único comando:

						```bash
						docker compose up -d
						```

						***

						## ✨ Características principales

						### Gestión de biblioteca
						- Catálogo completo con título, autor, ISBN, portada, sinopsis, género, editorial, año, páginas, idioma
						- Estados de lectura: `sin leer`, `leyendo`, `leído`, `abandonado`, `releer`
						- Rating personal de 1 a 5 estrellas con reseñas privadas
						- Registro de fechas de inicio y fin de lectura
						- Notas y highlights por libro
						- Colecciones y listas personalizadas (ej: "Ciencia ficción 2025", "Pendientes")
						- Sistema de etiquetas (tags) libre
						- Registro de préstamos a amigos

						### Scraper automático de metadatos
						- Búsqueda por ISBN-10, ISBN-13 o título
						- Fuentes: Google Books API, OpenLibrary, GoodReads (scraping HTML)
						- Jobs asíncronos con estado en tiempo real
						- Descarga y almacenamiento automático de portadas en Cloudflare R2
						- Importación masiva (bulk) por lista de ISBNs

						### Asistente IA con memoria
						- Chat en tiempo real con streaming vía WebSocket
						- Modelos gratuitos de OpenRouter: Llama 3.3, Gemma 3, DeepSeek R1, Phi-4
						- Memoria persistente de conversaciones (ChromaDB)
						- Indexación semántica de sinopsis y notas personales
						- Recomendaciones basadas en historial de lectura
						- Resúmenes automáticos de libros desde sinopsis o archivos

						### Estadísticas de lectura
						- Libros leídos por mes/año (gráfico de barras)
						- Géneros y autores más leídos (gráfico de torta)
						- Velocidad promedio de lectura (páginas/día)
						- Rachas de lectura activa
						- Metas de libros anuales con progreso

						### Almacenamiento
						- Portadas y archivos (PDF, ePub) en Cloudflare R2
						- 10 GB gratuitos en R2 (sin costos de egress)
						- Emulación local con MinIO para desarrollo offline
						- URLs presignadas para uploads directos desde el browser

						### Seguridad y privacidad
						- Autenticación JWT con refresh tokens
						- Passwords con bcrypt (cost factor 12)
						- Todos los datos viven en tu PC
						- Backends internos nunca expuestos al host (solo accesibles en red Docker)
						- Rate limiting en el API Gateway

						***

						## ��️ Stack tecnológico

						| Capa | Tecnología | Versión | Puerto |
						|------|-----------|---------|--------|
						| **Frontend** | Next.js + React + Tailwind CSS v4 | Next.js 16 | `3121` |
						| **API Gateway** | Jakarta EE 11 + Open Liberty + MicroProfile 7 | 26.x | `9181` / `9543` |
						| **Core API** | Go + Gin + GORM | Go 1.23 | `8741` (interno) |
						| **AI Service** | Python + Robyn + LangChain + OpenRouter | Python 3.12 | `8742` (interno) |
						| **Scraper** | Python + Litestar + httpx + msgspec | Python 3.12 | `8743` (interno) |
						| **Base de datos** | PostgreSQL + pgvector + pg_trgm | PG 16 | `5642` (interno) |
						| **Cache / Pub-Sub** | Redis Stack (RedisJSON + RedisSearch) | Redis 7 | `6481` (interno) |
						| **Vector Store** | ChromaDB (embeddings + memoria IA) | Latest | `8191` (interno) |
						| **Object Storage** | Cloudflare R2 / MinIO (dev) | — | `9341` (MinIO dev) |

						***

						## ��️ Arquitectura

						```
						Browser
						   │
						      ▼
						      Next.js 16 (:3121)
						         │ HTTP / WebSocket
						            ▼
						            Open Liberty Gateway (:9181)
						               │
						                  ├──► /api/books, /api/auth, /api/upload   ──► Go + Gin (:8741)
						                     │                                                │
						                        ├──► /api/ai/*, /api/ai/ws (WebSocket)    ──► Python Robyn (:8742)
						                           │                                                │
						                              └──► /api/scrape/*                        ──► Python Litestar (:8743)
						                                                                                   │
						                                                                                                             ┌──────────────────────────┤
						                                                                                                                                       ▼          ▼           ▼   ▼
						                                                                                                                                                            PostgreSQL   Redis      ChromaDB  R2/MinIO
						                                                                                                                                                                                   (:5642)   (:6481)    (:8191)
						                                                                                                                                                                                   ```

						                                                                                                                                                                                   ***

						                                                                                                                                                                                   ## �� Requisitos previos

						                                                                                                                                                                                   - **Docker Desktop** 4.x o superior (con Docker Compose v2)
						                                                                                                                                                                                   - **Git**
						                                                                                                                                                                                   - **Cuenta en Cloudflare** (gratis) para R2 — en desarrollo se usa MinIO local
						                                                                                                                                                                                   - **API Key de OpenRouter** (gratis) para los modelos de IA
						                                                                                                                                                                                   - **API Key de Google Books** (gratis, 1000 req/día) — opcional, OpenLibrary es el fallback

						                                                                                                                                                                                   > **RAM recomendada:** mínimo 6 GB libres para todos los contenedores. ChromaDB y el modelo de embeddings son los más demandantes.

						                                                                                                                                                                                   ***

						                                                                                                                                                                                   ## �� Inicio rápido

						                                                                                                                                                                                   ### 1. Clonar el repositorio

						                                                                                                                                                                                   ```bash
						                                                                                                                                                                                   git clone https://github.com/tu-usuario/bookshelf.git
						                                                                                                                                                                                   cd bookshelf
						                                                                                                                                                                                   ```

						                                                                                                                                                                                   ### 2. Configurar variables de entorno

						                                                                                                                                                                                   ```bash
						                                                                                                                                                                                   cp .env.example .env
						                                                                                                                                                                                   ```

						                                                                                                                                                                                   Editar `.env` con tus credenciales:

						                                                                                                                                                                                   ```env
						                                                                                                                                                                                   # Base de datos
						                                                                                                                                                                                   POSTGRES_USER=bookshelf
						                                                                                                                                                                                   POSTGRES_PASSWORD=changeme_strong_password
						                                                                                                                                                                                   POSTGRES_DB=bookshelf

						                                                                                                                                                                                   # JWT
						                                                                                                                                                                                   JWT_SECRET=your_super_secret_key_min_32_chars
						                                                                                                                                                                                   JWT_REFRESH_SECRET=another_super_secret_key

						                                                                                                                                                                                   # Cloudflare R2 (o dejar vacío para usar MinIO en dev)
						                                                                                                                                                                                   R2_ACCOUNT_ID=
						                                                                                                                                                                                   R2_ACCESS_KEY_ID=
						                                                                                                                                                                                   R2_SECRET_ACCESS_KEY=
						                                                                                                                                                                                   R2_BUCKET_NAME=bookshelf-storage
						                                                                                                                                                                                   R2_PUBLIC_URL=

						                                                                                                                                                                                   # OpenRouter
						                                                                                                                                                                                   OPENROUTER_API_KEY=sk-or-v1-...
						                                                                                                                                                                                   OPENROUTER_DEFAULT_MODEL=meta-llama/llama-3.3-70b-instruct:free

						                                                                                                                                                                                   # Google Books (opcional)
						                                                                                                                                                                                   GOOGLE_BOOKS_API_KEY=

						                                                                                                                                                                                   # Minio (solo desarrollo)
						                                                                                                                                                                                   MINIO_ROOT_USER=minioadmin
						                                                                                                                                                                                   MINIO_ROOT_PASSWORD=minioadmin123
						                                                                                                                                                                                   ```

						                                                                                                                                                                                   ### 3. Levantar todos los servicios

						                                                                                                                                                                                   ```bash
						                                                                                                                                                                                   docker compose up -d
						                                                                                                                                                                                   ```

						                                                                                                                                                                                   ### 4. Verificar que todo esté corriendo

						                                                                                                                                                                                   ```bash
						                                                                                                                                                                                   docker compose ps
						                                                                                                                                                                                   ```

						                                                                                                                                                                                   ### 5. Abrir la aplicación

						                                                                                                                                                                                   Navegar a [http://localhost:3121](http://localhost:3121)

						                                                                                                                                                                                   El primer usuario que se registra obtiene el rol de administrador automáticamente.

						                                                                                                                                                                                   ***

						                                                                                                                                                                                   ## �� Estructura del proyecto

						                                                                                                                                                                                   ```
						                                                                                                                                                                                   bookshelf/
						                                                                                                                                                                                   ├── apps/
						                                                                                                                                                                                   │   └── web/                    # Next.js 16 frontend
						                                                                                                                                                                                   │       ├── app/                # App Router (layouts, pages)
						                                                                                                                                                                                   │       ├── components/         # Componentes reutilizables
						                                                                                                                                                                                   │       ├── lib/                # Utilidades, hooks, stores
						                                                                                                                                                                                   │       └── public/             # Assets estáticos
						                                                                                                                                                                                   │
						                                                                                                                                                                                   ├── services/
						                                                                                                                                                                                   │   ├── core-api/               # Go + Gin (API principal)
						                                                                                                                                                                                   │   │   ├── cmd/                # Entry point
						                                                                                                                                                                                   │   │   ├── internal/
						                                                                                                                                                                                   │   │   │   ├── handlers/       # HTTP handlers
						                                                                                                                                                                                   │   │   │   ├── models/         # Structs + GORM models
						                                                                                                                                                                                   │   │   │   ├── repositories/   # Capa de acceso a datos
						                                                                                                                                                                                   │   │   │   └── services/       # Lógica de negocio
						                                                                                                                                                                                   │   │   └── migrations/         # Migraciones SQL (goose)
						                                                                                                                                                                                   │   │
						                                                                                                                                                                                   │   ├── ai-service/             # Python + Robyn (LLM + ChromaDB)
						                                                                                                                                                                                   │   │   ├── app/
						                                                                                                                                                                                   │   │   │   ├── routers/        # Endpoints Robyn
						                                                                                                                                                                                   │   │   │   ├── services/       # LangChain, ChromaDB
						                                                                                                                                                                                   │   │   │   └── models/         # msgspec Structs
						                                                                                                                                                                                   │   │   └── requirements.txt
						                                                                                                                                                                                   │   │
						                                                                                                                                                                                   │   └── scraper/                # Python + Litestar (scraping)
						                                                                                                                                                                                   │       ├── app/
						                                                                                                                                                                                   │       │   ├── routers/        # Endpoints Litestar
						                                                                                                                                                                                   │       │   ├── scrapers/       # Google Books, OpenLibrary, GoodReads
						                                                                                                                                                                                   │       │   └── models/         # msgspec Structs
						                                                                                                                                                                                   │       └── requirements.txt
						                                                                                                                                                                                   │
						                                                                                                                                                                                   ├── gateway/                    # Jakarta EE 11 + Open Liberty
						                                                                                                                                                                                   │   ├── src/main/java/
						                                                                                                                                                                                   │   │   └── dev/bookshelf/
						                                                                                                                                                                                   │   │       ├── gateway/        # Proxy inverso JAX-RS
						                                                                                                                                                                                   │   │       ├── filters/        # JWT validation, rate limiting
						                                                                                                                                                                                   │   │       └── health/         # MicroProfile Health
						                                                                                                                                                                                   │   └── src/main/liberty/
						                                                                                                                                                                                   │       └── config/server.xml
						                                                                                                                                                                                   │
						                                                                                                                                                                                   ├── infra/
						                                                                                                                                                                                   │   ├── postgres/
						                                                                                                                                                                                   │   │   └── init.sql            # Schema inicial
						                                                                                                                                                                                   │   ├── redis/
						                                                                                                                                                                                   │   │   └── redis.conf
						                                                                                                                                                                                   │   └── chromadb/
						                                                                                                                                                                                   │
						                                                                                                                                                                                   ├── docker-compose.yml          # Producción local (todos los servicios)
						                                                                                                                                                                                   ├── docker-compose.dev.yml      # Override con hot-reload y tools de dev
						                                                                                                                                                                                   ├── .env.example
						                                                                                                                                                                                   ├── PHASES.md                   # Plan de desarrollo por fases
						                                                                                                                                                                                   ├── CONTRIBUTING.md
						                                                                                                                                                                                   ├── LICENSE                     # GNU AGPL v3.0
						                                                                                                                                                                                   └── README.md
						                                                                                                                                                                                   ```

						                                                                                                                                                                                   ***

						                                                                                                                                                                                   ## �� Puertos de todos los servicios

						                                                                                                                                                                                   | Servicio | Puerto host | Puerto contenedor | Acceso |
						                                                                                                                                                                                   |----------|-------------|-------------------|--------|
						                                                                                                                                                                                   | Frontend (Next.js) | `3121` | `3000` | Público |
						                                                                                                                                                                                   | Gateway HTTP | `9181` | `9080` | Público |
						                                                                                                                                                                                   | Gateway HTTPS | `9543` | `9443` | Público |
						                                                                                                                                                                                   | Core API (Go) | — | `8741` | Solo red interna |
						                                                                                                                                                                                   | AI Service (Robyn) | — | `8742` | Solo red interna |
						                                                                                                                                                                                   | Scraper (Litestar) | — | `8743` | Solo red interna |
						                                                                                                                                                                                   | PostgreSQL | — | `5642` | Solo red interna |
						                                                                                                                                                                                   | Redis | — | `6481` | Solo red interna |
						                                                                                                                                                                                   | ChromaDB | — | `8191` | Solo red interna |
						                                                                                                                                                                                   | MinIO API (dev) | `9341` | `9000` | Dev solamente |
						                                                                                                                                                                                   | MinIO Console (dev) | `9342` | `9001` | Dev solamente |
						                                                                                                                                                                                   | pgAdmin (dev) | `5151` | `80` | Dev solamente |
						                                                                                                                                                                                   | Redis Commander (dev) | `8283` | `8081` | Dev solamente |

						                                                                                                                                                                                   ***

						                                                                                                                                                                                   ## �� Desarrollo y testing

						                                                                                                                                                                                   ### Modo desarrollo (con hot-reload)

						                                                                                                                                                                                   ```bash
						                                                                                                                                                                                   # Levantar solo las dependencias base (DB, cache)
						                                                                                                                                                                                   docker compose up -d postgres redis chromadb minio

						                                                                                                                                                                                   # Frontend con hot-reload
						                                                                                                                                                                                   cd apps/web && npm run dev

						                                                                                                                                                                                   # Core API con Air (Go hot-reload)
						                                                                                                                                                                                   cd services/core-api && air

						                                                                                                                                                                                   # AI Service con Robyn watch mode
						                                                                                                                                                                                   cd services/ai-service && python main.py --dev

						                                                                                                                                                                                   # Scraper con Litestar watch mode
						                                                                                                                                                                                   cd services/scraper && python main.py --reload
						                                                                                                                                                                                   ```

						                                                                                                                                                                                   ### Tests

						                                                                                                                                                                                   ```bash
						                                                                                                                                                                                   # Go: tests unitarios e integración
						                                                                                                                                                                                   cd services/core-api && go test ./...

						                                                                                                                                                                                   # Python AI Service
						                                                                                                                                                                                   cd services/ai-service && pytest

						                                                                                                                                                                                   # Python Scraper
						                                                                                                                                                                                   cd services/scraper && pytest

						                                                                                                                                                                                   # Frontend: Vitest + Playwright E2E
						                                                                                                                                                                                   cd apps/web && npm test && npm run test:e2e
						                                                                                                                                                                                   ```

						                                                                                                                                                                                   ### Herramientas de desarrollo incluidas

						                                                                                                                                                                                   | Herramienta | URL | Descripción |
						                                                                                                                                                                                   |-------------|-----|-------------|
						                                                                                                                                                                                   | pgAdmin | http://localhost:5151 | Admin de PostgreSQL |
						                                                                                                                                                                                   | Redis Commander | http://localhost:8283 | Admin de Redis |
						                                                                                                                                                                                   | MinIO Console | http://localhost:9342 | Admin de Object Storage |
						                                                                                                                                                                                   | Gateway Health | http://localhost:9181/health | MicroProfile Health |
						                                                                                                                                                                                   | Scraper OpenAPI | http://localhost:9181/api/scrape/docs | Swagger auto-generado |

						                                                                                                                                                                                   ***

						                                                                                                                                                                                   ## �� Paquetes principales por servicio

						                                                                                                                                                                                   ### Frontend — Next.js 16
						                                                                                                                                                                                   | Paquete | Propósito |
						                                                                                                                                                                                   |---------|-----------|
						                                                                                                                                                                                   | `tailwindcss` v4 | Estilos utilitarios |
						                                                                                                                                                                                   | `zustand` | Estado global (catálogo, sesión) |
						                                                                                                                                                                                   | `@tanstack/react-query` | Server state, cache de API |
						                                                                                                                                                                                   | `react-hook-form` + `zod` | Formularios con validación |
						                                                                                                                                                                                   | `recharts` | Gráficas de estadísticas |
						                                                                                                                                                                                   | `@radix-ui/react-*` | Componentes accesibles sin estilos |
						                                                                                                                                                                                   | `lucide-react` | Iconos |
						                                                                                                                                                                                   | `pdfjs-dist` | Visor de PDFs inline |

						                                                                                                                                                                                   ### Core API — Go
						                                                                                                                                                                                   | Paquete | Propósito |
						                                                                                                                                                                                   |---------|-----------|
						                                                                                                                                                                                   | `gin-gonic/gin` | HTTP framework |
						                                                                                                                                                                                   | `gorm.io/gorm` | ORM + migraciones |
						                                                                                                                                                                                   | `golang-jwt/jwt/v5` | Autenticación JWT |
						                                                                                                                                                                                   | `aws/aws-sdk-go-v2/service/s3` | Cloudflare R2 / MinIO |
						                                                                                                                                                                                   | `redis/go-redis/v9` | Cliente Redis |
						                                                                                                                                                                                   | `pressly/goose/v3` | Migraciones SQL |
						                                                                                                                                                                                   | `viper` | Configuración desde ENV |
						                                                                                                                                                                                   | `uber-go/zap` | Logging estructurado |
						                                                                                                                                                                                   | `testify` | Testing |

						                                                                                                                                                                                   ### AI Service — Python + Robyn
						                                                                                                                                                                                   | Paquete | Propósito |
						                                                                                                                                                                                   |---------|-----------|
						                                                                                                                                                                                   | `robyn` | Framework ASGI (Rust runtime) |
						                                                                                                                                                                                   | `langchain` + `langchain-openai` | Cadenas LLM + OpenRouter |
						                                                                                                                                                                                   | `chromadb` | Vector store cliente |
						                                                                                                                                                                                   | `sentence-transformers` | Generación de embeddings local |
						                                                                                                                                                                                   | `redis` | Cache de respuestas LLM |
						                                                                                                                                                                                   | `httpx` | HTTP async |
						                                                                                                                                                                                   | `msgspec` | Serialización ultrarrápida |

						                                                                                                                                                                                   ### Scraper — Python + Litestar
						                                                                                                                                                                                   | Paquete | Propósito |
						                                                                                                                                                                                   |---------|-----------|
						                                                                                                                                                                                   | `litestar` | Framework ASGI (más rápido que FastAPI) |
						                                                                                                                                                                                   | `httpx` | HTTP async concurrente |
						                                                                                                                                                                                   | `beautifulsoup4` + `lxml` | Parsing HTML (GoodReads) |
						                                                                                                                                                                                   | `msgspec` | DTOs tipados y validación |
						                                                                                                                                                                                   | `redis` | Job queue y pub/sub |
						                                                                                                                                                                                   | `boto3` / `aiobotocore` | Upload de portadas a R2 |
						                                                                                                                                                                                   | `tenacity` | Retry automático en fallos de red |

						                                                                                                                                                                                   ### Gateway — Jakarta EE 11
						                                                                                                                                                                                   | Feature | Descripción |
						                                                                                                                                                                                   |---------|-------------|
						                                                                                                                                                                                   | `Jakarta REST 3.1` | Proxy inverso con JAX-RS Client |
						                                                                                                                                                                                   | `MicroProfile JWT 2.1` | Validación de tokens JWT |
						                                                                                                                                                                                   | `MicroProfile Health 4.0` | Endpoints `/health/live` y `/health/ready` |
						                                                                                                                                                                                   | `MicroProfile Metrics 5.1` | Métricas Prometheus |
						                                                                                                                                                                                   | `MicroProfile Fault Tolerance 4.0` | Circuit breaker hacia backends |
						                                                                                                                                                                                   | `MicroProfile OpenAPI 3.1` | Documentación automática |

						                                                                                                                                                                                   ***

						                                                                                                                                                                                   ## �� Modelos de IA disponibles (OpenRouter :free)

						                                                                                                                                                                                   | Modelo | Contexto | Ideal para |
						                                                                                                                                                                                   |--------|----------|------------|
						                                                                                                                                                                                   | `meta-llama/llama-3.3-70b-instruct:free` | 128k tokens | Chat, recomendaciones complejas |
						                                                                                                                                                                                   | `google/gemma-3-27b-it:free` | 96k tokens | Resúmenes, análisis |
						                                                                                                                                                                                   | `deepseek/deepseek-r1:free` | 64k tokens | Razonamiento, preguntas complejas |
						                                                                                                                                                                                   | `microsoft/phi-4:free` | 16k tokens | Respuestas rápidas y concisas |
						                                                                                                                                                                                   | `mistralai/mistral-7b-instruct:free` | 32k tokens | Fallback ligero |

						                                                                                                                                                                                   El modelo se configura por defecto en `.env` y el usuario puede cambiarlo desde la UI en Configuración.

						                                                                                                                                                                                   ***

						                                                                                                                                                                                   ## ��️ Esquema de base de datos (resumen)

						                                                                                                                                                                                   ```sql
						                                                                                                                                                                                   -- Usuarios y autenticación
						                                                                                                                                                                                   users (id, email, username, password_hash, role, created_at)
						                                                                                                                                                                                   refresh_tokens (id, user_id, token_hash, expires_at)

						                                                                                                                                                                                   -- Catálogo principal
						                                                                                                                                                                                   books (id, title, subtitle, isbn_10, isbn_13, cover_url, synopsis,
						                                                                                                                                                                                          year, pages, language, publisher, created_at, updated_at)
						                                                                                                                                                                                          authors (id, name, bio, photo_url)
						                                                                                                                                                                                          book_authors (book_id, author_id)
						                                                                                                                                                                                          genres (id, name, slug)
						                                                                                                                                                                                          book_genres (book_id, genre_id)
						                                                                                                                                                                                          tags (id, name, user_id)
						                                                                                                                                                                                          book_tags (book_id, tag_id)

						                                                                                                                                                                                          -- Biblioteca personal del usuario
						                                                                                                                                                                                          user_books (id, user_id, book_id, status, rating, review,
						                                                                                                                                                                                                      started_at, finished_at, notes, created_at)
						                                                                                                                                                                                                      reading_logs (id, user_book_id, pages_read, logged_at)

						                                                                                                                                                                                                      -- Colecciones
						                                                                                                                                                                                                      collections (id, user_id, name, description, is_public, created_at)
						                                                                                                                                                                                                      collection_books (collection_id, book_id, position)

						                                                                                                                                                                                                      -- Préstamos
						                                                                                                                                                                                                      loans (id, user_id, book_id, borrower_name, loaned_at, returned_at)

						                                                                                                                                                                                                      -- Scraping jobs
						                                                                                                                                                                                                      scrape_jobs (id, user_id, query, status, result_book_id, error, created_at)

						                                                                                                                                                                                                      -- IA
						                                                                                                                                                                                                      chat_sessions (id, user_id, title, created_at)
						                                                                                                                                                                                                      chat_messages (id, session_id, role, content, created_at)
						                                                                                                                                                                                                      ```

						                                                                                                                                                                                                      ***

						                                                                                                                                                                                                      ## �� Seguridad

						                                                                                                                                                                                                      - Contraseñas hasheadas con **bcrypt** (cost 12)
						                                                                                                                                                                                                      - **JWT** con expiración corta (15 min) + refresh tokens (7 días) en Redis
						                                                                                                                                                                                                      - Backends nunca expuestos al host — solo accesibles en `backend-net`
						                                                                                                                                                                                                      - CORS configurado en el gateway — solo acepta el origen del frontend
						                                                                                                                                                                                                      - Rate limiting por IP en el gateway (configurable en `server.xml`)
						                                                                                                                                                                                                      - Variables de entorno secretas nunca en el código ni en el repositorio
						                                                                                                                                                                                                      - Validación de tipos estricta en todos los endpoints (msgspec, Zod, Go types)

						                                                                                                                                                                                                      ***

						                                                                                                                                                                                                      ## �� Contribuir

						                                                                                                                                                                                                      1. Fork del repositorio
						                                                                                                                                                                                                      2. Crear rama desde `main`: `git checkout -b feature/mi-feature`
						                                                                                                                                                                                                      3. Leer `PHASES.md` para entender en qué fase encaja tu contribución
						                                                                                                                                                                                                      4. Commits en español o inglés con formato convencional: `feat:`, `fix:`, `docs:`
						                                                                                                                                                                                                      5. Asegurarse de que los tests pasen: `docker compose run --rm core-api go test ./...`
						                                                                                                                                                                                                      6. Abrir PR describiendo el cambio y la fase a la que pertenece

						                                                                                                                                                                                                      Al contribuir, aceptas que tu código queda bajo la misma licencia **GNU AGPL v3.0** del proyecto. Si planeas contribuciones significativas, firma el CLA (Contributor License Agreement) que encontrarás en `CONTRIBUTING.md`.

						                                                                                                                                                                                                      ***

						                                                                                                                                                                                                      ## �� Licencia

						                                                                                                                                                                                                      Bookshelf es software libre distribuido bajo los términos de la **GNU Affero General Public License v3.0 (AGPL-3.0)**.

						                                                                                                                                                                                                      Esto significa:
						                                                                                                                                                                                                      
						                                                                                                                                                                                                      - ✅ Puedes usar, estudiar, modificar y distribuir este software libremente
						                                                                                                                                                                                                      - ✅ El uso personal y self-hosted es completamente libre, sin restricciones
						                                                                                                                                                                                                      - ⚠️  Si ofreces Bookshelf como servicio web (SaaS), **debes publicar tu código fuente** bajo AGPL-3.0
						                                                                                                                                                                                                      - ⚠️  Las obras derivadas deben distribuirse bajo la misma licencia

						                                                                                                                                                                                                      Ver el archivo [LICENSE](./LICENSE) para el texto completo.

						                                                                                                                                                                                                      Para usos comerciales o bajo licencia diferente, contactar al equipo del proyecto.

						                                                                                                                                                                                                      ***

						                                                                                                                                                                                                      ## �� Reconocimientos

						                                                                                                                                                                                                      - [OpenRouter](https://openrouter.ai/) por los modelos de IA gratuitos
						                                                                                                                                                                                                      - [Open Liberty](https://openliberty.io/) por el runtime Jakarta EE de IBM
						                                                                                                                                                                                                      - [ChromaDB](https://www.trychroma.com/) por el vector store open source
						                                                                                                                                                                                                      - [Litestar](https://litestar.dev/) por el framework Python moderno y rápido
						                                                                                                                                                                                                      - [Robyn](https://robyn.tech/) por el framework Python con runtime Rust
						                                                                                                                                                                                                      - Todos los proyectos self-hosted que inspiraron este: Gitea, Nextcloud, Immich, Paperless-ngx

						                                                                                                                                                                                                      ***

						                                                                                                                                                                                                      *Bookshelf — Tu biblioteca personal, a tu manera.*
					]
				]
			]
		]
	]
]
