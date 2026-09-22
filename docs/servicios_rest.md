# Servicios REST propuestos

Los siguientes servicios representan funcionalidades reales del módulo BFA Espacial que posteriormente podrán ser implementadas mediante una API REST con Spring Boot.

| Funcionalidad | Método HTTP | Endpoint | Descripción |
|---|---|---|---|
| Iniciar intento | POST | `/api/intentos` | Crea un nuevo intento BFA para el estudiante |
| Consultar intento | GET | `/api/intentos/{id}` | Recupera el estado del intento |
| Obtener subtest | GET | `/api/intentos/{id}/subtests/{subtest}` | Obtiene el contenido del subtest activo |
| Registrar respuestas | POST | `/api/intentos/{id}/respuestas` | Registra respuestas del estudiante |
| Finalizar subtest | POST | `/api/intentos/{id}/subtests/{subtest}/finalizar` | Cierra el segmento actual |
| Consultar resultado | GET | `/api/resultados/{id}` | Obtiene S1, S2, ST y percentiles |
| Consultar reactivos | GET | `/api/reactivos` | Consulta reactivos del módulo |
| Registrar reactivo | POST | `/api/reactivos` | Registra un reactivo |
| Consultar baremos | GET | `/api/baremos` | Consulta tablas de conversión |
| Actualizar baremo | PUT | `/api/baremos/{id}` | Actualiza un baremo autorizado |


## Servicios REST principales

### 1. Iniciar intento

**Método:** POST  
**Endpoint:** `/api/intentos`

Permite crear un nuevo intento de evaluación BFA Espacial para el estudiante, respetando las reglas del sistema como el intento único por período.

### 2. Registrar respuestas

**Método:** POST  
**Endpoint:** `/api/intentos/{id}/respuestas`

Permite registrar las respuestas realizadas por el estudiante durante los subtests de la evaluación.

### 3. Consultar resultado

**Método:** GET  
**Endpoint:** `/api/resultados/{id}`

Permite consultar el resultado calculado del intento, incluyendo las puntuaciones S1, S2, ST y sus percentiles.


## Flujo representado

Estos tres servicios representan el flujo principal de la evaluación:

**Inicio del intento → Registro de respuestas → Consulta de resultados**

Se utiliza `POST` para crear o registrar información nueva en el servidor y `GET` para consultar información sin modificarla.