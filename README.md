# Clasificador de mensajes de commit

## 1. Descripción del proyecto

El proyecto **Clasificador de mensajes de commit** es una API REST desarrollada con FastAPI cuyo objetivo es clasificar mensajes de commit de acuerdo con categorías como `feat`, `fix`, `docs`, `test`, `chore` y `refactor`. La solución cuenta con dos motores de clasificación: un motor ECO basado en reglas y expresiones regulares, y un motor de inteligencia artificial mediante Ollama utilizando el modelo `qwen2.5-coder:1.5b`. Los resultados de las inferencias son almacenados en PostgreSQL para permitir su consulta y seguimiento. La solución se encuentra contenerizada mediante Docker Compose, facilitando su instalación, ejecución y reproducción en diferentes entornos.

---

## 2. Integrantes del equipo y perfil de hardware

### 2.1 Integrantes del equipo

Los integrantes del equipo que participaron en el desarrollo del proyecto son:

- **Integrante 1:** [Nombre del integrante]
- **Integrante 2:** [Nombre del integrante]
- **Integrante 3:** [Nombre del integrante]
- **Integrante 4:** [Nombre del integrante]

> Completar esta sección con los nombres reales de los integrantes antes de la entrega final.

### 2.2 Perfil de hardware utilizado

Durante las pruebas realizadas sobre el equipo de desarrollo se contó con aproximadamente:

- **Memoria RAM:** 15 GiB.
- **Memoria disponible durante las pruebas:** aproximadamente 10 GiB.
- **Memoria libre durante la medición:** aproximadamente 4.6 GiB.
- **Memoria Swap:** 3.7 GiB.
- **Swap utilizada durante la medición:** 0 B.
- **Sistema operativo:** Linux.
- **Python:** 3.12.3.
- **Docker:** utilizado para ejecutar la API y PostgreSQL.
- **Ollama:** utilizado para ejecutar localmente el modelo `qwen2.5-coder:1.5b`.

La memoria disponible fue suficiente para ejecutar los servicios de Docker, PostgreSQL, FastAPI y Ollama durante las pruebas realizadas.

---

## 3. Requisitos mínimos de hardware y software

### 3.1 Hardware mínimo recomendado

Para ejecutar el proyecto se recomienda como mínimo:

- Procesador de 64 bits.
- 8 GB de RAM.
- Al menos 10 GB de espacio libre en disco.
- Conexión a Internet para descargar las imágenes Docker, dependencias y modelo de Ollama.

Para trabajar cómodamente con Ollama se recomienda disponer de más memoria RAM, especialmente si se utilizan modelos de mayor tamaño.

### 3.2 Software requerido

El proyecto requiere:

- Linux.
- Git.
- Python 3.12 o superior.
- Docker.
- Docker Compose.
- Ollama.
- El modelo `qwen2.5-coder:1.5b`.

Para verificar Python:

    python3 --version

La versión utilizada durante el desarrollo fue:

    Python 3.12.3

Para verificar Git:

    git --version

Para verificar Docker:

    docker --version

Para verificar Docker Compose:

    docker compose version

Para verificar Ollama:

    ollama --version

---

## 4. Instalación paso a paso

### 4.1 Actualizar el sistema Linux

Desde una terminal Linux se puede actualizar la información de los paquetes mediante:

    sudo apt update

Opcionalmente, actualizar los paquetes instalados:

    sudo apt upgrade -y

### 4.2 Instalar Git

Si Git no está instalado:

    sudo apt install git -y

Comprobar la instalación:

    git --version

### 4.3 Instalar Python

Instalar Python y las herramientas necesarias para crear el entorno virtual:

    sudo apt install python3 python3-venv python3-pip -y

Comprobar la versión:

    python3 --version

### 4.4 Clonar el repositorio

Clonar el repositorio del proyecto:

    git clone <URL_DEL_REPOSITORIO>

Entrar en la carpeta:

    cd cd-clasificador-commits-equipo

### 4.5 Crear el entorno virtual

Crear el entorno virtual de Python:

    python3 -m venv .venv

Activarlo:

    source .venv/bin/activate

