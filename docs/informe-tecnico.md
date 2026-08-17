# Informe Técnico – Caracterización del Modelo Local

## Implementación de Soluciones de Inteligencia Artificial
**Semana 1 – AA1: Preparación del entorno y del motor de inferencia**

---

# 1. Objetivo

Caracterizar el comportamiento del modelo de lenguaje ejecutado localmente mediante Ollama, evaluando el entorno de ejecución, el consumo de recursos, el tiempo de respuesta y la calidad de las respuestas obtenidas. Esta información servirá como línea base para comparar el desempeño del modelo durante las pruebas de despliegue de la Semana 4.

---

# 2. Información del entorno

| Característica | Valor |
|----------------|-------|
| Perfil de hardware | A |
| Sistema Operativo | Ubuntu |
| Memoria RAM total | 15 GiB |
| Memoria disponible | 10 GiB |
| Memoria Swap | 3.7 GiB |

---

# 3. Información del modelo

| Dato | Valor |
|------|-------|
| Plataforma | Ollama |
| Modelo | qwen2.5-coder:1.5b |
| Identificador | d7372fd82851 |
| Tamaño en disco | 986 MB |

---

# 4. Pruebas de latencia

Para medir el tiempo de respuesta del modelo se ejecutó cinco veces el mismo comando mediante la API REST de Ollama utilizando `time curl`.

| Ejecución | Tiempo (s) |
|-----------|-----------:|
| 1 | 1.480 |
| 2 | 1.466 |
| 3 | 1.529 |
| 4 | 1.491 |
| 5 | 1.462 |

### Promedio de latencia

**1.486 segundos**

---

# 5. Consumo de memoria durante la inferencia

Durante la ejecución del modelo se monitoreó el uso de memoria mediante el comando:

```bash
free -h
```

Se obtuvo el siguiente comportamiento:

| Métrica | Valor |
|---------|-------|
| Memoria utilizada | 5.5 GiB |
| Memoria libre | 1.4 GiB |
| Memoria en caché | 9.1 GiB |
| Memoria disponible | 9 GiB |

Se observa que el modelo puede ejecutarse correctamente en un equipo con 16 GB de memoria RAM sin generar problemas de disponibilidad de recursos.

---

# 6. Evaluación cualitativa

**Calificación:** 5 / 5

### Observaciones

Durante las pruebas el modelo respondió correctamente tanto desde la consola (`ollama run`) como mediante su API REST (`curl`). Las respuestas fueron coherentes, completas y generadas en un tiempo aproximado de 1.5 segundos.

Al tratarse de un modelo orientado al desarrollo de software, se considera apropiado para las actividades del curso, especialmente para la clasificación de mensajes de commit y el soporte en tareas de programación.

---

# 7. Conclusiones

- Se verificó correctamente la instalación y funcionamiento de Ollama.
- El modelo **qwen2.5-coder:1.5b** quedó instalado y disponible para ser utilizado por aplicaciones externas mediante la API REST.
- La latencia promedio obtenida fue de **1.486 segundos**, lo que representa un tiempo de respuesta adecuado para un entorno de desarrollo local.
- El consumo de memoria observado confirma que el equipo dispone de recursos suficientes para ejecutar el modelo sin afectar significativamente el rendimiento del sistema.
- Esta caracterización servirá como línea base para comparar el comportamiento del modelo durante las pruebas de despliegue y rendimiento que se realizarán en la Semana 4.

## Semana 4 — Caracterización del modelo


## Matriz de pruebas — Semana 4

| ID | Tipo | Qué se verifica | Resultado esperado | Obtenido | Estado |
|---|---|---|---|---|---|
| P-01 | Funcional | GET `/health` responde | Código 200 y estado ok | Código 200 y estado `ok` | Cumple |
| P-02 | Funcional | POST `/clasificar` con motor eco | Código 200 y tipo correcto | Código 200 y clasificación correcta | Cumple |
| P-03 | Funcional | Motor inválido | Código 400 | Código 400 | Cumple |
| P-04 | Acceso | Rol `app_ia` intenta DROP TABLE | Error de permisos | `permission denied for table inferencias` | Cumple |
| P-05 | Conectividad | La API resuelve el host `db` | Devuelve una IP interna | `172.18.0.2` | Cumple |
| P-06 | Disponibilidad | Reinicio del contenedor de BD | La API se recupera sola | `/health` respondió correctamente después del reinicio | Cumple |
| P-07 | Persistencia | `down` y `up` conservan los datos | Los registros siguen existiendo | Los registros anteriores continuaron disponibles | Cumple |
| P-08 | Carga | 10 usuarios sobre el motor ECO | p95 < 800 ms y errores < 5 % | p95 = 66.28 ms; errores = 0.00 % | Cumple |
| P-09 | Caracterización | 10 inferencias con modelo | Promedio, mediana y p95 | Se realizaron 5 de las 10 inferencias requeridas | Pendiente |


## Análisis del cuello de botella

Al comparar P-08 y P-09 se observa que el principal cuello de botella del sistema se encuentra en la etapa de inferencia del modelo y no en la API ni en la base de datos.

En P-08, la prueba de carga sobre el motor ECO alcanzó 10 usuarios virtuales y obtuvo una latencia p95 de 66.28 ms, con una tasa de errores de 0.00 %. Estos resultados muestran que la API puede atender las solicitudes de clasificación mediante el motor ECO con una latencia baja y sin errores durante la prueba.

En P-09, las mediciones realizadas directamente sobre el modelo `qwen2.5-coder:1.5b` presentaron tiempos significativamente mayores. La primera ejecución tardó 10.16 s y las siguientes cuatro estuvieron entre 1.43 s y 1.78 s.

Por lo tanto, la mayor parte del tiempo se concentra en la inferencia del modelo. La primera ejecución presenta además un costo elevado asociado al arranque o carga inicial del modelo. La API y la base de datos no muestran ser el principal cuello de botella según los resultados obtenidos en P-08.

## Propuestas de mejora

### 1. Mantener el modelo cargado en memoria

En equipos con memoria suficiente se puede mantener el modelo cargado para evitar el costo de la primera inferencia. Esto permitiría reducir la latencia inicial observada en P-09, donde la primera ejecución alcanzó 10.16 s frente a tiempos posteriores entre 1.43 s y 1.78 s.

### 2. Utilizar el motor ECO como filtro previo

Se puede utilizar el motor ECO para clasificar primero los mensajes que puedan resolverse mediante reglas. Solamente los mensajes que no puedan clasificarse de manera confiable serían enviados al modelo Ollama. Esto reduciría la cantidad de inferencias y, por tanto, el consumo de recursos y la latencia general del sistema.

### 3. Implementar una caché

Se puede agregar una caché para almacenar la clasificación de mensajes repetidos. Si llega nuevamente un mensaje que ya fue procesado, el sistema podría devolver directamente el resultado almacenado sin ejecutar otra inferencia.

### 4. Limitar la concurrencia hacia el modelo

Se puede establecer un límite de solicitudes simultáneas hacia Ollama para evitar una sobrecarga del modelo y controlar el consumo de memoria. Esto permitiría mantener tiempos de respuesta más estables cuando aumente la cantidad de solicitudes.
