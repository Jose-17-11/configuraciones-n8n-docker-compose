# 🤖 Configuraciones de n8n con Docker Compose

## 🌟 Descripción del Proyecto

Este repositorio es una colección de plantillas de configuración (docker-compose.yml) optimizadas para desplegar el servidor de automatización n8n en distintos niveles de complejidad y escalabilidad.

El objetivo es proporcionar soluciones plug-and-play que van desde una configuración básica con SQLite hasta arquitecturas avanzadas con PostgreSQL, Redis, y múltiples Workers para entornos de producción de alto rendimiento.

**🗂️ Estructura del Repositorio**

El proyecto se organiza mediante carpetas, donde cada una contiene una configuración de n8n completa e independiente.

```
configuraciones-n8n-docker-compose/
├── avanzados-workers-n8n/       # Configuración actual (Postgres, Redis, Main + Worker)
│   ├── docker-compose.yml
│   └── .env.example
├── ...                         # Otras plantillas (por ejemplo, con Traefik, S3/MinIO, etc.)
├── .gitignore
└── README.md                   # Este documento
```

## Clonar el repositorio 

1. Clonar el Repositorio
    ```
    git clone https://github.com/Jose-17-11/configuraciones-n8n-docker-compose.git
    cd configuraciones-n8n-docker-compose/avanzados-workers-n8n
    ```

2. Configurar Variables de Entorno

    Copia el archivo de ejemplo y edita los valores para tu entorno. Es crítico cambiar las contraseñas y la llave de encriptación (N8N_ENCRYPTION_KEY).

    Variables Clave a Configurar:
    | Variable | Descripción | Valor Recomendado |
    | :--- | :--- | :--- |
    | `DB_POSTGRES_USER` | Usuario para la base de datos PostgreSQL. | `n8nuser` |
    | `DB_POSTGRES_PASSWORD` | Contraseña segura para el usuario Postgres. | **¡Contraseña Segura!** |
    | `DB_POSTGRES_DB` | Nombre de la base de datos de n8n. | `n8ndb` |
    | `N8N_ENCRYPTION_KEY` | Clave larga y única para encriptar credenciales en n8n. | **¡Cadena aleatoria y compleja!** |

3. Levantar los Contenedores
    **Utiliza Docker Compose para construir y ejecutar el stack en segundo plano:**
    ```
    docker compose up -d
    ```

4. Verificar el Estado y Acceder
    * Verificar logs:
        ```
        docker compose logs -f
        ```
    * Acceder a n8n: El servicio n8n-main está configurado para escuchar en el puerto 5671 (para evitar conflictos con el puerto por defecto 5678).
        * URL: http://localhost:5671
        * Credenciales por defecto: admin/admin (Definidas en n8n-main y n8n-worker, ¡cámbialas tan pronto accedas!).

## 🔎 Documentación de la Plantilla avanzados-workers-n8n

Esta configuración utiliza una **arquitectura robusta de escalabilidad** (**Main + Worker**) con bases de datos persistentes.

| Componente | Imagen Base | Función Principal | Enlace a la Documentación |
| :--- | :--- | :--- | :--- |
| **`postgres`** | `postgres:17` | **Base de Datos** (Persistencia de datos de n8n). | [Variables de la Imagen (Docker Hub)](https://hub.docker.com/_/postgres) |
| **`redis`** | `redis:latest` | **Broker de Colas** (Gestiona la comunicación entre Main y Worker). | [Documentación de Redis (Docker Hub)](https://hub.docker.com/_/redis) |
| **`n8n-main`** | `n8nio/n8n:latest` | **Instancia Principal** (Interfaz Web, Editor, API, Encola tareas). | [Documentación General de n8n](https://docs.n8n.io/hosting/) |
| **`n8n-worker`** | `n8nio/n8n:latest` | **Worker** (Procesamiento y ejecución de flujos en segundo plano). | [n8n Scaling: Queue Mode](https://docs.n8n.io/hosting/scaling/queue-mode/) |

---

## 🔑 Variables de Entorno Clave

| Variable / Comando | Servicio | Propósito | Enlace a la Documentación |
| :--- | :---: | :--- | :--- |
| `DB_TYPE` / `DB_POSTGRESDB_HOST` | Ambos | Configuración de la conexión a la base de datos PostgreSQL. | [n8n Database Configuration](https://docs.n8n.io/hosting/configuration/supported-databases-settings/#postgresdb) |
| `N8N_PORT=5671` | `n8n-main` | Puerto interno donde la aplicación web de n8n escucha las peticiones. | [n8n Environment Variables](https://docs.n8n.io/hosting/configuration/environment-variables/) |
| `EXECUTIONS_MODE=queue` | Ambos | Define que las ejecuciones deben ser gestionadas por la cola de Redis. | [n8n Scaling: Queue Mode](https://docs.n8n.io/hosting/scaling/queue-mode/) |
| `command: worker --concurrency=5` | `n8n-worker` | Define el rol del contenedor como Worker y establece el número de tareas paralelas. | [n8n Worker Configuration](https://docs.n8n.io/hosting/scaling/queue-mode/#worker-configuration) |
| `N8N_ENCRYPTION_KEY` | Ambos | Clave para cifrar credenciales. **Crítico para seguridad.** | [n8n Security Environment Vars](https://docs.n8n.io/hosting/configuration/configuration-examples/encryption-key/#set-a-custom-encryption-key) |
| `N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS` | Ambos | Fuerza la corrección de permisos para archivos de configuración. | [n8n Security Environment Vars](https://docs.n8n.io/hosting/configuration/environment-variables/security/) |
| `OFFLOAD_MANUAL_EXECUTIONS_TO_WORKERS=true` | `n8n-main` | Envía las ejecuciones iniciadas desde el editor al Worker. | [n8n Worker Offloading](https://docs.n8n.io/hosting/scaling/queue-mode/) |
| `N8N_BLOCK_ENV_ACCESS_IN_NODE=false` | Ambos | Controla si el código de los flujos puede acceder a las variables de entorno. | [n8n Security Environment Vars](https://docs.n8n.io/hosting/configuration/environment-variables/security/) |