Después de activarlo, la terminal debe mostrar `.venv` en el prompt.

### 4.6 Instalar las dependencias

Actualizar pip:

    python -m pip install --upgrade pip

Instalar las dependencias del proyecto:

    pip install -r requirements.txt

Para instalar las herramientas utilizadas en las pruebas:

    pip install pytest ruff

### 4.7 Configurar las variables de entorno

Crear el archivo `.env` a partir del archivo de ejemplo:

    cp .env.example .env

Editar el archivo:

    nano .env

La configuración debe contener los valores correspondientes al entorno.

Como referencia:

    DB_NAME=iadb
    DB_USER=app_ia
    DB_HOST=db
    DB_PORT=5432
    DB_PASSWORD=<contraseña_del_usuario_de_aplicacion>
    DB_ADMIN_PASSWORD=<contraseña_del_usuario_administrador>
    OLLAMA_URL=http://host.docker.internal:11434/api/generate
    MODELO_OLLAMA=qwen2.5-coder:1.5b
    MOTOR_POR_DEFECTO=eco

Las contraseñas reales no deben publicarse en el repositorio.

El archivo `.env` se encuentra incluido en `.gitignore`.

### 4.8 Instalar y configurar Ollama

Instalar Ollama siguiendo el procedimiento correspondiente al sistema Linux.

Comprobar que Ollama esté disponible:

    ollama --version

Descargar el modelo utilizado por el proyecto:

    ollama pull qwen2.5-coder:1.5b

Comprobar los modelos instalados:

    ollama list

Comprobar que la API de Ollama responda:

    curl http://localhost:11434/api/tags

La API utiliza Ollama mediante:

    http://host.docker.internal:11434/api/generate

### 4.9 Construir y levantar los contenedores

Desde la raíz del proyecto ejecutar:

    docker compose up -d --build

Este comando construye la imagen de la API y levanta los servicios definidos en `docker-compose.yml`.

Los servicios principales son:

- `api-ia`: API FastAPI.
- `db-ia`: PostgreSQL.

### 4.10 Verificar los contenedores

Ejecutar:

    docker compose ps

Se espera observar algo similar a:

    NAME      IMAGE                                COMMAND                  SERVICE   STATUS
    api-ia    cd-clasificador-commits-equipo-api   ...                      api       Up
    db-ia     postgres:16-alpine                   ...                      db        Up (healthy)

La API queda disponible en:

    http://localhost:8000

La documentación interactiva de FastAPI queda disponible en:

    http://localhost:8000/docs

---

## 5. Verificación de funcionamiento

### 5.1 Verificar el estado de los servicios

Ejecutar:

    docker compose ps

El contenedor de la API debe aparecer como `Up`.

El contenedor de PostgreSQL debe aparecer como `Up (healthy)`.

### 5.2 Verificar los logs de la API

Si se desea comprobar el funcionamiento de FastAPI:

    docker compose logs api

Para seguir los logs en tiempo real:

    docker compose logs -f api

### 5.3 Verificar PostgreSQL

Comprobar los logs:

    docker compose logs db

El contenedor debe iniciar correctamente y el healthcheck debe indicar que PostgreSQL está saludable.

---

## 6. Prueba de los tres endpoints

La API cuenta con tres endpoints principales:

- `GET /health`
- `POST /clasificar`
- `GET /inferencias`

### 6.1 GET /health

Este endpoint permite comprobar que la API esté funcionando correctamente y que tenga conexión con PostgreSQL.

Ejecutar:

    curl http://localhost:8000/health

Una respuesta exitosa debe indicar que la API y la base de datos se encuentran disponibles.

También se puede probar desde la documentación interactiva:

    http://localhost:8000/docs

### 6.2 POST /clasificar

Este endpoint permite clasificar un mensaje de commit.

#### Prueba con el motor ECO

Ejecutar:

    curl -X POST http://localhost:8000/clasificar \
      -H "Content-Type: application/json" \
      -d '{"texto":"fix error en login","motor":"eco"}'

La respuesta debe contener información relacionada con:

- Motor utilizado.
- Modelo.
- Entrada.
- Clasificación.
- Latencia.

