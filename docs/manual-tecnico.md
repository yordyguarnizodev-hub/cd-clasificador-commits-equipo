# Manual Técnico

## Clasificador de mensajes de commit

Este documento describe la arquitectura y las medidas básicas de seguridad implementadas en el sistema de clasificación de mensajes de commit.

---

# 1. Arquitectura

La solución está compuesta por un cliente, una API REST desarrollada con FastAPI, dos motores de clasificación, PostgreSQL y Ollama.

El flujo principal es:

```text
                         CLIENTE
                            |
                            | HTTP
                            v
                  +-------------------+
                  |      FastAPI      |
                  |    Puerto 8000    |
                  +-------------------+
                            |
                 +----------+----------+
                 |                     |
                 v                     v
          +-------------+       +-------------+
          | Motor ECO   |       |   Ollama    |
          |             |       | Puerto 11434|
          | Reglas      |       |             |
          +-------------+       +-------------+
                 |                     |
                 |                     |
                 +----------+----------+
                            |
                            v
                  +-------------------+
                  |    PostgreSQL     |
                  |    Puerto 5432    |
                  |                   |
                  |    Base: iadb     |
                  +-------------------+
```

## 1.1 Cliente

El cliente es cualquier aplicación capaz de realizar peticiones HTTP a la API.

Durante las pruebas de desarrollo se utilizó la documentación interactiva generada por FastAPI mediante:

```text
http://localhost:8000/docs
```

También es posible utilizar herramientas como Postman u otras aplicaciones que consuman APIs REST.

---

## 1.2 API FastAPI

La API REST constituye el componente central de la solución.

Se ejecuta mediante Uvicorn en el puerto:

```text
8000
```

Su función es recibir los mensajes de commit, seleccionar el motor de clasificación, registrar el resultado y devolver la respuesta al cliente.

La API expone los siguientes endpoints:

```text
GET  /health
POST /clasificar
GET  /inferencias
```

---

## 1.3 Motor ECO

El motor ECO es un clasificador basado en reglas y expresiones regulares.

No utiliza un modelo de inteligencia artificial y no requiere cargar el modelo de Ollama en memoria.

Busca palabras clave relacionadas con las categorías:

```text
feat
fix
docs
test
chore
refactor
```

Por ejemplo, un mensaje como:

```text
fix login bug
```

puede ser clasificado como:

```text
fix
```

El motor ECO funciona como línea base para comparar posteriormente el comportamiento del modelo de IA.

---

## 1.4 Motor Ollama

El segundo motor utiliza Ollama para realizar la clasificación mediante el modelo local:

```text
qwen2.5-coder:1.5b
```

Ollama se encuentra disponible mediante su API local en:

```text
http://localhost:11434
```

La API FastAPI envía el mensaje de commit a Ollama y procesa la respuesta para obtener una de las categorías permitidas.

---

## 1.5 PostgreSQL

PostgreSQL almacena las inferencias realizadas por la aplicación.

La base de datos utilizada es:

```text
iadb
```

PostgreSQL está disponible mediante el puerto:

```text
5432
```

La tabla principal utilizada por la aplicación es:

```text
inferencias
```

Esta tabla registra:

- Identificador de la inferencia.
- Fecha.
- Motor utilizado.
- Modelo utilizado.
- Texto de entrada.
- Clasificación obtenida.
- Latencia de la inferencia.

---

# 2. Seguridad

## 2.1 Puertos expuestos

La solución utiliza los siguientes puertos:

| Puerto | Componente | Función |
|--------|------------|---------|
| 8000 | FastAPI | Acceso a la API REST |
| 5432 | PostgreSQL | Conexión con la base de datos |
| 11434 | Ollama | API local del modelo |

El puerto 8000 permite que los clientes consuman los endpoints de la aplicación.

El puerto 5432 permite que FastAPI se comunique con PostgreSQL.

El puerto 11434 permite que FastAPI se comunique con Ollama para realizar inferencias.

En un entorno de producción se debería restringir el acceso a estos puertos según las necesidades de la arquitectura y evitar exponer servicios internos directamente a Internet.

---

## 2.2 Roles de PostgreSQL

Se utilizan dos roles principales.

### Usuario administrador

El usuario:

```text
postgres
```

se utiliza para tareas administrativas de PostgreSQL y para la configuración inicial de la base de datos.

Este usuario posee privilegios elevados y no debe utilizarse como usuario de conexión habitual de la aplicación.

### Usuario de aplicación

La aplicación utiliza:

```text
app_ia
```

Este usuario tiene privilegios mínimos sobre la base de datos.

Dispone de permisos para:

```text
SELECT
INSERT
```

sobre la tabla `inferencias`.

No se le conceden privilegios administrativos como:

```text
DROP
ALTER
```

ni permisos para modificar la estructura de la base de datos.

El objetivo es aplicar el principio de mínimo privilegio y reducir el impacto de un posible error o compromiso de la aplicación.

---

## 2.3 Manejo de secretos

Las credenciales y otros valores sensibles se almacenan mediante variables de entorno en el archivo:

```text
.env
```

Entre las variables utilizadas se encuentran:

```text
DB_HOST
DB_PORT
DB_NAME
DB_USER
DB_PASSWORD
DB_ADMIN_PASSWORD
OLLAMA_URL
MODELO_OLLAMA
MOTOR_POR_DEFECTO
```

El archivo `.env` contiene información sensible y no debe subirse al repositorio.

Por este motivo se encuentra incluido en `.gitignore`.

Para documentar las variables necesarias para configurar el proyecto se utiliza:

```text
.env.example
```

Este archivo no contiene las contraseñas reales y puede ser incluido en el repositorio.

---

## 2.4 ¿Qué hacer si se filtra una contraseña?

Si una contraseña utilizada por PostgreSQL fuera expuesta accidentalmente, se debe considerar comprometida.

El procedimiento recomendado sería:

1. Cambiar inmediatamente la contraseña afectada.
2. Actualizar el valor correspondiente en el archivo `.env`.
3. Reiniciar los servicios que utilicen la credencial.
4. Verificar que la aplicación pueda conectarse nuevamente a PostgreSQL.
5. Revisar el repositorio para comprobar que la contraseña no permanezca expuesta en otros archivos o commits.
6. Si la contraseña fue publicada en un repositorio remoto, considerar la credencial comprometida aunque posteriormente se elimine del archivo.

El archivo `.env` debe permanecer fuera del control de versiones.

---

# 3. Estado de la implementación

Al momento de elaborar este manual se verificó el funcionamiento de:

```text
GET  /health
POST /clasificar
GET  /inferencias
```

También se verificó la clasificación utilizando tanto:

```text
motor = eco
```

como:

```text
motor = ollama
```

Las inferencias realizadas fueron registradas correctamente en PostgreSQL.
