# Frontend Despacho - Innovatech Chile

Aplicación web desarrollada con React + Vite que permite gestionar órdenes de compra y despachos de Innovatech Chile.

## Tecnologías
- React 18
- Vite
- Tailwind CSS
- Nginx (servidor de producción)
- Docker (multi-stage build)
- GitHub Actions (CI/CD)

## Estructura del proyecto
front_despacho/
├── src/
│   ├── componentes/
│   │   └── CrudAdmin/
│   │       ├── TableCompras.jsx      # Consulta GET /api/v1/ventas
│   │       ├── TableDespachos.jsx    # Consulta GET /api/v1/despachos
│   │       ├── FormDespacho.jsx      # POST /api/v1/despachos + PUT /api/v1/ventas/{id}
│   │       └── FormCierreDespacho.jsx # PUT /api/v1/despachos/{id}
│   └── Routes/
├── Dockerfile
├── nginx.conf
└── .github/
└── workflows/
└── deploy.yml

## Comunicación con los backends

El frontend usa Nginx como reverse proxy para enrutar las peticiones:

| Ruta | Backend destino |
|---|---|
| `/api/v1/ventas` | `back-ventas:8080` (IP privada EC2) |
| `/api/v1/despachos` | `back-despachos:8081` (IP privada EC2) |

## Levantar con Docker

```bash
# Construir imagen
docker build -t front-despacho .

# Ejecutar contenedor
docker run -d --name front-despacho -p 80:80 front-despacho

# Verificar
docker ps
```

## Dockerfile (multi-stage)

- **Etapa 1 (builder):** Node 20 Alpine, instala dependencias y compila con `npm run build`
- **Etapa 2 (producción):** Nginx Alpine sirve los archivos estáticos del directorio `dist/`, usuario no root `appuser`, puerto 80

## Pipeline CI/CD

Se activa con `push` a la rama `deploy`:

1. **Build:** Construye la imagen Docker con Nginx
2. **Push:** Publica en Docker Hub (`daniel0netto/front-despacho:latest`)
3. **Deploy:** Conecta via SSH a EC2 Frontend y actualiza el contenedor

### Secrets requeridos en GitHub

| Secret | Descripción |
|---|---|
| `DOCKER_USERNAME` | Usuario de Docker Hub |
| `DOCKER_TOKEN` | Token de acceso de Docker Hub |
| `EC2_HOST` | IP pública de la instancia EC2 Frontend |
| `EC2_SSH_KEY` | Clave privada SSH (.pem) |

## Despliegue en AWS EC2

El frontend corre en una **subred pública** (`3.88.19.210`) y es el único componente accesible desde Internet. Los backends están en subred privada y solo son accesibles desde el frontend a través de la red interna de AWS y las reglas de Security Groups.

## Flujo completo de la aplicación
Usuario → Internet → EC2 Frontend (Nginx:80)
│
Nginx reverse proxy
┌─────┴──────┐
▼            ▼
back-ventas:8080  back-despachos:8081
│            │
mysql-ventas  mysql-despachos
