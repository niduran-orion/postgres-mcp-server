# MCP PostgreSQL Server (Stateful and Dual Transport)

> **Fork Note**: Este es un fork del proyecto original [`@ahmedmustahid/postgres-mcp-server`](https://github.com/ahmedmustahid/postgres-mcp-server). Este fork no está publicado en npm y debe ejecutarse localmente desde el código fuente.

A Model Context Protocol (MCP) server that provides both HTTP and Stdio transports for interacting with PostgreSQL databases. This server exposes database resources and tools through both transport methods, allowing for flexible integration in different environments.

## Features

- **Dual Transport Support**: Both HTTP (StreamableHTTPServerTransport) and Stdio (StdioServerTransport)
- **Database Resources**: List tables and retrieve schema information
- **Query Tool**: Execute read-only SQL queries
- **Stateful Sessions**: HTTP transport supports session management
- **Docker Support**: Containerized deployments for both transports
- **Production Ready**: Graceful shutdown, error handling, and logging

## Quick Start

### Prerequisites

- Node.js >= 18.0.0 (para desarrollo local)
- npm
- Docker y Docker Compose (para despliegue con contenedores)
- PostgreSQL database (incluido en el docker-compose)

### Opción 1: Instalación Local (Desarrollo)

1. Clone este repositorio:
```bash
git clone https://github.com/niduran-orion/postgres-mcp-server.git
cd postgres-mcp-server
```

2. Instala las dependencias:
```bash
npm install
# o usando el Makefile
make install
```

3. Compila el proyecto:
```bash
npm run build
# o usando el Makefile
make build
```

4. Crea un archivo `.env` basado en `.env.example`:
```bash
cp .env.example .env
# Edita .env con tus credenciales
```

5. Ejecuta el servidor:
```bash
# HTTP Server (puerto 4000 por defecto)
npm run dev:http

# Stdio Server
npm run dev:stdio
```

### Opción 2: Despliegue con Docker (Recomendado)

Este proyecto incluye un `docker-compose.yml` completo que despliega:
- PostgreSQL 17
- Airflow (webserver, scheduler, init)
- MCP Server (HTTP y Stdio)

#### Pasos para despliegue:

1. Clone el repositorio:
```bash
git clone https://github.com/niduran-orion/postgres-mcp-server.git
cd postgres-mcp-server
```

2. Crea el archivo `.env` desde el template:
```bash
cp .env.example .env
```

3. Edita el archivo `.env` con tus credenciales:
```bash
nano .env
```

Configuración mínima requerida:
```bash
# PostgreSQL (compartido por Airflow y MCP)
POSTGRES_USER=airflow
POSTGRES_PASSWORD=tu_password_seguro
POSTGRES_DB=airflow

# MCP Server
POSTGRES_USERNAME=airflow
POSTGRES_PASSWORD=tu_password_seguro
POSTGRES_HOST=postgres
POSTGRES_DATABASE=airflow

# Airflow
AIRFLOW__CORE__FERNET_KEY=genera_una_key_con_python_cryptography
```

4. Genera una Fernet Key para Airflow (si no la tienes):
```bash
python3 -c "from cryptography.fernet import Fernet; print(Fernet.generate_key().decode())"
```

5. Levanta todos los servicios:
```bash
docker-compose up -d
```

6. Verifica que los servicios estén corriendo:
```bash
docker-compose ps
```

#### Acceso a los servicios:

- **Airflow Web UI**: http://localhost:8080 (usuario: `admin`, contraseña: `admin`)
- **MCP HTTP Server**: http://localhost:4000
- **MCP HTTP Endpoint**: http://localhost:4000/mcp
- **PostgreSQL**: localhost:5432

#### Comandos útiles:

```bash
# Ver logs de todos los servicios
docker-compose logs -f

# Ver logs de MCP Server
docker-compose logs -f mcp-http

# Ver logs de Airflow
docker-compose logs -f airflow-webserver

# Detener todos los servicios
docker-compose down

# Detener y eliminar volúmenes (⚠️ elimina datos de PostgreSQL)
docker-compose down -v

# Reconstruir las imágenes
docker-compose build

# Reiniciar un servicio específico
docker-compose restart mcp-http
```

### Environment Setup (Desarrollo Local)

### Environment Setup (Desarrollo Local)

Para desarrollo local sin Docker, crea un archivo `.env`:

```bash
# PostgreSQL Database Configuration
POSTGRES_USERNAME=your_username
POSTGRES_PASSWORD=your_password
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DATABASE=your_database

# HTTP Server Configuration
PORT=4000
HOST=0.0.0.0

# CORS Configuration
CORS_ORIGIN=http://localhost:8080,http://localhost:4000

# Environment
NODE_ENV=development
```

### Running the Server (Desarrollo Local)

#### Modo Desarrollo

```bash
# HTTP Server (puerto 4000 por defecto)
npm run dev:http
# o
make dev-http

# Stdio Server
npm run dev:stdio
# o
make dev-stdio
```

#### Modo Producción

```bash
# HTTP Server
npm run start:http
# o
make start-http

# Stdio Server
npm run start:stdio
# o
make start-stdio
```

## Docker/Podman Usage (Servicios Individuales)

Si solo quieres ejecutar el MCP Server sin Airflow, necesitarás crear un docker-compose simplificado o ejecutar los contenedores manualmente:

### Usando Docker (Solo MCP)

```bash
# Construir solo la imagen del MCP HTTP
docker build -f src/http/Dockerfile -t mcp-http:latest .

# Ejecutar el contenedor (asegúrate de tener PostgreSQL corriendo)
docker run -d \
  --name mcp-http \
  -p 4000:4000 \
  -e POSTGRES_USERNAME=your_user \
  -e POSTGRES_PASSWORD=your_password \
  -e POSTGRES_HOST=your_host \
  -e POSTGRES_PORT=5432 \
  -e POSTGRES_DATABASE=your_database \
  -e PORT=4000 \
  mcp-http:latest
```

```

### Usando Podman

Similar al uso de Docker, pero con comandos de Podman:

```bash
# Iniciar Podman machine (si usas macOS/Windows)
podman machine start

# Construir la imagen
podman build -f src/http/Dockerfile -t mcp-http:latest .

# Ejecutar el contenedor
podman run -d \
  --name mcp-http \
  -p 4000:4000 \
  -e POSTGRES_USERNAME=your_user \
  -e POSTGRES_PASSWORD=your_password \
  -e POSTGRES_HOST=your_host \
  -e POSTGRES_PORT=5432 \
  -e POSTGRES_DATABASE=your_database \
  -e PORT=4000 \
  mcp-http:latest
```

## Integración con Claude Desktop

Para usar este servidor con Claude Desktop, necesitas configurarlo con la ruta local del proyecto compilado.

### Configuración

1. Compila el proyecto:
```bash
npm run build
# o
make build
```

2. Edita tu archivo de configuración de Claude Desktop (`claude_desktop_config.json`):

**En macOS/Linux**: `~/Library/Application Support/Claude/claude_desktop_config.json`
**En Windows**: `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "postgres-mcp-server": {
      "command": "node",
      "args": [
        "/ruta/completa/al/proyecto/postgres-mcp-server/dist/src/stdioIndex.js"
      ],
      "env": {
        "POSTGRES_USERNAME": "your-username",
        "POSTGRES_PASSWORD": "your-password",
        "POSTGRES_HOST": "hostname",
        "POSTGRES_DATABASE": "database-name"
      }
    }
  }
}
```

> **Nota**: Reemplaza `/ruta/completa/al/proyecto` con la ruta absoluta donde clonaste este repositorio.

3. Reinicia Claude Desktop
#### Verificar que el servidor MCP está habilitado

Verifica desde la ventana de Claude Desktop:

![Claude Desktop Window](images/cd_window.png)

#### Usar el servidor MCP desde Claude Desktop

Ejemplo de prompt: Muestra la tabla `sales` del último año.

![Result](images/cd_mcp.png)


## Pruebas con MCP Inspector

MCP Inspector es una herramienta útil para probar y depurar servidores MCP.

### Instalación de MCP Inspector

```bash
npm install -g @modelcontextprotocol/inspector
# o usa npx sin instalarlo
```

### Probar el servidor Stdio

Después de compilar el proyecto, ejecuta:

```bash
npx @modelcontextprotocol/inspector node /ruta/completa/al/proyecto/dist/src/stdioIndex.js
```

> Reemplaza `/ruta/completa/al/proyecto` con la ruta donde clonaste este repositorio.

![Stdio in MCP Inspector](images/mcp_insp_stdio.png)

### Probar el servidor HTTP

1. Primero, inicia el servidor HTTP (en una terminal con las variables de entorno configuradas):

```bash
npm run start:http
# o
make start-http
```

2. En otra terminal, ejecuta el MCP Inspector:

```bash
npx @modelcontextprotocol/inspector
```

3. Selecciona `Streamable HTTP` del menú desplegable e ingresa la URL: `http://localhost:4000/mcp` (puerto actualizado)

