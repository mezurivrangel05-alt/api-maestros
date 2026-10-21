# API de maestros

API REST con Node.js, Express y PostgreSQL. La ruta de ejemplo queda disponible localmente en `http://localhost:3000/api/maestros/4`, equivalente a la forma de la URL que compartiste.

## Levantar todo en Docker

1. Instala e inicia Docker Desktop (motor Linux) y abre una terminal en la carpeta del proyecto.
2. Opcionalmente copia `.env.example` a `.env` y cambia `POSTGRES_PASSWORD`.
3. Ejecuta `docker compose up --build -d`.
4. Abre la API en `http://localhost:3001/api/maestros/4` y el n8n de este proyecto en `http://localhost:5679`.

Compose crea la imagen `api-maestros-api`, el contenedor de PostgreSQL y los volúmenes persistentes `api-maestros_postgres_data` y `api-maestros_n8n_data`. La API se publica en el puerto local 3001 para evitar conflictos con otros servicios existentes en el puerto 3000; dentro de Docker sigue escuchando en 3000. El n8n de este proyecto usa el puerto local 5679 para convivir con la instancia que ya ocupa 5678. Los servicios reinician automáticamente con Docker mediante `restart: unless-stopped`. Para apagarlos: `docker compose down`; los datos permanecen en el volumen. `docker compose down -v` también borra los datos.

Al iniciar la API, se asegura la tabla `maestros` y se insertan ejemplos para los IDs del 1 al 6 si esos registros todavía no existen. Los datos que ya estén guardados se conservan. También se hace al usar un volumen que ya estaba inicializado.

## Endpoints

| Método | Ruta | Uso |
| --- | --- | --- |
| GET | `/api/maestros` | Listar todos (acepta `limit` y `offset`) |
| GET | `/api/maestros/1` a `/api/maestros/6` | Consultar cualquiera de los seis ejemplos |
| POST | `/api/maestros` | Crear maestro |
| PUT | `/api/maestros/:id` | Reemplazar datos del maestro |
| DELETE | `/api/maestros/:id` | Eliminar maestro |
| GET | `/health` | Comprobar API y base de datos |

Ejemplo para crear:

```json
{
  "nombre": "María",
  "apellido": "López",
  "especialidad": "Ciencias",
  "correo": "maria@example.com",
  "telefono": "5551234567"
}
```

`nombre`, `apellido` y `especialidad` son obligatorios. El correo, si se informa, debe ser único.

## n8n

En el primer acceso a `http://localhost:5678`, termina la configuración inicial del usuario propietario. En n8n, importa `n8n/workflows/consulta-maestros.json` (menú **Workflows → Import from file**), guarda y activa el flujo. Una vez activo, consulta `http://api:3000/api/maestros/4` cada minuto desde la red de Compose. Para cambiar el intervalo o el id, edita el nodo en n8n. La instancia y sus flujos guardados sobreviven reinicios en `n8n_data`.

## Subir a GitHub

Inicializa o conecta este directorio a un repositorio GitHub y sube los archivos del proyecto. No subas `.env`, contraseñas ni datos reales. En una nueva carpeta sin repositorio: `git init`, `git add .`, `git commit -m "Crear API de maestros con PostgreSQL y n8n"`, después configura el remoto de GitHub y ejecuta `git push -u origin main`.

## Desplegar en Render

El archivo `render.yaml` define el servicio web Docker y una base PostgreSQL administrada. Conecta el repositorio a Render y crea un **Blueprint**; Render provisionará los recursos. El plan de base de datos del Blueprint puede requerir cambiarse según disponibilidad y costo de Render. Al desplegar, la URL pública tendrá la forma `https://<servicio>.onrender.com/api/maestros/4`.

El servicio n8n de este Compose es local. Para que el flujo consuma la API desplegada, impórtalo también en una instancia de n8n accesible por internet y cambia la URL del nodo a la URL pública de Render. No incluyas secretos de producción en el repositorio.
