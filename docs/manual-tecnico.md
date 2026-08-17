# Manual técnico

## Clasificador de mensajes de commit

Este documento describe la arquitectura, funcionamiento, seguridad, endpoints, modelo de datos, respaldo y restauración, decisiones de diseño y limitaciones implementadas en el sistema de clasificación de mensajes de commit.

---

## 1. Arquitectura

La solución está compuesta por un cliente, una API REST desarrollada con FastAPI, dos motores de clasificación, PostgreSQL y Ollama.

El flujo principal es:

    CLIENTE
        |
        | HTTP
        v
    +-------------------+
    |      FastAPI      |
    |    Puerto 8000    |
    +-------------------+
        |
        +--------------------+
        |                    |
        v                    v
    +-------------+      +-------------+
    | Motor ECO   |      |   Ollama    |
    |             |      | Puerto 11434|
    | Reglas      |      |             |
    +-------------+      +-------------+
        |                    |
        +---------+----------+
                  |
                  v
          +-------------------+
          |    PostgreSQL     |
          |    Puerto 5432    |
          |                   |
          |    Base: iadb     |
          +-------------------+

### 1.1 Cliente

El cliente es cualquier aplicación capaz de realizar peticiones HTTP a la API.

Durante las pruebas de desarrollo se utilizó la documentación interactiva generada por FastAPI mediante:

http://localhost:8000/docs

También es posible utilizar herramientas como Postman u otras aplicaciones que consuman APIs REST.

### 1.2 API FastAPI

La API REST constituye el componente central de la solución.

Se ejecuta mediante Uvicorn en el puerto 8000.

Su función es recibir los mensajes de commit, seleccionar el motor de clasificación, registrar el resultado y devolver la respuesta al cliente.

La API expone los siguientes endpoints:

- GET /health
- POST /clasificar
- GET /inferencias

### 1.3 Motor ECO

El motor ECO es un clasificador basado en reglas y expresiones regulares.

No utiliza un modelo de inteligencia artificial y no requiere cargar el modelo de Ollama en memoria.

Busca palabras clave relacionadas con las categorías:

- feat
- fix
- docs
- test
- chore
- refactor

Por ejemplo, un mensaje como "fix login bug" puede ser clasificado como "fix".

El motor ECO funciona como línea base para comparar posteriormente el comportamiento del modelo de IA.

### 1.4 Motor Ollama

El segundo motor utiliza Ollama para realizar la clasificación mediante el modelo local:

qwen2.5-coder:1.5b

Ollama se encuentra disponible mediante su API local en:

http://localhost:11434

La API FastAPI envía el mensaje de commit a Ollama y procesa la respuesta para obtener una de las categorías permitidas.

### 1.5 PostgreSQL

PostgreSQL almacena las inferencias realizadas por la aplicación.

La base de datos utilizada es:

iadb

PostgreSQL está disponible mediante el puerto 5432.

La tabla principal utilizada por la aplicación es:

inferencias

Esta tabla registra:

- Identificador de la inferencia.
- Fecha.
- Motor utilizado.
- Modelo utilizado.
- Texto de entrada.
- Clasificación obtenida.
- Latencia de la inferencia.

---

## 2. Seguridad

### 2.1 Puertos expuestos

La solución utiliza los siguientes puertos:

| Puerto | Componente | Función |
|---|---|---|
| 8000 | FastAPI | Acceso a la API REST |
| 5432 | PostgreSQL | Conexión con la base de datos |
| 11434 | Ollama | API local del modelo |

El puerto 8000 permite que los clientes consuman los endpoints de la aplicación.

El puerto 5432 permite que FastAPI se comunique con PostgreSQL.

El puerto 11434 permite que FastAPI se comunique con Ollama para realizar inferencias.

En un entorno de producción se debería restringir el acceso a estos puertos según las necesidades de la arquitectura y evitar exponer servicios internos directamente a Internet.

### 2.2 Roles de PostgreSQL

Se utilizan dos roles principales.

#### Usuario administrador

El usuario `postgres` se utiliza para tareas administrativas de PostgreSQL y para la configuración inicial de la base de datos.