#### MCP Tools:
![MCP Tools in MCP Inspector](images/http_tool.png)

#### MCP Resources:
![MCP Resource in MCP Inspector](images/http_resource.png)


## Configuration

### Environment Variables

Debes especificar estas variables en el archivo `.env`:

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `POSTGRES_USERNAME` | PostgreSQL username | - | Yes |
| `POSTGRES_PASSWORD` | PostgreSQL password | - | Yes |
| `POSTGRES_HOST` | PostgreSQL host | - | Yes |
| `POSTGRES_PORT` | PostgreSQL port | 5432 | No |
| `POSTGRES_DATABASE` | PostgreSQL database name | - | Yes |
| `PORT` | HTTP server port | 4000 | No |
| `HOST` | HTTP server host | 0.0.0.0 | No |
| `CORS_ORIGIN` | Allowed CORS origins (comma-separated) | localhost:8080,localhost:4000 | No |
| `NODE_ENV` | Environment mode | development | No |


## Resources

### Hello World (`hello://world`)
A simple greeting message for testing.

### Database Tables (`database://tables`)
Lists all tables in the public schema with their schema URIs.

### Database Schema (`database://tables/{tableName}/schema`)
Returns column information for a specific table.

## Tools

### query
Execute read-only SQL queries against the database.

