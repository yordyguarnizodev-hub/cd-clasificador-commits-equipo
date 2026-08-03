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
