# Final Sistemas de Computación II - CRUD de Personas

CRUD simple de personas (nombre, apellido, DNI) dockerizado por completo: frontend, backend y base de datos corren cada uno en su propio contenedor, coordinados con Docker Compose.

## Stack

- **Frontend:** React + Vite, servido en producción por NGINX
- **Backend:** Node.js + Express
- **Base de datos:** MongoDB (contenedor local, con volumen persistente)
- **Comunicación:** NGINX actúa como reverse proxy, reenviando las peticiones `/api/*` del frontend al backend. Así solo hace falta exponer un único servicio hacia afuera.

## Estructura del proyecto

```
tpsistemas2/
├── docker-compose.yml
├── backend/
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── .env
│   ├── server.js
│   ├── models/
│   │   └── person.js
│   └── routes/
│       └── personRoutes.js
└── frontend/
    ├── Dockerfile
    ├── .dockerignore
    ├── nginx.conf
    └── src/
        └── App.jsx
```
## Variables de entorno

`backend/.env`:

```
MONGO_URI=mongodb://mongo:27017/sistemas2
PORT=3001
```

`mongo` es el nombre del servicio de Mongo en `docker-compose.yml` — Docker resuelve ese nombre dentro de la red interna `app-network`, no hace falta una IP fija.

## Cómo levantarlo

Requisito: tener Docker Desktop instalado y corriendo.

```bash
docker compose up --build
```

Esto levanta 3 contenedores:

| Servicio | Puerto | Descripción |
|---|---|---|
| `mongo` | 27017 | Base de datos MongoDB con volumen persistente (`mongo-data`) |
| `backend` | 3001 | API REST en Express, conectada a `mongo` por el nombre del servicio |
| `frontend` | 80 | React compilado, servido por NGINX, con proxy interno hacia `backend` |

Una vez levantado, la app queda disponible en:

```
http://localhost
```

Para bajar todo:

```bash
docker compose down
```

(Los datos de Mongo persisten entre reinicios gracias al volumen `mongo-data`. Para borrarlos también: `docker compose down -v`.)

## Endpoints de la API

| Método | Ruta | Descripción |
|---|---|---|
| GET | `/api/persons` | Lista todas las personas |
| GET | `/api/persons/:id` | Obtiene una persona por ID |
| POST | `/api/persons` | Crea una persona |
| PUT | `/api/persons/:id` | Actualiza una persona |
| DELETE | `/api/persons/:id` | Elimina una persona |

Todos los campos (`nombre`, `apellido`, `dni`) son obligatorios; el backend valida con Mongoose y devuelve `400` con un mensaje de error si falta alguno.

## Exposición pública (ngrok)

Para que el proyecto sea accesible desde afuera de la red local:

```bash
ngrok http 80
```

Solo se expone el puerto 80 (frontend). El NGINX de ese contenedor reenvía internamente las llamadas a `/api/` hacia el backend, así que no hace falta un segundo túnel.