Por ejemplo, un mensaje que contenga `fix` puede ser clasificado como:

    fix

#### Prueba con Ollama

Ejecutar:

    curl -X POST http://localhost:8000/clasificar \
      -H "Content-Type: application/json" \
      -d '{"texto":"agregar autenticación de usuarios","motor":"ollama"}'

La respuesta debe contener la clasificación generada por el modelo configurado.

### 6.3 GET /inferencias

Este endpoint permite consultar las inferencias almacenadas en PostgreSQL.

Ejecutar:

    curl http://localhost:8000/inferencias

También se puede especificar un límite:

    curl "http://localhost:8000/inferencias?limite=20"

La respuesta debe devolver los registros almacenados por la API.

Cada registro contiene información como:

- `id`
- `fecha`
- `motor`
- `modelo`
- `entrada`
- `salida`
- `latencia_ms`

---

## 7. Pruebas del proyecto

### 7.1 Pruebas automatizadas

Las pruebas se ejecutan mediante:

    python -m pytest -v

Durante el desarrollo se verificó que las pruebas finalizaran correctamente.

### 7.2 Ruff

Para comprobar la calidad y estilo del código:

    ruff check app/

El resultado esperado es:

    All checks passed!

### 7.3 Construcción de Docker

Para comprobar que la imagen Docker se construya correctamente:

    docker build -t api-ia:ci .

La construcción debe finalizar con:

    FINISHED

### 7.4 Prueba de carga

El proyecto incluye una prueba de carga mediante k6.

El archivo utilizado es:

    tests/carga/prueba_carga.js

La prueba permitió evaluar:

- Cantidad de solicitudes.
- Latencia.
- Porcentaje de errores.
- Respuestas HTTP.
- Comportamiento con usuarios virtuales.

Los resultados obtenidos fueron:

| Métrica | Resultado |
|---|---:|
| Peticiones HTTP | 649 |
| Checks realizados | 1298 |
| Checks exitosos | 100 % |
| Checks fallidos | 0 % |
| Latencia promedio | 24.05 ms |
| p90 | 48.83 ms |
| p95 | 66.28 ms |
| Latencia máxima | 149.7 ms |
| Tasa de errores | 0.00 % |
| Usuarios virtuales máximos | 10 |

Los criterios de aceptación fueron:

- `p95 < 800 ms`
- `http_req_failed < 5 %`

Los resultados cumplieron ambos criterios.

---

## 8. Solución de problemas

Durante el desarrollo se presentaron diferentes errores. Los siguientes corresponden a problemas reales encontrados durante la implementación y las soluciones aplicadas.

### 8.1 Error de indentación en `app/main.py`

Durante el desarrollo de la API apareció:

    IndentationError: unexpected indent

El error indicaba que existía una indentación incorrecta en `app/main.py`.

El problema se encontraba alrededor de la línea donde se ejecutaba:

    filas = cur.fetchall()

La solución consistió en revisar la indentación del bloque de código y alinear correctamente las instrucciones dentro de sus respectivos bloques.

Después de corregirlo se volvió a ejecutar la API y Uvicorn pudo cargar correctamente la aplicación.

### 8.2 Ruff SIM117: `with` anidados

Ruff detectó:

    SIM117 Use a single `with` statement with multiple contexts instead of nested `with` statements

El código inicialmente utilizaba:

    with conexion() as con:
        with con.cursor() as cur:

Ruff recomendó combinar ambos contextos.

La solución aplicada fue:

    with conexion() as con, con.cursor() as cur:

Después se ejecutó:

    ruff check app/ --fix

y posteriormente se volvió a verificar el proyecto.

### 8.3 Ruff BLE001: captura genérica de `Exception`

Ruff también detectó:

    BLE001 Do not catch blind exception: `Exception`

El problema estaba en un bloque que utilizaba:

    except Exception:

Se revisó el manejo de errores y se ajustó el código para cumplir las reglas de Ruff.

Después se ejecutó nuevamente:

    ruff check app/

Finalmente se obtuvo:

    All checks passed!

### 8.4 Error 500 relacionado con Ollama

