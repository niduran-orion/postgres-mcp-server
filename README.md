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

- Node.js >= 18.0.0
- npm
- PostgreSQL database (local o remota)

### Installation

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

### Environment Setup

Crea un archivo `.env` en el directorio raíz del proyecto con las credenciales de tu base de datos:

```bash
# .env
# PostgreSQL Database Configuration
POSTGRES_USERNAME=your_username
POSTGRES_PASSWORD=your_password
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DATABASE=your_database

# HTTP Server Configuration
PORT=3000
HOST=0.0.0.0

# CORS Configuration (comma-separated list of allowed origins)
CORS_ORIGIN=http://localhost:8080,http://localhost:3000

# Environment
NODE_ENV=development
```

### Running the Server

#### Modo Desarrollo

```bash
# HTTP Server (puerto 3000 por defecto)
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

### Environment Variables

## Docker/Podman Usage

### Usando Docker

1. Asegúrate de tener Docker instalado
2. Crea tu archivo `.env` con las credenciales de la base de datos
3. Ejecuta los comandos:

```bash
# Construir las imágenes
make docker-build

# Levantar el servidor HTTP
make docker-up

# Ver logs
make docker-logs

# Detener los contenedores
make docker-down

# Limpiar todo
make docker-clean
```

### Usando Podman

1. Instala Podman desde [aquí](https://podman.io/docs/installation)
2. Instala `uv` desde [aquí](https://docs.astral.sh/uv/getting-started/installation/)
3. Instala podman-compose: `uv add podman-compose` (o `uv sync` para sincronizar los paquetes en `pyproject.toml`)
4. Crea tu archivo `.env` con las credenciales

```bash
# Cargar las variables de entorno
set -a
source .env
set +a

# Iniciar Podman (si no está corriendo)
podman machine start

# Levantar el servidor HTTP
make podman-up

# Ver logs
make podman-logs

# Detener
make podman-down

# Limpiar todo
make podman-clean
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

3. Selecciona `Streamable HTTP` del menú desplegable e ingresa la URL: `http://localhost:3000/mcp` (o el puerto que hayas configurado)

#### MCP Tools:
![MCP Tools in MCP Inspector](images/http_tool.png)

#### MCP Resources:
![MCP Resource in MCP Inspector](images/http_resource.png)


## Configuration

### Environment Variables

You have to specify these inside the .env file.

| Variable | Description | Default | Required |
|----------|-------------|---------|----------|
| `POSTGRES_USERNAME` | PostgreSQL username | - | Yes |
| `POSTGRES_PASSWORD` | PostgreSQL password | - | Yes |
| `POSTGRES_HOST` | PostgreSQL host | - | Yes |
| `POSTGRES_DATABASE` | PostgreSQL database name | - | Yes |
| `PORT` | HTTP server port | 3000 | No |
| `HOST` | HTTP server host | 0.0.0.0 | No |
| `CORS_ORIGIN` | Allowed CORS origins (comma-separated) | localhost:8080,localhost:3000 | No |
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