Este usuario posee privilegios elevados y no debe utilizarse como usuario de conexión habitual de la aplicación.

#### Usuario de aplicación

La aplicación utiliza:

app_ia

Este usuario tiene privilegios mínimos sobre la base de datos.

Dispone de permisos para:

- SELECT
- INSERT

sobre la tabla `inferencias`.

No se le conceden privilegios administrativos como:

- DROP
- ALTER

ni permisos para modificar la estructura de la base de datos.

El objetivo es aplicar el principio de mínimo privilegio y reducir el impacto de un posible error o compromiso de la aplicación.

### 2.3 Manejo de secretos

Las credenciales y otros valores sensibles se almacenan mediante variables de entorno en el archivo:

.env

Entre las variables utilizadas se encuentran:

- DB_HOST
- DB_PORT
- DB_NAME
- DB_USER
- DB_PASSWORD
- DB_ADMIN_PASSWORD
- OLLAMA_URL
- MODELO_OLLAMA
- MOTOR_POR_DEFECTO

El archivo `.env` contiene información sensible y no debe subirse al repositorio.

Por este motivo se encuentra incluido en `.gitignore`.

Para documentar las variables necesarias para configurar el proyecto se utiliza:

.env.example

Este archivo no contiene las contraseñas reales y puede ser incluido en el repositorio.

### 2.4 ¿Qué hacer si se filtra una contraseña?

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

## 3. Estado de la implementación

Al momento de elaborar este manual se verificó el funcionamiento de:

- GET /health
- POST /clasificar
- GET /inferencias

También se verificó la clasificación utilizando tanto el motor `eco` como el motor `ollama`.

Las inferencias realizadas fueron registradas correctamente en PostgreSQL.

---

## 4. Respaldo y restauración de la base de datos

### 4.1 Objetivo

El sistema utiliza PostgreSQL para almacenar las inferencias realizadas por la API.

Para garantizar la recuperación de los datos ante una pérdida accidental, se implementó y probó un procedimiento de respaldo y restauración.

### 4.2 Creación del respaldo

El respaldo se genera mediante `pg_dump` desde el contenedor de PostgreSQL.

Comandos utilizados:

    mkdir -p backups

    docker compose exec -T db pg_dump -U postgres iadb > backups/respaldo_$(date +%F).sql

    ls -lh backups/

El archivo generado contiene la información necesaria para recuperar la base de datos.

### 4.3 Verificación del respaldo

Antes de realizar la prueba de restauración se verificó la cantidad de registros almacenados mediante:

    docker compose exec -T db psql -U postgres -d iadb -c "SELECT COUNT(*) FROM inferencias;"

La cantidad inicial registrada fue:

1308 registros

### 4.4 Simulación de pérdida de datos

Para comprobar el procedimiento de recuperación se simuló una pérdida de datos mediante:

    docker compose exec -T db psql -U postgres -d iadb -c "TRUNCATE inferencias;"

Después de ejecutar el comando se verificó nuevamente la cantidad de registros.

El resultado fue:

0 registros

Esto permitió comprobar que los datos de la tabla habían sido eliminados temporalmente para realizar la prueba de recuperación.

### 4.5 Restauración

Se restauró la información utilizando el archivo de respaldo generado anteriormente:

    cat backups/respaldo_$(date +%F).sql | docker compose exec -T db psql -U postgres -d iadb

Después de la restauración se volvió a consultar la cantidad de registros mediante:

    docker compose exec -T db psql -U postgres -d iadb -c "SELECT COUNT(*) FROM inferencias;"

El resultado fue:

1308 registros

Por lo tanto, la prueba de respaldo y restauración fue exitosa, ya que se recuperó la misma cantidad de registros que existía antes de la simulación de pérdida de datos.

El proceso comprobado fue:

- Antes del desastre: 1308 registros.
- Después de TRUNCATE: 0 registros.
- Después de restaurar: 1308 registros.

### 4.6 Periodicidad propuesta

Se propone realizar un respaldo diario de la base de datos durante un periodo de baja actividad.

También se recomienda conservar varias copias históricas para permitir la recuperación ante diferentes escenarios de pérdida de información.

### 4.7 Responsable

