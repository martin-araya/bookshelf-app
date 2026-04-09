# �� PHASES.md — Plan de Desarrollo por Fases

> Cada fase es una unidad completa: frontend + backend + base de datos.
> Flujo de trabajo: **desarrollar → dockerizar → testear → aprobar → siguiente fase**.
> Crear un chat separado por fase para resolver dudas específicas sin mezclar contextos.

***

## Tabla de contenidos

| Fase | Nombre | Stack principal | Estado |
|------|--------|----------------|--------|
| [Fase 0](#fase-0) | Infraestructura base | Docker Compose + monorepo | ⬜ Pendiente |
| [Fase 1](#fase-1) | Landing & páginas estáticas | Next.js 16 | ⬜ Pendiente |
| [Fase 2](#fase-2) | Autenticación completa | Next.js + Go/Gin + PostgreSQL | ⬜ Pendiente |
| [Fase 3](#fase-3) | Catálogo de libros (CRUD) | Next.js + Go/Gin + PG + R2 | ⬜ Pendiente |
| [Fase 4](#fase-4) | API Gateway | Open Liberty + Jakarta EE 11 | ⬜ Pendiente |
| [Fase 5](#fase-5) | Scraper de metadatos | Next.js + Litestar + Redis | ⬜ Pendiente |
| [Fase 6](#fase-6) | Colecciones, tags y estadísticas | Next.js + Go/Gin + PG | ⬜ Pendiente |
| [Fase 7](#fase-7) | Asistente IA con memoria | Next.js + Robyn + ChromaDB | ⬜ Pendiente |
| [Fase 8](#fase-8) | Búsqueda avanzada y recomendaciones | Next.js + Go + Python | ⬜ Pendiente |
| [Fase 9](#fase-9) | PWA, configuración y pulido final | Full stack | ⬜ Pendiente |

***

## Fase 0

### ��️ Infraestructura base y estructura del monorepo

**Objetivo:** Tener el esqueleto completo del proyecto listo: estructura de carpetas, docker-compose configurado con todos los servicios, redes definidas y bases de datos inicializadas. Al finalizar esta fase, un `docker compose up -d` debe levantar todos los contenedores saludables.

**Duración estimada:** 1–2 días

***

#### �� Estructura y repositorio

- [ ] Inicializar repositorio Git con `main` como rama principal
- [ ] Crear estructura de carpetas del monorepo: `apps/web`, `services/core-api`, `services/ai-service`, `services/scraper`, `gateway/`
- [ ] Agregar `LICENSE` con el texto completo de GNU AGPL v3.0
- [ ] Agregar `.gitignore` para Node, Go, Python y Java
- [ ] Agregar `.env.example` con todas las variables documentadas
- [ ] Agregar `CONTRIBUTING.md` con instrucciones básicas y mención al CLA

#### �� Docker Compose

- [ ] `docker-compose.yml` con los 11 servicios definidos (aunque algunos tengan imagen placeholder)
- [ ] `docker-compose.dev.yml` con overrides para desarrollo: volúmenes bind-mount, hot-reload, tools de dev
- [ ] Definir las 3 redes: `frontend-net`, `backend-net`, `storage-net`
- [ ] Configurar volúmenes persistentes nombrados: `postgres_data`, `redis_data`, `chromadb_data`, `minio_data`
- [ ] Todos los puertos en rangos no estándar (ver mapa de puertos en README)
- [ ] Health checks en PostgreSQL, Redis y MinIO para que otros contenedores esperen

#### �� Base de datos — PostgreSQL

- [ ] Imagen base: `ankane/pgvector:pg16` (incluye pgvector)
- [ ] Script `infra/postgres/init.sql` que instala extensiones: `pgvector`, `pg_trgm`, `pgcrypto`, `uuid-ossp`
- [ ] Crear usuario y base de datos con las variables de entorno
- [ ] Verificar que la extensión `pgvector` queda disponible

#### ⚡ Redis

- [ ] Imagen: `redis/redis-stack:latest` (incluye RedisJSON y RedisSearch)
- [ ] Configurar `redis.conf` con `maxmemory` y política `allkeys-lru`
- [ ] Persistencia RDB habilitada para no perder datos entre reinicios

#### �� ChromaDB

- [ ] Imagen: `chromadb/chroma:latest`
- [ ] Configurar persistencia en volumen nombrado
- [ ] Verificar endpoint `/api/v1/heartbeat` saludable

#### ☁️ MinIO (almacenamiento local dev)

- [ ] Imagen: `minio/minio` con comando `server /data --console-address ":9001"`
- [ ] Bucket inicial `bookshelf-storage` creado al inicio via `mc` (MinIO Client)
- [ ] Policy pública solo para la carpeta `covers/`

#### ✅ Criterios de aceptación de la Fase 0

- `docker compose up -d` sin errores
- `docker compose ps` muestra todos los servicios con estado `healthy`
- PostgreSQL acepta conexiones con las credenciales del `.env`
- Redis responde a `PING`
- MinIO Console accesible en `http://localhost:9342`
- ChromaDB responde en `http://localhost:8191/api/v1/heartbeat` (interno)

***

## Fase 1

### �� Landing, Home y páginas estáticas

**Objetivo:** Construir el frontend base de la aplicación: sistema de diseño propio, página de landing (pública), estructura de navegación, páginas informativas y skeleton de la app autenticada. No hay backend real aún — se trabaja con datos mock para definir todos los componentes visuales.

**Duración estimada:** 3–5 días

**Servicios activos:** `frontend` (Next.js 16)

***

#### �� Frontend — Next.js 16

**Setup inicial**
- [ ] `npx create-next-app@latest` con TypeScript, Tailwind CSS v4 y App Router
- [ ] Configurar Tailwind v4 con los tokens de diseño del sistema (paleta Nexus, espaciado 4px, tipografía)
- [ ] Instalar y configurar: Zustand, TanStack Query, react-hook-form, zod, lucide-react, Radix UI
- [ ] Configurar path aliases en `tsconfig.json`: `@/components`, `@/lib`, `@/hooks`
- [ ] Configurar `next.config.ts` con dominio de imágenes (localhost para MinIO, R2 para prod)
- [ ] Configurar ESLint + Prettier con reglas del proyecto

**Sistema de diseño**
- [ ] Definir CSS variables globales en `app/globals.css`: colores, espaciado, tipografía, sombras
- [ ] Componentes base: `Button` (primary, secondary, ghost, destructive), `Input`, `Label`, `Badge`
- [ ] Componentes de layout: `Container`, `Card`, `Separator`, `Skeleton`
- [ ] Tema claro/oscuro con `next-themes` y persistencia en cookie (compatible con SSR)
- [ ] Fuentes: DM Sans (body) + Geist Mono (código/puertos) via `next/font/google`

**Páginas**
- [ ] `/` — Landing page pública con hero, features, screenshots (mock), CTA de "Instalar"
- [ ] `/features` — Página de características con cards detalladas de cada módulo
- [ ] `/about` — Descripción del proyecto, stack, licencia, link al repo
- [ ] `/(app)/dashboard` — Skeleton del dashboard autenticado con datos mock
- [ ] `/(app)/library` — Skeleton del catálogo de libros con grid de cards mock
- [ ] `/(app)/book/[id]` — Skeleton de detalle de libro con todos los campos
- [ ] `/(app)/chat` — Skeleton del chat con el asistente IA
- [ ] `/not-found` — Página 404 con mensaje amigable y botón de regreso
- [ ] `error.tsx` global para errores inesperados

**Navegación y layout**
- [ ] `Navbar` responsive: logo, links de navegación, toggle tema, avatar de usuario (mock)
- [ ] `Sidebar` colapsable para la sección autenticada (desktop)
- [ ] Bottom navigation para móvil en la sección autenticada
- [ ] Breadcrumbs en páginas internas

**Componentes de libro (mock)**
- [ ] `BookCard` — card con portada, título, autor, rating, estado de lectura
- [ ] `BookGrid` — grid responsivo de `BookCard` con soporte de esqueletos
- [ ] `BookDetail` — vista completa de un libro con todos los campos
- [ ] `ReadingStatusBadge` — badge de color según estado

**Health check mínimo**
- [ ] Endpoint `/api/health` en Next.js (Route Handler) que devuelve `{ status: "ok", version: "0.1.0" }`

#### ✅ Criterios de aceptación de la Fase 1

- Landing page con lighthouse score > 90 en Performance, Accessibility, Best Practices
- Todas las páginas enlistadas accesibles sin errores 500
- Tema claro y oscuro funcionando y persistente
- Navegación fluida entre todas las rutas sin errores
- Responsive en 375px (móvil), 768px (tablet) y 1280px (desktop)
- No hay llamadas reales a API — solo datos mock estáticos

***

## Fase 2

### �� Autenticación completa

**Objetivo:** Implementar el sistema de autenticación end-to-end: registro, login, logout, refresh de tokens y protección de rutas. Al final de esta fase, el usuario puede crear una cuenta, iniciar sesión y la app recuerda la sesión entre reinicios.

**Duración estimada:** 3–4 días

**Servicios activos:** `frontend`, `core-api` (Go), `postgres`, `redis`

***

#### �� Frontend — Next.js 16

- [ ] Página `/login` — formulario con email, contraseña, validación Zod, manejo de errores del servidor
- [ ] Página `/register` — formulario con email, username, contraseña, confirmación, validación
- [ ] Página `/forgot-password` — solicitud de reset por email (flujo preparado para Fase 9)
- [ ] `AuthProvider` en layout raíz: gestiona token en memoria + refresh automático
- [ ] Middleware de Next.js para proteger rutas `/(app)/*` — redirige a `/login` si no hay sesión
- [ ] Store de Zustand `useAuthStore`: `{ user, accessToken, login, logout, refreshToken }`
- [ ] Interceptor de TanStack Query para adjuntar Bearer token en todas las peticiones
- [ ] Manejo de 401 automático: intenta refresh, si falla hace logout
- [ ] Avatar con iniciales del usuario en Navbar, menú de usuario con Logout

#### �� Backend — Go + Gin

**Estructura de carpetas:**
```
services/core-api/
├── cmd/server/main.go
├── internal/
│   ├── config/          # Viper: leer ENV
│   ├── database/        # Conexión GORM + pgvector
│   ├── cache/           # Cliente Redis
│   ├── middleware/       # JWT auth, CORS, rate limit, logger
│   ├── handlers/auth/   # Register, Login, Refresh, Logout
│   ├── models/          # User, RefreshToken (GORM structs)
│   ├── repositories/    # UserRepo, TokenRepo
│   └── services/auth/   # Lógica: hash, JWT, validación
├── migrations/          # SQL via goose
└── Dockerfile
```

- [ ] `POST /auth/register` — valida input, hashea password con bcrypt (cost 12), crea user, devuelve JWT
- [ ] `POST /auth/login` — valida credenciales, genera access token (15 min) + refresh token (7 días)
- [ ] `POST /auth/refresh` — valida refresh token en Redis, emite nuevo par de tokens (rotación)
- [ ] `POST /auth/logout` — revoca el refresh token en Redis (blacklist)
- [ ] `GET /users/me` — endpoint protegido, devuelve datos del usuario actual
- [ ] Middleware JWT: valida firma, expiración, extrae claims y los pone en el contexto Gin
- [ ] Middleware CORS: solo acepta origen del frontend
- [ ] Middleware de logging estructurado con `uber-go/zap`
- [ ] Graceful shutdown en `main.go`

#### �� Base de datos — PostgreSQL (migración 001)

```sql
-- 001_create_users.sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";

CREATE TABLE users (
	  id          UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	    email       VARCHAR(255) NOT NULL UNIQUE,
	      username    VARCHAR(50)  NOT NULL UNIQUE,
	        password_hash TEXT       NOT NULL,
	          role        VARCHAR(20)  NOT NULL DEFAULT 'user', -- 'user' | 'admin'
	            avatar_url  TEXT,
	              created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
	                updated_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW()
	                );

	                CREATE TABLE refresh_tokens (
	                	  id          UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	    user_id     UUID         NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	                	      token_hash  TEXT         NOT NULL UNIQUE,
	                	        expires_at  TIMESTAMPTZ  NOT NULL,
	                	          created_at  TIMESTAMPTZ  NOT NULL DEFAULT NOW()
	                	          );

	                	          CREATE INDEX idx_users_email    ON users(email);
	                	          CREATE INDEX idx_users_username ON users(username);
	                	          ```

	                	          #### ⚡ Redis

	                	          - [ ] Clave `refresh:token_hash` → `user_id` con TTL de 7 días
	                	          - [ ] Clave `blacklist:token_hash` para tokens revocados

	                	          #### ✅ Criterios de aceptación de la Fase 2

	                	          - Registro de usuario funcional, contraseña nunca en texto plano en DB
	                	          - Login devuelve JWT válido verificable con `jwt.io`
	                	          - Rutas `/(app)/*` redirigen a `/login` sin token
	                	          - Refresh token rota en cada uso (invalidación del anterior)
	                	          - Logout invalida el refresh token — no se puede reutilizar
	                	          - Token expirado devuelve 401, el cliente hace refresh automático

	                	          ***

	                	          ## Fase 3

	                	          ### �� Catálogo de libros — CRUD completo

	                	          **Objetivo:** El corazón de Bookshelf. El usuario puede agregar libros manualmente, ver su catálogo, editar la información, cambiar el estado de lectura, subir portadas a R2/MinIO y eliminar libros (soft delete).

	                	          **Duración estimada:** 5–7 días

	                	          **Servicios activos:** `frontend`, `core-api`, `postgres`, `redis`, `minio`

	                	          ***

	                	          #### �� Frontend — Next.js 16

	                	          - [ ] Página `/(app)/library` — grid de libros con filtros por estado, género, autor; paginación
	                	          - [ ] Página `/(app)/library/add` — formulario completo para agregar libro manualmente
	                	          - [ ] Página `/(app)/book/[id]` — vista completa: portada, todos los campos, acciones
	                	          - [ ] Página `/(app)/book/[id]/edit` — edición inline de todos los campos
	                	          - [ ] Componente `CoverUpload` — drag & drop para subir portada con preview
	                	          - [ ] Componente `ReadingStatusSelect` — selector de estado con colores
	                	          - [ ] Componente `StarRating` — rating interactivo de 1 a 5 estrellas
	                	          - [ ] Componente `NotesEditor` — textarea con soporte de Markdown básico
	                	          - [ ] Modal de confirmación para eliminar libro
	                	          - [ ] Barra de búsqueda rápida por título/autor con debounce de 300ms
	                	          - [ ] Loading skeletons en grid y en detalle
	                	          - [ ] Empty state animado cuando no hay libros

	                	          #### �� Backend — Go + Gin

	                	          - [ ] `GET    /books` — listar libros del usuario (paginación cursor-based, filtros por status/genre/tag)
	                	          - [ ] `POST   /books` — crear libro (validación completa del body)
	                	          - [ ] `GET    /books/:id` — detalle completo con autores, géneros, tags
	                	          - [ ] `PUT    /books/:id` — actualizar campos del libro (parcial, solo campos enviados)
	                	          - [ ] `DELETE /books/:id` — soft delete (campo `deleted_at`)
	                	          - [ ] `PATCH  /books/:id/status` — cambiar estado de lectura (optimizado, llamada frecuente)
	                	          - [ ] `PATCH  /books/:id/rating` — actualizar rating y review
	                	          - [ ] `GET    /books/search?q=` — búsqueda full-text con `pg_trgm` (trigram similarity)
	                	          - [ ] `POST   /upload/presign` — generar URL firmada S3 para upload de portada a R2/MinIO
	                	          - [ ] `POST   /upload/confirm` — confirmar que el upload fue exitoso y guardar URL en DB
	                	          - [ ] Cache en Redis: lista de libros del usuario (TTL 5 min, invalidado en mutaciones)

	                	          #### �� Base de datos — PostgreSQL (migración 002)

	                	          ```sql
	                	          -- 002_create_books_catalog.sql
	                	          CREATE TABLE authors (
	                	          	  id         UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	          	    name       VARCHAR(255) NOT NULL,
	                	          	      bio        TEXT,
	                	          	        photo_url  TEXT
	                	          	        );

	                	          	        CREATE TABLE books (
	                	          	        	  id          UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	          	        	    title       VARCHAR(500) NOT NULL,
	                	          	        	      subtitle    VARCHAR(500),
	                	          	        	        isbn_10     VARCHAR(10) UNIQUE,
	                	          	        	          isbn_13     VARCHAR(13) UNIQUE,
	                	          	        	            cover_url   TEXT,
	                	          	        	              synopsis    TEXT,
	                	          	        	                year        SMALLINT,
	                	          	        	                  pages       SMALLINT,
	                	          	        	                    language    VARCHAR(10) DEFAULT 'es',
	                	          	        	                      publisher   VARCHAR(255),
	                	          	        	                        deleted_at  TIMESTAMPTZ,
	                	          	        	                          created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
	                	          	        	                            updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
	                	          	        	                            );

	                	          	        	                            CREATE TABLE book_authors (book_id UUID REFERENCES books(id), author_id UUID REFERENCES authors(id), PRIMARY KEY (book_id, author_id));

	                	          	        	                            CREATE TABLE genres (id UUID PRIMARY KEY DEFAULT uuid_generate_v4(), name VARCHAR(100) NOT NULL UNIQUE, slug VARCHAR(100) NOT NULL UNIQUE);

	                	          	        	                            CREATE TABLE book_genres (book_id UUID REFERENCES books(id), genre_id UUID REFERENCES genres(id), PRIMARY KEY (book_id, genre_id));

	                	          	        	                            -- Relación usuario ↔ libro (datos personales de lectura)
	                	          	        	                            CREATE TABLE user_books (
	                	          	        	                            	  id           UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	          	        	                            	    user_id      UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	                	          	        	                            	      book_id      UUID NOT NULL REFERENCES books(id) ON DELETE CASCADE,
	                	          	        	                            	        status       VARCHAR(20) NOT NULL DEFAULT 'unread', -- unread|reading|read|abandoned|reread
	                	          	        	                            	          rating       SMALLINT CHECK (rating BETWEEN 1 AND 5),
	                	          	        	                            	            review       TEXT,
	                	          	        	                            	              started_at   DATE,
	                	          	        	                            	                finished_at  DATE,
	                	          	        	                            	                  notes        TEXT,
	                	          	        	                            	                    created_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
	                	          	        	                            	                      updated_at   TIMESTAMPTZ NOT NULL DEFAULT NOW(),
	                	          	        	                            	                        UNIQUE(user_id, book_id)
	                	          	        	                            	                        );

	                	          	        	                            	                        -- Full-text search
	                	          	        	                            	                        CREATE INDEX idx_books_fts ON books USING gin(to_tsvector('spanish', title || ' ' || COALESCE(synopsis, '')));
	                	          	        	                            	                        CREATE INDEX idx_books_trgm ON books USING gin(title gin_trgm_ops);
	                	          	        	                            	                        CREATE INDEX idx_user_books_status ON user_books(user_id, status);
	                	          	        	                            	                        ```

	                	          	        	                            	                        #### ✅ Criterios de aceptación de la Fase 3

	                	          	        	                            	                        - Agregar libro manualmente y verlo en el catálogo
	                	          	        	                            	                        - Editar todos los campos incluida la portada
	                	          	        	                            	                        - Cambiar estado de lectura desde la card y desde el detalle
	                	          	        	                            	                        - Soft delete: el libro no aparece en el listado pero existe en la DB
	                	          	        	                            	                        - Upload de portada funcional a MinIO con preview inmediato
	                	          	        	                            	                        - Búsqueda por título encuentra resultados con typos (trigram similarity)

	                	          	        	                            	                        ***

	                	          	        	                            	                        ## Fase 4

	                	          	        	                            	                        ### ☕ API Gateway — Open Liberty

	                	          	        	                            	                        **Objetivo:** Integrar el API Gateway de Jakarta EE 11 como único punto de entrada. Todo el tráfico de Next.js pasa por el gateway, que valida JWT, aplica rate limiting, hace routing hacia el backend correcto y centraliza los logs.

	                	          	        	                            	                        **Duración estimada:** 3–4 días

	                	          	        	                            	                        **Servicios activos:** `frontend`, `api-gateway`, `core-api`, `postgres`, `redis`

	                	          	        	                            	                        ***

	                	          	        	                            	                        #### ☕ Gateway — Jakarta EE 11 + Open Liberty

	                	          	        	                            	                        - [ ] `server.xml` con features: `restfulWS-3.1`, `mpJwt-2.1`, `mpHealth-4.0`, `mpMetrics-5.1`, `mpFaultTolerance-4.0`
	                	          	        	                            	                        - [ ] `GatewayResource.java` — JAX-RS resource que proxea todas las peticiones `/api/*` al backend correspondiente usando `jakarta.ws.rs.client.Client`
	                	          	        	                            	                        - [ ] Tabla de routing: `/api/books`, `/api/auth`, `/api/upload` → `http://core-api:8741`
	                	          	        	                            	                        - [ ] Tabla de routing: `/api/ai/*`, `/api/ai/ws` → `http://ai-service:8742`
	                	          	        	                            	                        - [ ] Tabla de routing: `/api/scrape/*` → `http://scraper:8743`
	                	          	        	                            	                        - [ ] `JwtFilter.java` — `ContainerRequestFilter` que valida token JWT en todas las rutas excepto `/api/auth/login` y `/api/auth/register`
	                	          	        	                            	                        - [ ] `RateLimitFilter.java` — limita a 100 req/min por IP usando `ConcurrentHashMap` + Redis
	                	          	        	                            	                        - [ ] `CorsFilter.java` — cabeceras CORS: solo acepta `http://localhost:3121`
	                	          	        	                            	                        - [ ] `HealthCheck.java` — MicroProfile Health con checks de conectividad a backends
	                	          	        	                            	                        - [ ] `GatewayMetrics.java` — contadores de peticiones por ruta con MicroProfile Metrics
	                	          	        	                            	                        - [ ] `@Retry` y `@CircuitBreaker` en las llamadas a cada backend (Fault Tolerance)
	                	          	        	                            	                        - [ ] Logging con `java.util.logging` — formato JSON para parsear con herramientas externas
	                	          	        	                            	                        - [ ] `Dockerfile` basado en `icr.io/appcafe/open-liberty:kernel-slim-java21-openj9`

	                	          	        	                            	                        #### �� Frontend — Next.js 16

	                	          	        	                            	                        - [ ] Cambiar todas las llamadas de API del cliente de `http://core-api:8741` a `http://localhost:9181`
	                	          	        	                            	                        - [ ] Actualizar el `Dockerfile` de Next.js para usar `NEXT_PUBLIC_API_URL=http://localhost:9181`
	                	          	        	                            	                        - [ ] Agregar indicador visual de estado del gateway en el dashboard (badge verde/rojo)

	                	          	        	                            	                        #### ✅ Criterios de aceptación de la Fase 4

	                	          	        	                            	                        - Todas las operaciones de Fase 2 y 3 siguen funcionando enrutadas por el gateway
	                	          	        	                            	                        - Petición sin token a ruta protegida devuelve 401 desde el gateway (sin llegar al backend)
	                	          	        	                            	                        - `http://localhost:9181/health` devuelve `{ status: "UP" }` con los checks de los backends
	                	          	        	                            	                        - Rate limiting activo: más de 100 req/min desde la misma IP devuelve 429
	                	          	        	                            	                        - Logs del gateway muestran cada petición con método, ruta, status y latencia

	                	          	        	                            	                        ***

	                	          	        	                            	                        ## Fase 5

	                	          	        	                            	                        ### ��️ Scraper de metadatos

	                	          	        	                            	                        **Objetivo:** El usuario puede buscar un libro por ISBN o título y el scraper lo enriquece automáticamente: descarga portada, sinopsis, autores, géneros y demás metadatos. Los jobs corren de forma asíncrona con actualización en tiempo real via Server-Sent Events.

	                	          	        	                            	                        **Duración estimada:** 4–5 días

	                	          	        	                            	                        **Servicios activos:** Todos los de Fase 4 + `scraper` (Litestar) + `minio`

	                	          	        	                            	                        ***

	                	          	        	                            	                        #### ��️ Backend — Python + Litestar

	                	          	        	                            	                        - [ ] Setup: `litestar` + `httpx` + `beautifulsoup4` + `msgspec` + `redis` + `boto3`
	                	          	        	                            	                        - [ ] `POST /scrape` — recibe `{ isbn?: string, title?: string }`, crea job en Redis, devuelve `{ job_id }`
	                	          	        	                            	                        - [ ] `GET  /scrape/{job_id}` — devuelve estado del job: `pending | running | done | error`
	                	          	        	                            	                        - [ ] `GET  /scrape/{job_id}/stream` — Server-Sent Events con progreso en tiempo real
	                	          	        	                            	                        - [ ] `POST /scrape/bulk` — acepta lista de hasta 50 ISBNs, crea un job por cada uno
	                	          	        	                            	                        - [ ] Worker interno (`asyncio.create_task`) que procesa la cola de jobs de Redis
	                	          	        	                            	                        - [ ] `GoogleBooksClient` — consulta `https://www.googleapis.com/books/v1/volumes?q=isbn:{isbn}`
	                	          	        	                            	                        - [ ] `OpenLibraryClient` — fallback en `https://openlibrary.org/api/books?bibkeys=ISBN:{isbn}`
	                	          	        	                            	                        - [ ] `GoodreadsScraper` — httpx + BeautifulSoup para portadas de alta resolución
	                	          	        	                            	                        - [ ] `CoverDownloader` — descarga portada y la sube directamente a R2/MinIO via aiobotocore
	                	          	        	                            	                        - [ ] `ResultMerger` — combina los resultados de las 3 fuentes priorizando calidad
	                	          	        	                            	                        - [ ] Retry automático con `tenacity` (3 intentos, backoff exponencial)
	                	          	        	                            	                        - [ ] Rate limiting propio: máximo 5 requests/seg por fuente para no ser bloqueado
	                	          	        	                            	                        - [ ] `Dockerfile` con imagen multi-stage para imagen final < 200MB

	                	          	        	                            	                        #### �� Frontend — Next.js 16

	                	          	        	                            	                        - [ ] Página `/(app)/library/import` — flujo de importación con 3 pasos
	                	          	        	                            	                          - **Paso 1:** Input de ISBN o título con sugerencias en tiempo real (debounce)
	                	          	        	                            	                            - **Paso 2:** Preview de resultados del scraper con opción de editar antes de guardar
	                	          	        	                            	                              - **Paso 3:** Confirmación y guardado en el catálogo
	                	          	        	                            	                              - [ ] Componente `ScrapingProgress` — muestra estado del job con spinner y mensajes de progreso via SSE
	                	          	        	                            	                              - [ ] Componente `BulkImport` — textarea para pegar múltiples ISBNs (uno por línea), progreso batch
	                	          	        	                            	                              - [ ] Preview de portada con el resultado del scraper antes de confirmar
	                	          	        	                            	                              - [ ] Notificación toast al completarse un job de scraping

	                	          	        	                            	                              #### �� Base de datos — PostgreSQL (migración 003)

	                	          	        	                            	                              ```sql
	                	          	        	                            	                              -- 003_create_scrape_jobs.sql
	                	          	        	                            	                              CREATE TABLE scrape_jobs (
	                	          	        	                            	                              	  id             UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	          	        	                            	                              	    user_id        UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	                	          	        	                            	                              	      query          VARCHAR(500) NOT NULL,   -- ISBN o título buscado
	                	          	        	                            	                              	        status         VARCHAR(20)  NOT NULL DEFAULT 'pending',
	                	          	        	                            	                              	          result_book_id UUID REFERENCES books(id),
	                	          	        	                            	                              	            error_message  TEXT,
	                	          	        	                            	                              	              sources_tried  TEXT[],                  -- ['google_books', 'open_library']
	                	          	        	                            	                              	                created_at     TIMESTAMPTZ NOT NULL DEFAULT NOW(),
	                	          	        	                            	                              	                  updated_at     TIMESTAMPTZ NOT NULL DEFAULT NOW()
	                	          	        	                            	                              	                  );
	                	          	        	                            	                              	                  ```

	                	          	        	                            	                              	                  #### ⚡ Redis — Job Queue

	                	          	        	                            	                              	                  ```
	                	          	        	                            	                              	                  LPUSH scrape:queue "{job_id}"          -- Encolar job
	                	          	        	                            	                              	                  SET scrape:job:{job_id} "{json}"       -- Estado del job (TTL 24h)
	                	          	        	                            	                              	                  PUBLISH scrape:updates "{job_id}:{status}"  -- Pub/Sub para SSE
	                	          	        	                            	                              	                  ```

	                	          	        	                            	                              	                  #### ✅ Criterios de aceptación de la Fase 5

	                	          	        	                            	                              	                  - Buscar por ISBN 9780743273565 encuentra "The Great Gatsby" con portada, sinopsis y autores
	                	          	        	                            	                              	                  - El progreso del scraping se actualiza en tiempo real en el UI sin polling
	                	          	        	                            	                              	                  - Si Google Books falla, OpenLibrary devuelve resultados
	                	          	        	                            	                              	                  - La portada se sube a MinIO y la URL queda guardada en la tabla `books`
	                	          	        	                            	                              	                  - Importación bulk de 10 ISBNs crea 10 jobs simultáneos con seguimiento individual
	                	          	        	                            	                              	                  - Jobs fallidos muestran el mensaje de error y permiten reintentar

	                	          	        	                            	                              	                  ***

	                	          	        	                            	                              	                  ## Fase 6

	                	          	        	                            	                              	                  ### ��️ Colecciones, tags y estadísticas de lectura

	                	          	        	                            	                              	                  **Objetivo:** Dar al usuario herramientas de organización avanzada: colecciones personalizadas, sistema de etiquetas libre, registro de progreso de páginas y un dashboard de estadísticas de lectura con gráficas.

	                	          	        	                            	                              	                  **Duración estimada:** 4–5 días

	                	          	        	                            	                              	                  **Servicios activos:** Todos los de Fase 5

	                	          	        	                            	                              	                  ***

	                	          	        	                            	                              	                  #### �� Frontend — Next.js 16

	                	          	        	                            	                              	                  - [ ] Página `/(app)/collections` — lista de colecciones del usuario con conteo de libros
	                	          	        	                            	                              	                  - [ ] Página `/(app)/collections/[id]` — libros de una colección con drag & drop para reordenar
	                	          	        	                            	                              	                  - [ ] Modal `CreateCollection` — nombre, descripción, visibilidad (privada/pública)
	                	          	        	                            	                              	                  - [ ] Componente `TagInput` — autocompletado de tags existentes al escribir, crear tags nuevos
	                	          	        	                            	                              	                  - [ ] Página `/(app)/stats` — dashboard de estadísticas con 5 gráficas (Recharts):
	                	          	        	                            	                              	                    - Libros leídos por mes (barras)
	                	          	        	                            	                              	                      - Géneros favoritos (torta)
	                	          	        	                            	                              	                        - Autores más leídos (horizontal bars)
	                	          	        	                            	                              	                          - Meta anual con barra de progreso
	                	          	        	                            	                              	                            - Streak de lectura (heatmap tipo GitHub)
	                	          	        	                            	                              	                            - [ ] Página `/(app)/loans` — lista de préstamos activos y historial
	                	          	        	                            	                              	                            - [ ] Componente `ReadingProgress` — barra de progreso de páginas actuales del libro
	                	          	        	                            	                              	                            - [ ] `ProgressLogger` — modal para registrar páginas leídas hoy

	                	          	        	                            	                              	                            #### �� Backend — Go + Gin

	                	          	        	                            	                              	                            - [ ] `GET/POST /collections` — listar y crear colecciones del usuario
	                	          	        	                            	                              	                            - [ ] `PUT/DELETE /collections/:id` — editar y eliminar colecciones
	                	          	        	                            	                              	                            - [ ] `POST /collections/:id/books` — agregar libro a colección
	                	          	        	                            	                              	                            - [ ] `DELETE /collections/:id/books/:bookId` — quitar libro de colección
	                	          	        	                            	                              	                            - [ ] `PATCH /collections/:id/reorder` — reordenar libros en colección
	                	          	        	                            	                              	                            - [ ] `GET/POST/DELETE /tags` — gestión de tags del usuario
	                	          	        	                            	                              	                            - [ ] `POST /books/:id/tags` — asignar tags a un libro
	                	          	        	                            	                              	                            - [ ] `POST /reading-logs` — registrar progreso de lectura (páginas leídas)
	                	          	        	                            	                              	                            - [ ] `GET /stats/overview` — resumen: total libros, leídos este año, en progreso
	                	          	        	                            	                              	                            - [ ] `GET /stats/by-month?year=2025` — libros terminados por mes
	                	          	        	                            	                              	                            - [ ] `GET /stats/genres` — distribución por géneros
	                	          	        	                            	                              	                            - [ ] `GET /stats/streak` — racha de días con lectura registrada
	                	          	        	                            	                              	                            - [ ] `GET/POST/PATCH /loans` — gestión de préstamos

	                	          	        	                            	                              	                            #### �� Base de datos — PostgreSQL (migraciones 004 y 005)

	                	          	        	                            	                              	                            ```sql
	                	          	        	                            	                              	                            -- 004_create_collections_tags.sql
	                	          	        	                            	                              	                            CREATE TABLE collections (
	                	          	        	                            	                              	                            	  id          UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	          	        	                            	                              	                            	    user_id     UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	                	          	        	                            	                              	                            	      name        VARCHAR(255) NOT NULL,
	                	          	        	                            	                              	                            	        description TEXT,
	                	          	        	                            	                              	                            	          is_public   BOOLEAN DEFAULT false,
	                	          	        	                            	                              	                            	            created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
	                	          	        	                            	                              	                            	            );

	                	          	        	                            	                              	                            	            CREATE TABLE collection_books (
	                	          	        	                            	                              	                            	            	  collection_id UUID REFERENCES collections(id) ON DELETE CASCADE,
	                	          	        	                            	                              	                            	            	    book_id       UUID REFERENCES books(id) ON DELETE CASCADE,
	                	          	        	                            	                              	                            	            	      position      INTEGER NOT NULL DEFAULT 0,
	                	          	        	                            	                              	                            	            	        PRIMARY KEY (collection_id, book_id)
	                	          	        	                            	                              	                            	            	        );

	                	          	        	                            	                              	                            	            	        CREATE TABLE tags (id UUID PRIMARY KEY DEFAULT uuid_generate_v4(), user_id UUID REFERENCES users(id) ON DELETE CASCADE, name VARCHAR(100) NOT NULL, UNIQUE(user_id, name));
	                	          	        	                            	                              	                            	            	        CREATE TABLE book_tags (book_id UUID REFERENCES books(id) ON DELETE CASCADE, tag_id UUID REFERENCES tags(id) ON DELETE CASCADE, PRIMARY KEY (book_id, tag_id));

	                	          	        	                            	                              	                            	            	        -- 005_create_reading_logs.sql
	                	          	        	                            	                              	                            	            	        CREATE TABLE reading_logs (
	                	          	        	                            	                              	                            	            	        	  id           UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	          	        	                            	                              	                            	            	        	    user_book_id UUID NOT NULL REFERENCES user_books(id) ON DELETE CASCADE,
	                	          	        	                            	                              	                            	            	        	      pages_read   SMALLINT NOT NULL,
	                	          	        	                            	                              	                            	            	        	        logged_at    DATE NOT NULL DEFAULT CURRENT_DATE
	                	          	        	                            	                              	                            	            	        	        );

	                	          	        	                            	                              	                            	            	        	        CREATE TABLE loans (
	                	          	        	                            	                              	                            	            	        	        	  id            UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	          	        	                            	                              	                            	            	        	        	    user_id       UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	                	          	        	                            	                              	                            	            	        	        	      book_id       UUID NOT NULL REFERENCES books(id) ON DELETE CASCADE,
	                	          	        	                            	                              	                            	            	        	        	        borrower_name VARCHAR(255) NOT NULL,
	                	          	        	                            	                              	                            	            	        	        	          loaned_at     DATE NOT NULL DEFAULT CURRENT_DATE,
	                	          	        	                            	                              	                            	            	        	        	            due_at        DATE,
	                	          	        	                            	                              	                            	            	        	        	              returned_at   DATE
	                	          	        	                            	                              	                            	            	        	        	              );
	                	          	        	                            	                              	                            	            	        	        	              ```

	                	          	        	                            	                              	                            	            	        	        	              #### ✅ Criterios de aceptación de la Fase 6

	                	          	        	                            	                              	                            	            	        	        	              - Crear colección, agregar libros, reordenarlos con drag & drop
	                	          	        	                            	                              	                            	            	        	        	              - Tags se asignan a libros y filtran el catálogo
	                	          	        	                            	                              	                            	            	        	        	              - Registrar 5 páginas leídas genera datos en el historial de lectura
	                	          	        	                            	                              	                            	            	        	        	              - Dashboard de estadísticas muestra datos reales del usuario (no mock)
	                	          	        	                            	                              	                            	            	        	        	              - Streak de lectura se incrementa al registrar progreso en días consecutivos

	                	          	        	                            	                              	                            	            	        	        	              ***

	                	          	        	                            	                              	                            	            	        	        	              ## Fase 7

	                	          	        	                            	                              	                            	            	        	        	              ### �� Asistente IA con memoria persistente

	                	          	        	                            	                              	                            	            	        	        	              **Objetivo:** El asistente IA integrado en la app puede responder preguntas sobre los libros del usuario, hacer recomendaciones basadas en historial y recordar conversaciones anteriores usando ChromaDB como memoria vectorial. Las respuestas se streaman en tiempo real vía WebSocket.

	                	          	        	                            	                              	                            	            	        	        	              **Duración estimada:** 5–7 días

	                	          	        	                            	                              	                            	            	        	        	              **Servicios activos:** Todos los de Fase 6 + `ai-service` (Robyn) + `chromadb`

	                	          	        	                            	                              	                            	            	        	        	              ***

	                	          	        	                            	                              	                            	            	        	        	              #### �� Backend — Python + Robyn

	                	          	        	                            	                              	                            	            	        	        	              - [ ] Setup: `robyn` + `langchain` + `langchain-openai` + `chromadb` + `sentence-transformers` + `redis`
	                	          	        	                            	                              	                            	            	        	        	              - [ ] `POST /ai/chat` — endpoint REST como fallback si WebSocket no disponible
	                	          	        	                            	                              	                            	            	        	        	              - [ ] `@app.websocket("/ai/ws")` — WebSocket que recibe mensajes y streamea la respuesta del LLM token a token
	                	          	        	                            	                              	                            	            	        	        	              - [ ] `ChromaMemory` — clase que gestiona 3 colecciones en ChromaDB:
	                	          	        	                            	                              	                            	            	        	        	                - `chat_history_{user_id}` — historial de conversaciones embebido
	                	          	        	                            	                              	                            	            	        	        	                  - `book_catalog_{user_id}` — sinopsis de libros del usuario para RAG
	                	          	        	                            	                              	                            	            	        	        	                    - `user_notes_{user_id}` — notas personales del usuario
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] `ContextBuilder` — recupera los 5 fragmentos más relevantes de ChromaDB para cada mensaje
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] `OpenRouterLLM` — wrapper de LangChain para OpenRouter con soporte de streaming
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] `ModelSelector` — permite cambiar modelo en runtime (Llama, Gemma, DeepSeek, Phi-4)
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Cache de respuestas en Redis (TTL 24h) para preguntas frecuentes idénticas
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] `POST /ai/recommend` — genera 5 recomendaciones basadas en libros leídos + embeddings
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] `POST /ai/summarize` — genera resumen de la sinopsis de un libro y lo guarda en ChromaDB
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] `BookIndexer` — proceso de indexación inicial: embebe todas las sinopsis del catálogo del usuario al iniciar sesión

	                	          	        	                            	                              	                            	            	        	        	                    #### �� Frontend — Next.js 16

	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Página `/(app)/chat` — UI de chat completa con lista de sesiones y área de conversación
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Componente `ChatMessage` — renderiza Markdown en las respuestas del asistente (react-markdown)
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Componente `StreamingMessage` — muestra el texto del asistente mientras se escribe (cursor parpadeante)
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Hook `useWebSocket` — gestiona conexión WebSocket con reconexión automática
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Panel lateral con sesiones de chat anteriores
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Botón "Nueva conversación" que limpia el contexto actual
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Componente `BookRecommendations` — cards de libros recomendados por el asistente
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Integración en detalle de libro: botón "Preguntarle al asistente sobre este libro"
	                	          	        	                            	                              	                            	            	        	        	                    - [ ] Indicador de modelo activo con selector en configuración

	                	          	        	                            	                              	                            	            	        	        	                    #### �� Base de datos — PostgreSQL (migración 006)

	                	          	        	                            	                              	                            	            	        	        	                    ```sql
	                	          	        	                            	                              	                            	            	        	        	                    -- 006_create_chat_history.sql
	                	          	        	                            	                              	                            	            	        	        	                    CREATE TABLE chat_sessions (
	                	          	        	                            	                              	                            	            	        	        	                    	  id         UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	          	        	                            	                              	                            	            	        	        	                    	    user_id    UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
	                	          	        	                            	                              	                            	            	        	        	                    	      title      VARCHAR(255),           -- auto-generado del primer mensaje
	                	          	        	                            	                              	                            	            	        	        	                    	        created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
	                	          	        	                            	                              	                            	            	        	        	                    	        );

	                	          	        	                            	                              	                            	            	        	        	                    	        CREATE TABLE chat_messages (
	                	          	        	                            	                              	                            	            	        	        	                    	        	  id         UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
	                	          	        	                            	                              	                            	            	        	        	                    	        	    session_id UUID NOT NULL REFERENCES chat_sessions(id) ON DELETE CASCADE,
	                	          	        	                            	                              	                            	            	        	        	                    	        	      role       VARCHAR(10) NOT NULL,   -- 'user' | 'assistant'
	                	          	        	                            	                              	                            	            	        	        	                    	        	        content    TEXT NOT NULL,
	                	          	        	                            	                              	                            	            	        	        	                    	        	          model_used VARCHAR(100),
	                	          	        	                            	                              	                            	            	        	        	                    	        	            created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
	                	          	        	                            	                              	                            	            	        	        	                    	        	            );
	                	          	        	                            	                              	                            	            	        	        	                    	        	            ```

	                	          	        	                            	                              	                            	            	        	        	                    	        	            #### ✅ Criterios de aceptación de la Fase 7

	                	          	        	                            	                              	                            	            	        	        	                    	        	            - Pregunta "¿Qué libros de ciencia ficción he leído?" devuelve respuesta basada en el catálogo real
	                	          	        	                            	                              	                            	            	        	        	                    	        	            - Las respuestas del asistente se streaman palabra por palabra en la UI
	                	          	        	                            	                              	                            	            	        	        	                    	        	            - Una nueva sesión recuerda el contexto de la conversación anterior (ChromaDB)
	                	          	        	                            	                              	                            	            	        	        	                    	        	            - El cambio de modelo en settings afecta inmediatamente al siguiente mensaje
	                	          	        	                            	                              	                            	            	        	        	                    	        	            - El cache de Redis reduce la latencia en preguntas repetidas

	                	          	        	                            	                              	                            	            	        	        	                    	        	            ***

	                	          	        	                            	                              	                            	            	        	        	                    	        	            ## Fase 8

	                	          	        	                            	                              	                            	            	        	        	                    	        	            ### �� Búsqueda avanzada y recomendaciones

	                	          	        	                            	                              	                            	            	        	        	                    	        	            **Objetivo:** Motor de búsqueda robusto con filtros combinados, búsqueda semántica (por significado, no solo palabras exactas) y sistema de recomendaciones automáticas basado en similitud de embeddings.

	                	          	        	                            	                              	                            	            	        	        	                    	        	            **Duración estimada:** 3–4 días

	                	          	        	                            	                              	                            	            	        	        	                    	        	            **Servicios activos:** Todos

	                	          	        	                            	                              	                            	            	        	        	                    	        	            ***

	                	          	        	                            	                              	                            	            	        	        	                    	        	            #### �� Frontend — Next.js 16

	                	          	        	                            	                              	                            	            	        	        	                    	        	            - [ ] Página `/(app)/search` — búsqueda avanzada con panel de filtros lateral
	                	          	        	                            	                              	                            	            	        	        	                    	        	              - Filtros: estado, género, autor, año (rango), rating, idioma, tags, colección
	                	          	        	                            	                              	                            	            	        	        	                    	        	                - Ordenar por: relevancia, título, año, rating, fecha agregado
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - Toggle: búsqueda semántica (IA) vs búsqueda exacta
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] Componente `SearchFilters` — filtros colapsables en móvil, siempre visible en desktop
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] Historial de búsquedas recientes (localStorage)
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] Página `/(app)/recommendations` — panel de recomendaciones personalizadas con explicación

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  #### �� Backend — Go + Gin

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] `POST /search` — búsqueda combinada: full-text (`tsvector`) + filtros SQL + paginación
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] `GET /search/suggestions?q=` — autocompletado en tiempo real con trigrams
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] Soporte de búsqueda semántica: el gateway redirige a Python cuando viene el flag `mode=semantic`

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  #### �� Backend — Python + Robyn

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] `POST /ai/search` — búsqueda semántica: embebe el query, busca en ChromaDB, devuelve IDs de libros relevantes
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] `GET /ai/recommendations` — top 10 libros similares al historial del usuario

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  #### ✅ Criterios de aceptación de la Fase 8

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - Búsqueda "aventuras en el espacio" encuentra libros de ciencia ficción sin necesitar esas palabras exactas (semántica)
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - Filtros combinados (género + año + rating) devuelven resultados correctos
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - Autocompletado sugiere títulos al escribir 2+ caracteres (< 150ms)

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  ***

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  ## Fase 9

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  ### �� PWA, configuración y pulido final

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  **Objetivo:** Convertir la app en una PWA instalable, agregar la página de configuración completa, optimizar performance y asegurarse de que todo esté bien probado y documentado.

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  **Duración estimada:** 4–5 días

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  ***

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  #### �� Frontend — Next.js 16

	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] PWA con `next-pwa`: manifest, service worker, íconos de app
	                	          	        	                            	                              	                            	            	        	        	                    	        	                  - [ ] Página `/(app)/settings` — configuración completa del usuario:
	                	          	        	                            	                              	                            	            	        	        	                    	        	                    - Perfil: avatar, username, email (cambiar con confirmación)
	                	          	        	                            	                              	                            	            	        	        	                    	        	                      - Seguridad: cambio de contraseña
	                	          	        	                            	                              	                            	            	        	        	                    	        	                        - Preferencias IA: modelo por defecto, temperatura, idioma del asistente
	                	          	        	                            	                              	                            	            	        	        	                    	        	                          - Metas de lectura: libros por año
	                	          	        	                            	                              	                            	            	        	        	                    	        	                            - Notificaciones: alertas de préstamos próximos a vencer
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Zona de peligro: exportar datos, eliminar cuenta
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] Exportación de catálogo en CSV y JSON
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] Importación desde CSV (compatible con Goodreads export)
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] Internacionalización base con `next-intl` (español + inglés)
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] Modo offline: catálogo disponible sin conexión via Service Worker
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] Optimización de imágenes de portadas con blur placeholder (base64)

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              #### �� Backend — Go + Gin

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] `GET  /export/json` — exporta todo el catálogo del usuario como JSON
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] `GET  /export/csv` — exporta en formato compatible con Goodreads
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] `POST /import/csv` — importa desde CSV de Goodreads (mapea campos)
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] `PATCH /users/me` — actualizar perfil, cambiar contraseña
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] `POST /users/me/avatar` — cambiar avatar (sube a R2/MinIO)
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - [ ] `DELETE /users/me` — eliminar cuenta y todos los datos (GDPR-ready)

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              #### ✅ Criterios de aceptación de la Fase 9

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - App instalable como PWA en Chrome y Safari
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Catálogo accesible sin conexión a internet
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Exportación CSV compatible con importación en Goodreads
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Cambio de contraseña funcional con confirmación vía email (o token por ahora)
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Lighthouse PWA score > 90
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Documentación de API completa en `/api/docs` (OpenAPI 3.1 via Open Liberty)

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ***

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ## �� Notas generales de desarrollo

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ### Convención de commits

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ```
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              feat(fase-2): agregar endpoint de refresh token
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              fix(fase-3): corregir paginación en listado de libros
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              docs(readme): actualizar instrucciones de setup
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              test(fase-5): agregar tests para GoogleBooksClient
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              chore(docker): optimizar imagen de core-api con multi-stage build
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ```

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ### Crear un chat por fase

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              Para cada fase, el contexto de desarrollo debe estar separado. Antes de iniciar una fase nueva, crear un chat con el resumen de:
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Stack de la fase
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Estado del docker-compose en ese momento
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Dependencias con fases anteriores
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              - Criterios de aceptación a validar

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ### Branching

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ```
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              main                    <- siempre funcional y testeado
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ├── fase/0-infra
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ├── fase/1-landing
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ├── fase/2-auth
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ├── fase/3-catalog
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              └── ...
	                	          	        	                            	                              	                            	            	        	        	                    	        	                              ```

	                	          	        	                            	                              	                            	            	        	        	                    	        	                              Merge a `main` solo cuando todos los criterios de aceptación de la fase estén cumplidos.
	                	          	        	                            	                              	                            	            	        	        	                    	        )
	                	          	        	                            	                              	                            	            	        	        	                    )
	                	          	        	                            	                              	                            	            	        	        )
	                	          	        	                            	                              	                            	            	        )
	                	          	        	                            	                              	                            	            )
	                	          	        	                            	                              	                            )
	                	          	        	                            	                              )
	                	          	        	                            )
	                	          	        )
	                	          )
	                )
)