**Parameters:**
- `sql` (string): The SQL query to execute


## Transport Differences

| Feature | HTTP Transport | Stdio Transport |
|---------|----------------|-----------------|
| Session Management | ✅ Stateful sessions | ❌ Stateless |
| Concurrent Connections | ✅ Multiple clients | ❌ Single process |
| Web Integration | ✅ REST API compatible | ❌ CLI only |
| Interactive Use | ✅ Via HTTP clients | ✅ Direct stdio |
| Docker Deployment | ✅ Web service | ✅ CLI container |

## Health Checks

The HTTP server includes a basic health check endpoint accessible at the `/health` endpoint with a GET request (returns 405 Method Not Allowed, confirming the server is responsive).

## Troubleshooting

### Problemas Comunes

1. **Errores de Conexión a la Base de Datos**
   ```bash
   # Verifica las credenciales en tu archivo .env
   # Asegúrate de que PostgreSQL esté corriendo y accesible
   psql -U your_username -h your_host -d your_database
   ```

2. **Puerto ya en Uso**
   ```bash
   # Cambia el PORT en .env o detén los servicios en conflicto
   lsof -i :3000
   # o
   netstat -tulpn | grep 3000
   ```

3. **El comando `node` no encuentra el archivo**
   ```bash
   # Asegúrate de haber compilado el proyecto primero
   npm run build
   
   # Verifica que los archivos existen en dist/
   ls -la dist/src/
   ```

4. **Problemas con Docker/Podman**
   ```bash
   # Limpia el caché de Docker
   make docker-clean
   docker system prune -a
   
   # Para Podman
   make podman-clean
   podman system prune -af --volumes
   ```