El procedimiento de respaldo y restauración será responsabilidad del encargado de administración y mantenimiento de la solución.

Esta persona deberá verificar periódicamente que los respaldos se generen correctamente y realizar pruebas de restauración de forma periódica.

### 4.8 Protección de los archivos de respaldo

Los archivos de respaldo pueden contener información almacenada por la aplicación, por lo que no deben publicarse en el repositorio.

El directorio utilizado para los respaldos:

backups/

se encuentra incluido en `.gitignore`.

---

## 5. Endpoints de la API

### 5.1 GET /health

Permite verificar la disponibilidad de la API y la conexión con PostgreSQL.

#### Solicitud

    GET /health

#### Respuesta exitosa

    {
      "estado": "ok",
      "base_datos": "ok"
    }

#### Código de respuesta

200 OK

Si la base de datos no está disponible, la API responde con:

503 Service Unavailable

### 5.2 POST /clasificar

Permite clasificar un mensaje de commit utilizando el motor ECO o el motor Ollama.

#### Solicitud

    POST /clasificar
    Content-Type: application/json

#### Ejemplo utilizando ECO

    {
      "texto": "fix error en login",
      "motor": "eco"
    }

#### Respuesta

    {
      "motor": "eco",
      "modelo": "reglas-v1",
      "entrada": "fix error en login",
      "tipo": "fix",
      "latencia_ms": 1
    }

El valor de `latencia_ms` depende del tiempo real de ejecución.

#### Ejemplo utilizando Ollama

    {
      "texto": "agregar autenticación de usuarios",
      "motor": "ollama"
    }

El motor Ollama utiliza el modelo configurado en:

MODELO_OLLAMA

#### Códigos de respuesta

- 200 OK
- 400 Bad Request

El código 400 se devuelve cuando se especifica un motor diferente de:

- eco
- ollama

### 5.3 GET /inferencias

Devuelve las últimas inferencias almacenadas en PostgreSQL.

#### Solicitud

    GET /inferencias

También permite especificar un límite:

    GET /inferencias?limite=20

#### Información devuelta

Cada registro contiene:

- id
- fecha
- motor
- modelo
- entrada
- salida
- latencia_ms

Los registros se ordenan desde el más reciente al más antiguo.

#### Ejemplo

    [
      {
        "id": 1308,
        "fecha": "2026-08-16T...",
        "motor": "eco",
        "modelo": "reglas-v1",
        "entrada": "fix error login",
        "salida": "fix",
        "latencia_ms": 1
      }
    ]

---

## 6. Modelo de datos

La aplicación utiliza PostgreSQL como sistema de gestión de base de datos.

La base de datos utilizada es:

iadb

La tabla principal utilizada por la aplicación es:

inferencias

### 6.1 Tabla inferencias

| Campo | Descripción |
|---|---|
| `id` | Identificador único de la inferencia |
| `fecha` | Fecha y hora en que se realizó la inferencia |
| `motor` | Motor utilizado para clasificar el mensaje |
| `modelo` | Modelo o versión del clasificador utilizado |
| `entrada` | Mensaje de commit recibido |
| `salida` | Categoría obtenida |
| `latencia_ms` | Tiempo empleado para realizar la clasificación en milisegundos |

Las categorías utilizadas por el sistema son:

- feat
- fix
- docs
- test
- chore
- refactor

Las inferencias son registradas después de procesar cada solicitud de clasificación.

---

## 7. Decisiones de diseño y limitaciones

### 7.1 Uso de FastAPI

Se seleccionó FastAPI para implementar la API REST debido a su integración con Python, su soporte para validación mediante Pydantic y la generación automática de documentación interactiva.

La documentación puede consultarse mediante:

http://localhost:8000/docs

### 7.2 Uso de un motor ECO

Se implementó un motor basado en reglas como línea base del sistema.

Esta decisión permite disponer de un clasificador sencillo, rápido y con bajo consumo de recursos, además de servir como referencia para comparar el comportamiento del modelo de inteligencia artificial.

Una limitación del motor ECO es que depende de las palabras y patrones definidos en las reglas. Por esta razón puede no clasificar correctamente mensajes que utilicen expresiones diferentes a las contempladas.