Durante las primeras pruebas con Ollama la API devolvió un error 500.

Se comprobó la comunicación entre Docker y Ollama y posteriormente se verificó que el servicio de Ollama estuviera disponible y que el modelo estuviera instalado.

Se comprobó mediante:

    curl http://localhost:11434/api/tags

También se verificaron los modelos instalados:

    ollama list

Finalmente se consiguió establecer correctamente la comunicación entre la API y Ollama.

### 8.5 `Connection refused` hacia Ollama desde Docker

En otra prueba apareció:

    ConnectionRefusedError: [Errno 111] Connection refused

La aplicación estaba intentando acceder a:

    host.docker.internal:11434

pero la conexión era rechazada.

Se verificó que Ollama estuviera ejecutándose en el sistema anfitrión y se revisó la configuración de Docker Compose.

Se utilizó en `docker-compose.yml`:

    extra_hosts:
      - "host.docker.internal:host-gateway"

y la API quedó configurada para utilizar:

    OLLAMA_URL: http://host.docker.internal:11434/api/generate

Después de levantar nuevamente los servicios, la comunicación funcionó correctamente.

### 8.6 `pytest` no encontraba el módulo `app`

Al ejecutar:

    pytest -v

apareció:

    ModuleNotFoundError: No module named 'app'

El problema ocurría al ejecutar directamente el ejecutable `pytest`, aunque el entorno virtual estaba activo.

Se comprobó que el intérprete de Python utilizado pertenecía al entorno virtual y que el módulo `app` podía importarse correctamente.

La solución utilizada para ejecutar las pruebas fue:

    python -m pytest -v

Con esta forma de ejecución se utilizó explícitamente el intérprete de Python del entorno virtual y las pruebas pasaron correctamente.

También se verificó que el módulo `app` estuviera disponible mediante Python.

### 8.7 Error de conexión de PostgreSQL mediante `localhost`

Durante las pruebas apareció:

    psycopg2.OperationalError: connection to server at "localhost" (127.0.0.1), port 5432 failed: Connection refused

El problema se produjo porque se intentó acceder a PostgreSQL mediante `localhost` desde el entorno donde no correspondía.

Dentro de Docker, la API debe comunicarse con PostgreSQL utilizando el nombre del servicio de Docker Compose:

    db

Por este motivo la configuración utiliza:

    DB_HOST=db

Después de levantar correctamente los servicios mediante Docker Compose y utilizar el nombre del servicio `db`, la conexión funcionó.

### 8.8 Los contenedores no aparecían al ejecutar `docker compose ps`

En una de las verificaciones `docker compose ps` no mostraba servicios activos.

Se comprobó que los contenedores no estaban levantados y posteriormente se ejecutó:

    docker compose up -d --build

Después se volvió a ejecutar:

    docker compose ps

Los servicios aparecieron correctamente:

    api-ia
    db-ia

PostgreSQL apareció además con estado:

    healthy

### 8.9 Prueba de respaldo y restauración

Durante la Semana 5 se realizó una prueba real de recuperación de PostgreSQL.

Primero se comprobó que existían:

    1308 registros

Después se simuló una pérdida mediante:

    docker compose exec -T db psql -U postgres -d iadb -c "TRUNCATE inferencias;"

El resultado fue:

    0 registros

Posteriormente se restauró el respaldo mediante:

    cat backups/respaldo_$(date +%F).sql | docker compose exec -T db psql -U postgres -d iadb

Finalmente se volvió a consultar la cantidad de registros y se recuperaron:

    1308 registros

La prueba confirmó que el procedimiento de respaldo y restauración funcionaba correctamente.

---

## 9. Base de datos

La aplicación utiliza PostgreSQL.

La base de datos es:

    iadb

La tabla principal es:

    inferencias

Para consultar la cantidad de registros:

    docker compose exec -T db psql -U postgres -d iadb -c "SELECT COUNT(*) FROM inferencias;"

La aplicación utiliza el usuario:

    app_ia

para realizar las operaciones necesarias sobre la base de datos.

El usuario administrativo `postgres` se utiliza para tareas administrativas y de configuración.

---