5. **Claude Desktop no detecta el servidor**
   - Verifica que la ruta en `claude_desktop_config.json` sea absoluta y correcta
   - Asegúrate de haber compilado el proyecto (`npm run build`)
   - Revisa los logs de Claude Desktop para ver errores específicos
   - Reinicia Claude Desktop después de cambiar la configuración

6. **Gestión de Sesiones (HTTP)**
   ```bash
   # Las sesiones se almacenan en memoria y se reiniciarán al reiniciar el servidor
   # Para producción, considera implementar almacenamiento de sesiones persistente
   ```

## Development

### Project Structure

```
postgres-mcp-server/
├── src/
│   ├── cli.ts              # CLI entry point
│   ├── index.ts            # HTTP server entry
│   ├── stdioIndex.ts       # Stdio server entry
│   ├── config/             # Configuration files
│   ├── http/               # HTTP transport implementation
│   ├── resources/          # MCP resources
│   ├── server/             # Core server logic
│   ├── stdio/              # Stdio transport implementation
│   ├── tools/              # MCP tools
│   └── utils/              # Utilities
├── dist/                   # Compiled output (generated)
├── docker-compose.yml      # Docker compose config
├── Makefile               # Build automation
├── package.json           # npm configuration
└── tsconfig.json          # TypeScript configuration
```

### Adding New Resources

1. Crea un nuevo archivo en `src/resources/`:
```typescript
// src/resources/myResource.ts
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';

export async function registerMyResource(server: McpServer) {
  server.resource({
    uri: 'my-resource://example',
    name: 'My Resource',
    description: 'Description of my resource',
    mimeType: 'application/json'
  }, async () => {
    return {
      contents: [{
        uri: 'my-resource://example',
        mimeType: 'application/json',
        text: JSON.stringify({ data: 'example' })
      }]
    };
  });
}
```

2. Importa y registra en `src/server/server.ts`:
```typescript
import { registerMyResource } from '../resources/myResource.js';

// En la función setupServer
await registerMyResource(server);
```

3. Recompila el proyecto:
```bash
npm run build
```

### Adding New Tools

1. Crea un nuevo archivo en `src/tools/`:
```typescript
// src/tools/myTool.ts
import { McpServer } from '@modelcontextprotocol/sdk/server/mcp.js';

export async function registerMyTool(server: McpServer) {
  server.tool({
    name: 'my-tool',
    description: 'Description of my tool',
    inputSchema: {
      type: 'object',
      properties: {
        param: {
          type: 'string',
          description: 'Parameter description'
        }
      },
      required: ['param']
    }
  }, async ({ param }) => {
    // Tool implementation
    return {
      content: [{
        type: 'text',
        text: `Result: ${param}`
      }]
    };
  });
}
```

2. Importa y registra en `src/server/server.ts`
3. Recompila el proyecto

### Running Tests

```bash
# Actualmente no hay tests configurados
npm run lint    # Linting (no configurado aún)
npm run format  # Formatting (no configurado aún)
```

## Contributing

Este es un fork personal del proyecto original. Si deseas contribuir:

1. Haz fork de este repositorio
2. Crea una rama para tu feature (`git checkout -b feature/AmazingFeature`)
3. Haz commit de tus cambios (`git commit -m 'Add some AmazingFeature'`)
4. Push a la rama (`git push origin feature/AmazingFeature`)
5. Abre un Pull Request

Para contribuciones al proyecto original, visita: https://github.com/ahmedmustahid/postgres-mcp-server

## License

MIT

## Credits

Este proyecto es un fork de [postgres-mcp-server](https://github.com/ahmedmustahid/postgres-mcp-server) por Ahmed Mustahid.

Proyecto original publicado en npm como `@ahmedmustahid/postgres-mcp-server`.

## Resources and Links

- [Model Context Protocol Documentation](https://modelcontextprotocol.io)
- [MCP SDK](https://github.com/modelcontextprotocol/sdk)
- [Proyecto Original](https://github.com/ahmedmustahid/postgres-mcp-server)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