### 7.3 Uso de Ollama

Ollama permite ejecutar localmente el modelo:

qwen2.5-coder:1.5b

Esto evita depender de un servicio externo de inteligencia artificial para realizar las inferencias.

Una limitación es que la inferencia requiere más recursos y presenta una latencia mayor que el motor ECO.

Durante las pruebas se observó además un tiempo elevado en la primera ejecución del modelo, seguido por tiempos menores en las ejecuciones posteriores.

### 7.4 PostgreSQL

PostgreSQL se utiliza para almacenar las inferencias y permitir su consulta posterior.

Se utiliza un usuario de aplicación con privilegios limitados para reducir los riesgos asociados a la operación de la API.

Una limitación de la implementación actual es que la solución está diseñada principalmente para un entorno de desarrollo y evaluación, por lo que un despliegue productivo requeriría controles adicionales de seguridad, monitoreo y disponibilidad.

### 7.5 Docker Compose

Docker Compose permite ejecutar los componentes principales de la solución de manera coordinada:

- API
- PostgreSQL

La API se conecta al servicio de PostgreSQL mediante el nombre:

db

y ambos servicios utilizan una red Docker común.

Esta configuración facilita la instalación y reproducción del entorno.

### 7.6 Limitaciones actuales

Entre las principales limitaciones identificadas se encuentran:

- La API no implementa autenticación de usuarios.
- La API utiliza HTTP sin una capa HTTPS propia.
- Ollama depende de los recursos disponibles en el equipo donde se ejecuta.
- El modelo presenta una latencia inicial mayor durante su carga.
- El motor ECO depende de reglas y palabras clave previamente definidas.
- El sistema no implementa actualmente un mecanismo de caché para mensajes repetidos.
- El almacenamiento y rotación de respaldos requiere una estrategia adicional para un entorno productivo.
- El sistema está orientado principalmente a un entorno académico y de evaluación.

Para un entorno productivo sería necesario complementar la solución con autenticación, HTTPS, monitoreo, gestión centralizada de secretos, políticas de respaldo más robustas y mecanismos adicionales de escalabilidad.

---

## 8. Verificación técnica final

Como parte de la validación de la solución se realizaron pruebas funcionales, pruebas de acceso, pruebas de conectividad, pruebas de disponibilidad, pruebas de persistencia y pruebas de carga.

Las pruebas funcionales automatizadas fueron ejecutadas mediante:

    python -m pytest -v

El resultado fue:

    5 passed

También se verificó el estilo del código mediante:

    ruff check app/

La verificación de Ruff finalizó correctamente con todos los controles aprobados.

La imagen Docker de la API fue construida correctamente mediante:

    docker build -t api-ia:ci .

La prueba de carga con k6 obtuvo los siguientes resultados:

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
| Tasa de errores HTTP | 0.00 % |
| Usuarios virtuales máximos | 10 |

Los criterios de aceptación establecidos fueron:

- p95 menor a 800 ms.
- Tasa de errores menor al 5 %.

Los resultados obtenidos fueron:

- p95: 66.28 ms.
- Tasa de errores: 0.00 %.

Por lo tanto, la prueba de carga cumplió los criterios establecidos.

---

## 9. Caracterización del modelo Ollama

Durante la caracterización del modelo `qwen2.5-coder:1.5b` se realizaron cinco mediciones.

| Ejecución | Tiempo real |
|---|---:|
| 1 | 10.16 s |
| 2 | 1.43 s |
| 3 | 1.70 s |
| 4 | 1.75 s |
| 5 | 1.78 s |

El promedio de las cinco ejecuciones fue aproximadamente 3.36 segundos.

La primera ejecución presentó una latencia significativamente mayor, de 10.16 segundos, mientras que las siguientes cuatro ejecuciones estuvieron entre 1.43 y 1.78 segundos.

El promedio de las ejecuciones 2 a 5 fue aproximadamente 1.67 segundos.

Durante la medición de memoria se obtuvo:

| Recurso | Resultado |
|---|---:|
| RAM total | 15 GiB |
| RAM usada | 5.1 GiB |
| RAM libre | 4.6 GiB |
| RAM disponible | 10 GiB |
| Swap total | 3.7 GiB |
| Swap utilizada | 0 B |

No se observó utilización de Swap durante la medición.

---

## 10. Análisis del cuello de botella

Al comparar la prueba de carga del motor ECO con la caracterización del modelo Ollama se observa que el principal cuello de botella del sistema se encuentra en la etapa de inferencia del modelo y no en la API ni en la base de datos.

La prueba de carga sobre el motor ECO alcanzó 10 usuarios virtuales y obtuvo una latencia p95 de 66.28 ms, con una tasa de errores de 0.00 %. Estos resultados muestran que la API puede atender las solicitudes de clasificación mediante el motor ECO con una latencia baja y sin errores durante la prueba.

Por otra parte, las mediciones realizadas directamente sobre el modelo `qwen2.5-coder:1.5b` presentaron tiempos significativamente mayores. La primera ejecución tardó 10.16 segundos y las siguientes cuatro estuvieron entre 1.43 y 1.78 segundos.

Por lo tanto, la mayor parte del tiempo se concentra en la inferencia del modelo. La primera ejecución presenta además un costo elevado asociado al arranque o carga inicial del modelo.

La API y la base de datos no muestran ser el principal cuello de botella según los resultados obtenidos.

---

## 11. Propuestas de mejora

### 11.1 Mantener el modelo cargado en memoria

En equipos con memoria suficiente se puede mantener el modelo cargado para evitar el costo de la primera inferencia.

Esto permitiría reducir la latencia inicial observada durante la caracterización, donde la primera ejecución alcanzó 10.16 segundos frente a tiempos posteriores entre 1.43 y 1.78 segundos.

### 11.2 Utilizar el motor ECO como filtro previo

Se puede utilizar el motor ECO para clasificar primero los mensajes que puedan resolverse mediante reglas.

Solamente los mensajes que no puedan clasificarse de manera confiable serían enviados al modelo Ollama.

Esto reduciría la cantidad de inferencias realizadas y, por tanto, el consumo de recursos y la latencia general del sistema.

### 11.3 Implementar una caché

Se puede agregar una caché para almacenar la clasificación de mensajes repetidos.

Si llega nuevamente un mensaje que ya fue procesado, el sistema podría devolver directamente el resultado almacenado sin ejecutar otra inferencia.

Esto permitiría reducir el tiempo de respuesta y el consumo de recursos en solicitudes repetidas.

### 11.4 Limitar la concurrencia hacia Ollama

Se puede establecer un límite de solicitudes simultáneas hacia Ollama para evitar una sobrecarga del modelo y controlar el consumo de memoria.

Esto permitiría mantener tiempos de respuesta más estables cuando aumente la cantidad de solicitudes concurrentes.

---

## 12. Conclusión

El sistema de clasificación de mensajes de commit cuenta con una arquitectura basada en FastAPI, PostgreSQL, un motor ECO basado en reglas y un motor de inteligencia artificial mediante Ollama.

Durante las pruebas realizadas se verificó el funcionamiento de los endpoints principales, la conexión con PostgreSQL, el control de privilegios del usuario de aplicación, la persistencia de los datos y el procedimiento de respaldo y restauración.

La prueba de respaldo permitió recuperar correctamente los 1308 registros existentes antes de la simulación de pérdida de datos.

Las pruebas de carga realizadas con k6 demostraron un comportamiento satisfactorio del motor ECO, obteniendo un p95 de 66.28 ms y una tasa de errores de 0.00 %, cumpliendo los criterios establecidos.

La caracterización del modelo `qwen2.5-coder:1.5b` permitió identificar que la inferencia constituye el principal cuello de botella del sistema. También se identificó un tiempo mayor durante la primera ejecución debido al costo de carga inicial del modelo.

Como mejoras futuras se propone mantener el modelo cargado en memoria cuando los recursos lo permitan, utilizar el motor ECO como filtro previo, implementar una caché para mensajes repetidos y limitar la concurrencia hacia Ollama.

La solución queda documentada y preparada para las etapas finales de operación, entrega y evaluación del proyecto.
