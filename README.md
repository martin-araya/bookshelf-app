# 📚 Bookshelf

> **Tu biblioteca personal, a tu manera.** Self-hosted, open source y 100% autogestionada en tu propio PC. Sin suscripciones, sin telemetría, sin nubes propietarias.

<div align="center">

[![License: AGPL v3](https://img.shields.io/badge/License-AGPL_v3-blueviolet.svg?style=flat-square)](https://www.gnu.org/licenses/agpl-3.0)
[![Docker](https://img.shields.io/badge/Docker-Compose_v2-2496ED?style=flat-square&logo=docker&logoColor=white)](https://docs.docker.com/compose/)
[![Next.js](https://img.shields.io/badge/Next.js-16-black?style=flat-square&logo=next.js)](https://nextjs.org/)
[![Go](https://img.shields.io/badge/Go-1.23-00ADD8?style=flat-square&logo=go)](https://go.dev/)
[![Python](https://img.shields.io/badge/Python-3.12-3572A5?style=flat-square&logo=python&logoColor=white)](https://python.org/)
[![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-336791?style=flat-square&logo=postgresql&logoColor=white)](https://postgresql.org/)
[![Status](https://img.shields.io/badge/Status-En_desarrollo-orange?style=flat-square)]()

</div>

---

## Tabla de contenidos

- [¿Qué es Bookshelf?](#-qué-es-bookshelf)
- [Características principales](#-características-principales)
- [Stack tecnológico](#️-stack-tecnológico)
- [Arquitectura](#️-arquitectura)
- [Redes Docker](#-redes-docker)
- [Requisitos previos](#-requisitos-previos)
- [Inicio rápido](#-inicio-rápido)
- [Estructura del proyecto](#-estructura-del-proyecto)
- [Puertos de todos los servicios](#-puertos-de-todos-los-servicios)
- [Desarrollo y testing](#-desarrollo-y-testing)
- [Paquetes principales por servicio](#-paquetes-principales-por-servicio)
- [Modelos de IA disponibles](#-modelos-de-ia-disponibles-openrouter-free)
- [Esquema de base de datos](#️-esquema-de-base-de-datos-resumen)
- [Seguridad](#-seguridad)
- [Variables de entorno](#-variables-de-entorno)
- [Roadmap](#-roadmap)
- [Preguntas frecuentes](#-preguntas-frecuentes)
- [Contribuir](#-contribuir)
- [Licencia](#-licencia)
- [Reconocimientos](#-reconocimientos)

---

## 📖 ¿Qué es Bookshelf?

**Bookshelf** es una aplicación web full-stack para gestionar tu biblioteca personal de libros físicos y digitales, completamente auto-hospedada en tu propio PC. Es la alternativa self-hosted a Goodreads, StoryGraph y Libib: sin algoritmos, sin anuncios, sin que nadie vea lo que lees.

Con Bookshelf puedes catalogar tu colección completa, hacer seguimiento detallado de tu lectura, enriquecer metadatos automáticamente con un scraper inteligente por ISBN, y conversar con un asistente IA que conoce tu biblioteca, recuerda tus conversaciones anteriores y te recomienda lecturas basadas en tu historial real.

Todo el sistema arranca con un único comando:

```bash
docker compose up -d
```

Y se apaga igual de simple:

```bash
docker compose down
```

Tus datos siempre en tu máquina. Tus libros, tu privacidad.

---

## ✨ Características principales

### 📚 Gestión de biblioteca
- Catálogo completo: título, autor, ISBN-10/13, portada, sinopsis, género, editorial, año, páginas, idioma
- Estados de lectura: `sin leer` · `leyendo` · `leído` · `abandonado` · `releer`
- Rating personal de 1 a 5 estrellas con reseñas privadas
- Registro de fechas de inicio y fin de lectura
- Notas y highlights por libro con soporte de Markdown
- Colecciones y listas personalizadas (ej: "Ciencia ficción 2025", "Pendientes del verano")
- Sistema de etiquetas (tags) completamente libre
- Registro de préstamos a amigos con fecha de devolución

### 🕷️ Scraper automático de metadatos
- Búsqueda por ISBN-10, ISBN-13 o título en lenguaje natural
- Fuentes: Google Books API · OpenLibrary · GoodReads (scraping HTML)
- Jobs asíncronos con progreso en tiempo real vía Server-Sent Events
- Descarga y almacenamiento automático de portadas en Cloudflare R2
- Importación masiva (bulk) por lista de ISBNs — hasta 50 a la vez
- Sistema de retry automático con backoff exponencial si la fuente falla

### 🤖 Asistente IA con memoria
- Chat en tiempo real con streaming de respuestas vía WebSocket
- Modelos 100% gratuitos de OpenRouter: Llama 3.3 · Gemma 3 · DeepSeek R1 · Phi-4
- Memoria persistente de conversaciones almacenada en ChromaDB
- Indexación semántica de sinopsis y notas personales para búsqueda por significado
- Recomendaciones personalizadas basadas en historial real de lectura
- Resúmenes automáticos de libros generados desde sinopsis o archivos subidos

### 📊 Estadísticas de lectura
- Libros terminados por mes y año (gráfico de barras)
- Géneros y autores más leídos (gráfico de torta)
- Velocidad promedio de lectura (páginas/día)
- Rachas de lectura activa (heatmap tipo GitHub)
- Metas de libros anuales con barra de progreso

### ☁️ Almacenamiento de archivos
- Portadas y archivos PDF/ePub en Cloudflare R2 (10 GB gratuitos, sin costos de egress)
- Emulación local con MinIO para desarrollo completamente offline
- URLs presignadas para uploads directos desde el browser sin pasar por el backend

### 🔐 Seguridad y privacidad
- Autenticación JWT (access 15 min) + refresh tokens rotantes (7 días) en Redis
- Passwords hasheados con bcrypt (cost factor 12)
- Todos tus datos viven exclusivamente en tu PC
- Servicios de backend nunca expuestos al host — solo accesibles dentro de la red Docker
- Rate limiting por IP en el API Gateway (configurable)
- CORS estricto: solo acepta peticiones del origen del frontend

---

## 🏗️ Stack tecnológico

| Capa | Tecnología | Versión | Puerto host |
|------|-----------|---------|-------------|
| **Frontend** | Next.js · React · Tailwind CSS v4 | Next.js 16 | `3121` |
| **API Gateway** | Jakarta EE 11 · Open Liberty · MicroProfile 7 | 26.x | `9181` / `9543` |
| **Core API** | Go · Gin · GORM | Go 1.23 | interno |
| **AI Service** | Python · Robyn · LangChain · OpenRouter | Python 3.12 | interno |
| **Scraper** | Python · Litestar · httpx · msgspec | Python 3.12 | interno |
| **Base de datos** | PostgreSQL · pgvector · pg_trgm | PG 16 | interno |
| **Cache / Pub-Sub** | Redis Stack · RedisJSON · RedisSearch | Redis 7 | interno |
| **Vector Store** | ChromaDB | Latest | interno |
| **Object Storage** | Cloudflare R2 / MinIO (dev) | — | `9341` (dev) |

> Los servicios marcados como **interno** son accesibles únicamente dentro de la red Docker privada `backend-net`. Ningún puerto de backend queda expuesto en el host.

---

## 🗺️ Arquitectura

```
┌─────────────────────────────────────────────────────┐
│                     BROWSER / PWA                    │
└──────────────────────────┬──────────────────────────┘
						   │ HTTP + WebSocket
						   ▼
┌─────────────────────────────────────────────────────┐
│              Next.js 16  (:3121)                     │
│        App Router · React 19 · Turbopack             │
└──────────────────────────┬──────────────────────────┘
						   │ HTTP + WebSocket
						   ▼
┌─────────────────────────────────────────────────────┐
│       Open Liberty API Gateway  (:9181/:9543)        │
│   JWT Validation · Rate Limit · Circuit Breaker      │
└──────┬────────────────────┬────────────────┬────────┘
	   │                    │                │
	   ▼                    ▼                ▼
┌─────────────┐  ┌─────────────────┐  ┌────────────────┐
│ Go + Gin    │  │ Python + Robyn  │  │Python + Litestar│
│ Core API    │  │  AI Service     │  │    Scraper      │
│  (:8741)    │  │   (:8742)       │  │   (:8743)       │
└──────┬──────┘  └───────┬─────────┘  └───────┬────────┘
	   │                 │                     │
	   └────────┬────────┴─────────┬───────────┘
				│                  │
	┌───────────▼───┐    ┌─────────▼───────┐
	│  PostgreSQL   │    │  Redis Stack    │
	│  + pgvector   │    │  + RedisJSON    │
	│   (:5642)     │    │   (:6481)       │
	└───────────────┘    └─────────────────┘
				 │
	┌────────────▼──────────┐    ┌──────────────┐
	│      ChromaDB         │    │  R2 / MinIO  │
	│  (vector embeddings)  │    │  (archivos)  │
	│      (:8191)          │    │  (:9341 dev) │
	└───────────────────────┘    └──────────────┘
```

---

## 🌐 Redes Docker

Bookshelf define tres redes aisladas para minimizar la superficie de ataque:

| Red | Propósito | Servicios |
|-----|-----------|-----------|
| `frontend-net` | Comunicación frontend ↔ gateway | Next.js, Open Liberty |
| `backend-net` | Comunicación interna de servicios | Open Liberty, Go API, Robyn, Litestar, PostgreSQL, Redis, ChromaDB |
| `storage-net` | Acceso a almacenamiento de objetos | Go API, Litestar, MinIO |

El frontend **nunca** tiene acceso directo a los servicios de backend ni a la base de datos. Toda petición pasa por el gateway.

---

## 📋 Requisitos previos

| Requisito | Versión mínima | Notas |
|-----------|---------------|-------|
| **Docker Desktop** | 4.x | Incluye Docker Compose v2 |
| **Git** | 2.x | — |
| **RAM libre** | 6 GB | ChromaDB y embeddings son los más demandantes |
| **Disco libre** | 5 GB | Para imágenes Docker e inicialización |
| **Cuenta Cloudflare** | Gratis | Solo para producción — en dev se usa MinIO local |
| **API Key OpenRouter** | Gratis | Para los modelos de IA; límite diario generoso |
| **API Key Google Books** | Gratis (1000/día) | Opcional — OpenLibrary es el fallback gratuito |

> **💡 Tip:** Para desarrollo offline completo, no necesitas ninguna cuenta externa. MinIO reemplaza R2 y OpenRouter puede apuntar a un modelo local con Ollama si lo deseas.

---

## 🚀 Inicio rápido

### 1. Clonar el repositorio

```bash
git clone https://github.com/tu-usuario/bookshelf.git
cd bookshelf
```

### 2. Configurar variables de entorno

```bash
cp .env.example .env
```

Editar `.env` con tus credenciales (ver sección [Variables de entorno](#-variables-de-entorno) para referencia completa).

### 3. Levantar todos los servicios

```bash
# Producción local (todos los servicios en Docker)
docker compose up -d

# Solo desarrollo (dependencias en Docker, apps en host con hot-reload)
docker compose -f docker-compose.dev.yml up -d
```

### 4. Verificar que todo esté corriendo

```bash
docker compose ps
# Todos los servicios deben mostrar estado "healthy"
```

### 5. Abrir la aplicación

Navegar a **[http://localhost:3121](http://localhost:3121)**

> El primer usuario en registrarse obtiene el rol `admin` automáticamente.

### Apagar el sistema

```bash
docker compose down          # Apaga sin borrar datos
docker compose down -v       # ⚠️  Apaga Y borra todos los volúmenes (reset total)
```

---

## 📁 Estructura del proyecto

```
bookshelf/
├── apps/
│   └── web/                        # Next.js 16 frontend
│       ├── app/                    # App Router: layouts, pages, route handlers
│       │   ├── (public)/           # Rutas sin autenticación (landing, login)
│       │   └── (app)/              # Rutas protegidas (library, chat, stats)
│       ├── components/             # Componentes React reutilizables
│       │   ├── ui/                 # Primitivos de diseño (Button, Card, Input…)
│       │   └── features/           # Componentes específicos (BookCard, ChatMessage…)
│       ├── lib/                    # Utilidades, clientes API, validaciones Zod
│       ├── hooks/                  # Custom hooks (useWebSocket, useBooks…)
│       ├── stores/                 # Zustand stores (auth, catalog, chat)
│       └── public/                 # Assets estáticos e íconos PWA
│
├── services/
│   ├── core-api/                   # Go + Gin: API principal
│   │   ├── cmd/server/main.go      # Entry point
│   │   ├── internal/
│   │   │   ├── config/             # Viper: configuración desde ENV
│   │   │   ├── database/           # GORM + pgvector setup
│   │   │   ├── cache/              # Cliente Redis
│   │   │   ├── middleware/          # JWT, CORS, rate limit, logger (Zap)
│   │   │   ├── handlers/           # HTTP handlers por dominio
│   │   │   ├── models/             # Structs GORM
│   │   │   ├── repositories/       # Capa de acceso a datos
│   │   │   └── services/           # Lógica de negocio
│   │   ├── migrations/             # Archivos SQL versionados (goose)
│   │   └── Dockerfile
│   │
│   ├── ai-service/                 # Python + Robyn: LLM + ChromaDB
│   │   ├── app/
│   │   │   ├── routers/            # Endpoints Robyn (REST + WebSocket)
│   │   │   ├── services/           # LangChain, ChromaDB, embeddings
│   │   │   └── models/             # msgspec Structs tipados
│   │   ├── requirements.txt
│   │   └── Dockerfile
│   │
│   └── scraper/                    # Python + Litestar: scraping de metadatos
│       ├── app/
│       │   ├── routers/            # Endpoints Litestar (con OpenAPI auto)
│       │   ├── scrapers/           # Clientes: Google Books, OpenLibrary, GoodReads
│       │   ├── workers/            # Job queue con asyncio + Redis
│       │   └── models/             # msgspec Structs tipados
│       ├── requirements.txt
│       └── Dockerfile
│
├── gateway/                        # Jakarta EE 11 + Open Liberty
│   ├── src/main/java/dev/bookshelf/
│   │   ├── gateway/                # Proxy inverso JAX-RS
│   │   ├── filters/                # JWT validation, rate limiting, CORS
│   │   └── health/                 # MicroProfile Health checks
│   ├── src/main/liberty/config/
│   │   └── server.xml              # Configuración Open Liberty
│   ├── pom.xml
│   └── Dockerfile
│
├── infra/
│   ├── postgres/init.sql           # Extensiones: pgvector, pg_trgm, pgcrypto
│   ├── redis/redis.conf            # maxmemory, persistencia RDB
│   └── minio/                      # Configuración inicial de buckets
│
├── docker-compose.yml              # Stack completo de producción local
├── docker-compose.dev.yml          # Overrides: hot-reload, pgAdmin, Redis Commander
├── .env.example                    # Plantilla de variables de entorno documentada
├── PHASES.md                       # Plan de desarrollo por fases
├── CONTRIBUTING.md                 # Guía para contribuidores + CLA
├── CHANGELOG.md                    # Historial de cambios por versión
├── LICENSE                         # GNU AGPL v3.0
└── README.md
```

---

## 🔌 Puertos de todos los servicios

| Servicio | Puerto host | Puerto contenedor | Acceso |
|----------|:-----------:|:-----------------:|--------|
| Frontend (Next.js) | **`3121`** | `3000` | 🌐 Público |
| Gateway HTTP | **`9181`** | `9080` | 🌐 Público |
| Gateway HTTPS | **`9543`** | `9443` | 🌐 Público |
| Core API (Go) | — | `8741` | 🔒 Red interna |
| AI Service (Robyn) | — | `8742` | 🔒 Red interna |
| Scraper (Litestar) | — | `8743` | 🔒 Red interna |
| PostgreSQL | — | `5642` | 🔒 Red interna |
| Redis | — | `6481` | 🔒 Red interna |
| ChromaDB | — | `8191` | 🔒 Red interna |
| MinIO API | **`9341`** | `9000` | 🛠️ Dev only |
| MinIO Console | **`9342`** | `9001` | 🛠️ Dev only |
| pgAdmin | **`5151`** | `80` | 🛠️ Dev only |
| Redis Commander | **`8283`** | `8081` | 🛠️ Dev only |

---

## 🧪 Desarrollo y testing

### Modo desarrollo (con hot-reload)

```bash
# 1. Levantar solo las dependencias (DB, cache, vector store, storage)
docker compose -f docker-compose.dev.yml up -d postgres redis chromadb minio

# 2. Frontend con hot-reload (Turbopack)
cd apps/web && npm run dev                          # → http://localhost:3121

# 3. Core API con Air (Go hot-reload)
cd services/core-api && air                        # → :8741

# 4. AI Service con Robyn en modo desarrollo
cd services/ai-service && python main.py --dev     # → :8742

# 5. Scraper con Litestar reload
cd services/scraper && python main.py --reload     # → :8743

# 6. Gateway con Liberty dev mode (recompila en cambios)
cd gateway && mvn liberty:dev                      # → :9181
```

### Comandos de tests

```bash
# Core API — tests unitarios e integración (Go)
cd services/core-api && go test ./... -v -cover

# AI Service — pytest con cobertura
cd services/ai-service && pytest --cov=app --cov-report=term-missing

# Scraper — pytest con cobertura
cd services/scraper && pytest --cov=app --cov-report=term-missing

# Frontend — Vitest (unit) + Playwright (E2E)
cd apps/web && npm test
cd apps/web && npm run test:e2e

# Gateway — Maven Failsafe (integration tests)
cd gateway && mvn verify
```

### Herramientas de administración incluidas (dev)

| Herramienta | URL | Descripción |
|-------------|-----|-------------|
| pgAdmin 4 | http://localhost:5151 | Administrador visual de PostgreSQL |
| Redis Commander | http://localhost:8283 | Explorador de claves Redis en tiempo real |
| MinIO Console | http://localhost:9342 | Explorador de buckets y objetos |
| Gateway Health | http://localhost:9181/health | MicroProfile Health (live + ready) |
| Gateway Metrics | http://localhost:9181/metrics | Métricas Prometheus |
| Scraper OpenAPI | http://localhost:9181/api/scrape/docs | Swagger UI auto-generado por Litestar |

---

## 📦 Paquetes principales por servicio

### Frontend — Next.js 16

| Paquete | Versión | Propósito |
|---------|---------|-----------|
| `tailwindcss` | v4 | Estilos utilitarios con design tokens |
| `zustand` | ^5 | Estado global (catálogo, sesión, chat) |
| `@tanstack/react-query` | ^5 | Server state management y cache de API |
| `react-hook-form` + `zod` | latest | Formularios con validación de esquema |
| `recharts` | ^2 | Gráficas de estadísticas de lectura |
| `@radix-ui/react-*` | latest | Componentes accesibles headless (dialogs, etc.) |
| `lucide-react` | latest | Iconos SVG consistentes |
| `react-markdown` | ^9 | Renderizado de respuestas del asistente |
| `pdfjs-dist` | latest | Visor de PDFs integrado en el browser |
| `next-pwa` | latest | Service Worker y manifest para PWA |

### Core API — Go + Gin

| Paquete | Propósito |
|---------|-----------|
| `gin-gonic/gin` | HTTP framework de alta performance |
| `gorm.io/gorm` | ORM con auto-migrations |
| `gorm.io/driver/postgres` | Driver PostgreSQL para GORM |
| `golang-jwt/jwt/v5` | Generación y validación de JWT |
| `aws/aws-sdk-go-v2/service/s3` | Cliente Cloudflare R2 / MinIO |
| `redis/go-redis/v9` | Cliente Redis con soporte de Pub/Sub |
| `pressly/goose/v3` | Migraciones SQL versionadas |
| `spf13/viper` | Configuración desde ENV y archivos |
| `uber-go/zap` | Logging estructurado de alta performance |
| `stretchr/testify` | Assertions y mocking para tests |

### AI Service — Python + Robyn

| Paquete | Propósito |
|---------|-----------|
| `robyn` | Framework ASGI con runtime en Rust |
| `langchain` + `langchain-openai` | Cadenas LLM y prompts para OpenRouter |
| `chromadb` | Cliente del vector store |
| `sentence-transformers` | Generación de embeddings local (`all-MiniLM-L6-v2`) |
| `redis` | Cache de respuestas LLM (TTL 24h) |
| `httpx` | HTTP async para llamadas a OpenRouter |
| `msgspec` | Serialización 10× más rápida que Pydantic |

### Scraper — Python + Litestar

| Paquete | Propósito |
|---------|-----------|
| `litestar` | Framework ASGI moderno (más rápido que FastAPI) |
| `httpx` | HTTP async concurrente para múltiples fuentes |
| `beautifulsoup4` + `lxml` | Parsing HTML para scraping de GoodReads |
| `msgspec` | DTOs tipados y validación de respuestas |
| `redis` | Job queue (LPUSH/BRPOP) y Pub/Sub para SSE |
| `aiobotocore` | Upload async de portadas a R2/MinIO |
| `tenacity` | Retry automático con backoff exponencial |

### Gateway — Jakarta EE 11 + Open Liberty

| Feature MicroProfile | Descripción |
|---------------------|-------------|
| `Jakarta REST 3.1` | Proxy inverso con `jakarta.ws.rs.client.Client` |
| `MicroProfile JWT 2.1` | Validación de tokens JWT en cada request |
| `MicroProfile Health 4.0` | Endpoints `/health/live` y `/health/ready` |
| `MicroProfile Metrics 5.1` | Métricas compatibles con Prometheus |
| `MicroProfile Fault Tolerance 4.0` | `@CircuitBreaker` y `@Retry` por backend |
| `MicroProfile OpenAPI 3.1` | Documentación de API generada automáticamente |

---

## 🤖 Modelos de IA disponibles (OpenRouter :free)

| Modelo | Tokens de contexto | Ideal para |
|--------|:-----------------:|------------|
| `meta-llama/llama-3.3-70b-instruct:free` | 128k | Chat general y recomendaciones complejas |
| `google/gemma-3-27b-it:free` | 96k | Resúmenes de libros y análisis |
| `deepseek/deepseek-r1:free` | 64k | Razonamiento profundo sobre lecturas |
| `microsoft/phi-4:free` | 16k | Respuestas rápidas y preguntas simples |
| `mistralai/mistral-7b-instruct:free` | 32k | Fallback ligero de alta disponibilidad |

El modelo por defecto se configura en `.env` y el usuario puede cambiarlo desde la interfaz en **Configuración → IA**. Las respuestas idénticas se cachean en Redis con TTL de 24 horas para reducir latencia y consumo de cuota.

---

## 🗄️ Esquema de base de datos (resumen)

```sql
-- Usuarios y autenticación
users            (id, email, username, password_hash, role, avatar_url, created_at)
refresh_tokens   (id, user_id, token_hash, expires_at)

-- Catálogo de libros
books            (id, title, subtitle, isbn_10, isbn_13, cover_url, synopsis,
				  year, pages, language, publisher, deleted_at, created_at)
authors          (id, name, bio, photo_url)
book_authors     (book_id, author_id)
genres           (id, name, slug)
book_genres      (book_id, genre_id)

-- Biblioteca personal del usuario
user_books       (id, user_id, book_id, status, rating, review,
				  started_at, finished_at, notes, created_at)
reading_logs     (id, user_book_id, pages_read, logged_at)
tags             (id, user_id, name)
book_tags        (book_id, tag_id)

-- Organización
collections      (id, user_id, name, description, is_public, created_at)
collection_books (collection_id, book_id, position)
loans            (id, user_id, book_id, borrower_name, loaned_at, due_at, returned_at)

-- Scraping
scrape_jobs      (id, user_id, query, status, result_book_id, error_message, created_at)

-- Asistente IA
chat_sessions    (id, user_id, title, created_at)
chat_messages    (id, session_id, role, content, model_used, created_at)
```

**Extensiones PostgreSQL habilitadas:** `pgvector` (búsqueda semántica) · `pg_trgm` (full-text fuzzy) · `pgcrypto` · `uuid-ossp`

---

## 🔐 Seguridad

| Aspecto | Implementación |
|---------|---------------|
| **Contraseñas** | bcrypt con cost factor 12 |
| **Tokens de acceso** | JWT RS256, expiración 15 minutos |
| **Refresh tokens** | Rotación en cada uso, TTL 7 días en Redis |
| **Aislamiento de red** | Backends en red privada Docker, nunca expuestos al host |
| **CORS** | Gateway acepta solo el origen exacto del frontend |
| **Rate limiting** | 100 req/min por IP configurable en `server.xml` |
| **Secretos** | Nunca en código ni en Git; solo en `.env` (en `.gitignore`) |
| **Validación de tipos** | msgspec (Python) · Zod (TypeScript) · Go types en todos los endpoints |
| **Soft delete** | Los libros eliminados nunca se borran físicamente (campo `deleted_at`) |

---

## ⚙️ Variables de entorno

Referencia completa de `.env.example`:

```env
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# BASE DE DATOS
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
POSTGRES_USER=bookshelf
POSTGRES_PASSWORD=changeme_strong_password   # ⚠️ Cambiar en producción
POSTGRES_DB=bookshelf

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# AUTENTICACIÓN JWT
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
JWT_SECRET=your_super_secret_key_min_32_chars
JWT_REFRESH_SECRET=another_different_secret_key
JWT_ACCESS_EXPIRY=15m
JWT_REFRESH_EXPIRY=168h                      # 7 días

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# CLOUDFLARE R2 (dejar vacío para usar MinIO en dev)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
R2_ACCOUNT_ID=
R2_ACCESS_KEY_ID=
R2_SECRET_ACCESS_KEY=
R2_BUCKET_NAME=bookshelf-storage
R2_PUBLIC_URL=                               # ej: https://pub.r2.dev/tu-bucket

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# MINIO (desarrollo local — no tocar si usas R2)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin123

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# OPENROUTER (IA)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
OPENROUTER_API_KEY=sk-or-v1-...
OPENROUTER_DEFAULT_MODEL=meta-llama/llama-3.3-70b-instruct:free
OPENROUTER_SITE_URL=http://localhost:3121    # Para identificación en OpenRouter

# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
# SCRAPER (opcional — OpenLibrary es fallback gratis)
# ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GOOGLE_BOOKS_API_KEY=
```

---

## 🗓️ Roadmap

El desarrollo sigue un plan de 10 fases progresivas. Consulta [`PHASES.md`](./PHASES.md) para el detalle completo de cada fase con checklists ejecutables.

| Fase | Nombre | Servicios |
|:----:|--------|-----------|
| **0** | Infraestructura base y monorepo | Docker, PG, Redis, ChromaDB, MinIO |
| **1** | Landing y páginas estáticas | Next.js 16 |
| **2** | Autenticación completa | Next.js · Go · PostgreSQL · Redis |
| **3** | Catálogo de libros (CRUD) | Next.js · Go · PostgreSQL · R2 |
| **4** | API Gateway | Open Liberty · Jakarta EE 11 |
| **5** | Scraper de metadatos | Litestar · Redis Jobs · MinIO |
| **6** | Colecciones, tags y estadísticas | Next.js · Go · Recharts |
| **7** | Asistente IA con memoria | Robyn · ChromaDB · OpenRouter |
| **8** | Búsqueda avanzada y recomendaciones | Go · Python · pgvector |
| **9** | PWA, configuración y pulido final | Full stack |

---

## ❓ Preguntas frecuentes

**¿Necesito una cuenta de Cloudflare o pagar algo?**
No para desarrollo. MinIO emula R2 localmente sin ningún costo ni cuenta externa. Para usar R2 en producción, Cloudflare R2 ofrece 10 GB gratuitos con egress ilimitado.

**¿Los modelos de IA tienen límite de uso?**
Los modelos `:free` de OpenRouter tienen límites diarios generosos para uso personal. Si los alcanzas en un día intenso, el asistente simplemente indica que el cupo se agotó y se retoma al día siguiente.

**¿Puedo usar Bookshelf sin conexión a internet?**
Sí, completamente. Todo el stack corre local. Solo el scraper de metadatos y el asistente IA necesitan internet para funcionar. El catálogo, estadísticas y colecciones funcionan 100% offline.

**¿Qué pasa con mis datos si actualizo a una nueva versión?**
Las migraciones de base de datos son versionadas con `goose` y se ejecutan automáticamente al iniciar. Tus datos nunca se pierden en una actualización normal.

**¿Puedo importar mis libros desde Goodreads?**
Sí, en la Fase 9 se implementa importación desde el CSV de exportación de Goodreads.

**¿Funciona en Windows / macOS / Linux?**
Sí. Docker Desktop abstrae todas las diferencias del sistema operativo.

---

## 🤝 Contribuir

¡Las contribuciones son bienvenidas! Para contribuir:

1. **Fork** del repositorio en GitHub
2. **Leer** [`PHASES.md`](./PHASES.md) para entender en qué fase encaja tu contribución
3. **Crear rama** desde `main`:
   ```bash
   git checkout -b feat/mi-feature
   # o
   git checkout -b fix/descripcion-del-bug
   ```
4. **Hacer commits** con formato convencional:
   ```
   feat(fase-3): agregar endpoint de búsqueda por ISBN
   fix(fase-2): corregir rotación de refresh tokens
   docs(readme): mejorar sección de inicio rápido
   ```
5. **Pasar los tests** antes de abrir PR:
   ```bash
   cd services/core-api && go test ./...
   cd apps/web && npm test
   ```
6. **Abrir Pull Request** describiendo el cambio y la fase a la que pertenece

> Al contribuir, aceptas que tu código queda bajo la licencia **GNU AGPL v3.0** del proyecto. Para contribuciones significativas, firma el CLA en [`CONTRIBUTING.md`](./CONTRIBUTING.md).

---

## 📄 Licencia

Bookshelf es software libre distribuido bajo la **GNU Affero General Public License v3.0 (AGPL-3.0)**.

```
Copyright (C) 2026  Bookshelf Contributors

This program is free software: you can redistribute it and/or modify
it under the terms of the GNU Affero General Public License as published
by the Free Software Foundation, either version 3 of the License, or
(at your option) any later version.
```

**¿Qué significa en la práctica?**

| Uso | ¿Permitido? |
|-----|:-----------:|
| Uso personal y self-hosted en tu PC | ✅ Libre |
| Modificar el código para tu uso personal | ✅ Libre |
| Distribuir el código modificado | ✅ Con misma licencia |
| Ofrecer como servicio web (SaaS) | ⚠️ Debes publicar tu código |
| Crear versión comercial cerrada | ❌ No permitido bajo AGPL |

Ver [`LICENSE`](./LICENSE) para el texto legal completo.

---

## 🙏 Reconocimientos

Bookshelf existe gracias a estos proyectos open source:

- **[OpenRouter](https://openrouter.ai/)** — Acceso unificado a modelos de IA gratuitos
- **[Open Liberty](https://openliberty.io/)** — Runtime Jakarta EE de IBM, ligero y production-ready
- **[ChromaDB](https://www.trychroma.com/)** — Vector store open source para memoria semántica
- **[Litestar](https://litestar.dev/)** — El framework Python ASGI moderno que FastAPI debería ser
- **[Robyn](https://robyn.tech/)** — Framework Python con core en Rust para WebSockets de alta performance
- **[pgvector](https://github.com/pgvector/pgvector)** — Extensión de vectores para PostgreSQL
- La comunidad self-hosted que inspiró este proyecto: **Gitea · Nextcloud · Immich · Paperless-ngx · Kavita**

---

<div align="center">

**📚 Bookshelf** · Tu biblioteca personal, a tu manera · AGPL-3.0

</div>