## 10. Respaldo de la base de datos

Crear el directorio de respaldos:

    mkdir -p backups

Crear un respaldo:

    docker compose exec -T db pg_dump -U postgres iadb > backups/respaldo_$(date +%F).sql

Comprobar el archivo:

    ls -lh backups/

Restaurar el respaldo:

    cat backups/respaldo_$(date +%F).sql | docker compose exec -T db psql -U postgres -d iadb

Los archivos de respaldo no deben subirse al repositorio.

El directorio `backups/` se encuentra incluido en `.gitignore`.

---

## 11. Detener el proyecto

Para detener los contenedores:

    docker compose down

Para detener los contenedores y eliminar los volúmenes:

    docker compose down -v

El segundo comando debe utilizarse con precaución porque elimina los volúmenes de Docker y, por tanto, los datos almacenados en ellos.

---

## 12. Estructura principal del proyecto

    cd-clasificador-commits-equipo/
    ├── .github/
    │   └── workflows/
    │       └── ci.yml
    ├── app/
    │   ├── __init__.py
    │   └── main.py
    ├── db/
    │   └── init.sql
    ├── docs/
    │   ├── informe-tecnico.md
    │   └── manual-tecnico.md
    ├── tests/
    │   ├── __init__.py
    │   ├── conftest.py
    │   ├── test_api.py
    │   └── carga/
    │       └── prueba_carga.js
    ├── .dockerignore
    ├── .env.example
    ├── .gitignore
    ├── docker-compose.yml
    ├── Dockerfile
    ├── requirements.txt
    └── README.md

---

## 13. Seguridad

Las credenciales reales no deben almacenarse en el repositorio.

El archivo `.env` debe permanecer fuera del control de versiones.

El archivo `.env.example` sirve para indicar las variables necesarias sin incluir las contraseñas reales.

Los archivos de respaldo ubicados en `backups/` tampoco deben publicarse.

En un entorno productivo se recomienda implementar:

- HTTPS.
- Autenticación.
- Gestión segura de secretos.
- Restricción de puertos.
- Monitoreo.
- Políticas de respaldo.
- Rotación de credenciales.

---

## 14. Limitaciones conocidas

La solución presenta las siguientes limitaciones:

- La API no implementa autenticación de usuarios.
- La API utiliza HTTP sin una capa HTTPS propia.
- Ollama depende de los recursos disponibles en el equipo donde se ejecuta.
- El modelo presenta una latencia inicial mayor durante su carga.
- El motor ECO depende de reglas y palabras clave previamente definidas.
- No se implementa actualmente una caché para mensajes repetidos.
- El almacenamiento y rotación de respaldos requiere una estrategia adicional para producción.
- El proyecto está orientado principalmente a un entorno académico y de evaluación.

---

## 15. Video de demostración

Video de demostración del funcionamiento de la solución:

**[PEGAR AQUÍ EL ENLACE AL VIDEO DE DEMOSTRACIÓN]**

El video debe mostrar como mínimo:

1. Levantamiento de los servicios mediante Docker Compose.
2. Estado de los contenedores.
3. Consulta de `GET /health`.
4. Clasificación mediante `POST /clasificar`.
5. Consulta de `GET /inferencias`.
6. Funcionamiento del motor ECO.
7. Funcionamiento del motor Ollama.
8. Evidencia de las pruebas realizadas.

---

## 16. Versión final

La versión final de entrega corresponde a:

    v1.0.0

Esta versión representa la entrega estable del sistema después de completar las pruebas, documentación, respaldo y validación final.

---

## 17. Estado del proyecto

El proyecto se encuentra preparado para la etapa final de entrega.

Se verificaron:

- Funcionamiento de la API.
- Conexión con PostgreSQL.
- Clasificación mediante ECO.
- Clasificación mediante Ollama.
- Persistencia de inferencias.
- Pruebas automatizadas.
- Calidad del código con Ruff.
- Construcción de Docker.
- Prueba de carga con k6.
- Respaldo y restauración de PostgreSQL.
- Documentación técnica.
- Integración continua mediante GitHub Actions.

El sistema queda preparado para su revisión y entrega final.
