# Contexto del Taller #1 — BFA Espacial → API REST con Spring Boot

**Para:** Maura, Joseph, Alexander y Davide · **Léelo completo antes de empezar tu commit (10 min).**
**Recuerden:** el taller es una **propuesta**. No se implementa ningún endpoint, controller ni repository. Todo lo que aquí aparece como endpoint o clase es **propuesto**, nunca «ya implementado».

| Si eres… | Lee primero | Y luego usa |
|---|---|---|
| Maura (repo, integración, cierre) | §1, §3.1, §6, §7 | §5 para armar `defensa.md` |
| Joseph (contexto, problema, flujo) | §2, §3.3–3.4 | §5 (preguntas 1-3, 6) |
| Alexander (servicios REST) | §4 | §2.6 (reglas) y §5 (preguntas 4, 6, 8) |
| Davide (arquitectura y diagrama) | §3, §6 | §5 (preguntas 5, 7, 9) |

---

## 1. Qué pide el taller y cómo se califica

**Objetivo:** proponer la arquitectura para transformar el proyecto *BFA Espacial* (analizado y diseñado en Ingeniería de Software I) en una **solución basada en servicios web**: una **API REST con Spring Boot** que un cliente web consume por HTTP/JSON y que es la única que habla con la base de datos.

**Entregables:** (1) diagrama de arquitectura, (2) descripción de las capas, (3) propuesta de servicios REST, (4) preparación de la exposición y defensa.

| Criterio de la rúbrica | Pts | Qué debe verse en nuestro entregable |
|---|---|---|
| Diseño de la arquitectura | 7 | Diagrama con: cliente, API REST Spring Boot, Controller, Service, lógica de negocio, Repository, infraestructura, base de datos, y la comunicación cliente↔API (HTTP/HTTPS + JSON) y API↔BD |
| Definición de capas | 6 | Cada capa con su responsabilidad, en orden `Cliente → Controller → Service → Repository → Base de datos` |
| Comunicación entre componentes | 4 | Flujo completo de **solicitud** y de **respuesta** (ida y vuelta) |
| Servicios REST propuestos | 3 | ≥ 3 funcionalidades **reales** del proyecto con método, endpoint y descripción |
| Defensa y justificación | 5 | Respuestas claras y coherentes a las 9 preguntas (§5) |
| **Total** | **25** | |

---

## 2. El proyecto de referencia: IQTest · módulo BFA Espacial

### 2.1 Qué es
**IQTest** es una plataforma web de evaluación psicométrica para la admisión de la **Universidad Americana (UAM, Nicaragua)**. Está formada por varios tests que actúan como **módulos**. Nuestro proyecto es **un solo módulo**: **BFA Espacial**, la parte espacial de la **Batería Factorial de Aptitudes (BFA)**.
Cliente del proyecto: MSc. Álvaro Muñoz, Coordinación de Psicología, Facultad de Ciencias Médicas.

> Resumen para el README: *BFA Espacial es un módulo de IQTest encargado de administrar una evaluación psicométrica espacial compuesta por subtests cronometrados. El sistema gestiona intentos, registra respuestas, calcula puntuaciones, convierte resultados a percentiles mediante baremos y permite integrar los resultados con IQTest.*

### 2.2 Problema que resuelve
La prueba se hacía en papel. El módulo **digitaliza y automatiza** su aplicación: controla el tiempo de cada subtest de forma confiable, garantiza un solo intento por aspirante y periodo, califica automáticamente, convierte a percentiles con las normas oficiales y entrega resultados en segundos al evaluador y al Dashboard de IQTest.

### 2.3 Actores

| Actor | Qué hace con el sistema |
|---|---|
| **Aspirante / Estudiante** | Realiza la evaluación. Nunca ve resultados ni respuestas correctas |
| **Evaluador / Psicólogo / Docente** | Consulta resultados y revisa respuestas (con la clave) |
| **Administrador del módulo** | Mantiene reactivos, versiones de formulario y baremos |
| **Dashboard IQTest** | Sistema externo que consulta los resultados del módulo |
| **Sistema de autenticación IQTest** | Sistema externo que identifica al aspirante (su CIF). El módulo **no** hace login propio |

### 2.4 Funcionalidades principales
Iniciar/reanudar intento · realizar la evaluación · presentar subtests · registrar respuestas · controlar el tiempo · impedir retroceso · calcular puntuaciones · convertir a percentiles · consultar resultados · integrar resultados con IQTest · administrar reactivos · administrar baremos.

### 2.5 Subtests (se aplican **en este orden**, sin volver atrás)

| Orden | Código | Nombre | Ejercicios | Tiempo |
|---|---|---|---|---|
| 1 | **S1a** | Figuras Idénticas | 27 | 3 min |
| 2 | **S2** | Desplazamiento Espacial | 34 | 5 min |
| 3 | **S1b** | Ladrillos-Cubos | 49 | 3 min 30 s |

Un **reactivo** es un ejercicio (ítem) del test; cada uno tiene opciones de respuesta y una es la correcta.

### 2.6 Reglas de negocio (van en la capa «Lógica de negocio»)

| Regla | Enunciado corto |
|---|---|
| RN-BFA-01 | **Intento único** por aspirante (CIF) y periodo académico |
| RN-BFA-02 | Hay 3 versiones rotativas del formulario por año; la versión se asigna al iniciar el intento |
| RN-BFA-03 | Subtests **secuenciales** S1a → S2 → S1b; **sin retroceso** |
| RN-BFA-04 | Al agotarse el tiempo, el subtest se **cierra automáticamente** (tolerancia ≤ 1 s) |
| RN-BFA-05 | Un subtest cerrado **no admite** cambios de respuesta |
| RN-BFA-06 | Resultados (puntajes y percentiles) en **menos de 3 s** tras cerrar el último subtest |
| RN-BFA-07 | Soportar **≥ 300 sesiones simultáneas** |
| RN-BFA-08 | Las **respuestas correctas nunca** se exponen al aspirante |
| RN-BFA-09 | Si se cae la conexión, el **reloj sigue en el servidor** y las respuestas se sincronizan al volver |
| RN-BFA-10 | Cálculo: `S1 = S1a + S1b`, `ST = S1 + S2`, y percentiles por tabla de baremo |

### 2.7 Cómo se calcula el resultado (la «matriz de puntuación»)
1. Se cuentan los **aciertos** de cada subtest → puntuaciones directas (PD): `S1a`, `S1b`, `S2`.
2. Se suman: `S1 = S1a + S1b` y `ST = S1 + S2` (Total Espacial).
3. Cada PD (de S1, S2 y ST) se convierte a **percentil** consultando el **baremo**: una tabla *(factor, puntuación directa) → percentil*, tomada de las **Normas Nacionales de la BFA, Nicaragua 1992**.

Ejemplo ilustrativo (valores de la tabla real): `S1a=16, S1b=5 → S1=21 → percentil 80`; `S2=18 → percentil 75`; `ST=39 → percentil 80`.
Si una PD cae en un hueco de la tabla, se usa el percentil de la PD inmediata inferior y se deja constancia en la auditoría.

### 2.8 Datos que guarda el módulo (nivel conceptual)
**Intentos** (uno por CIF y periodo) · **ejecuciones de subtest** (con hora de inicio y cierre) · **respuestas** · **reactivos** y sus **opciones** · **versiones de formulario** · **baremos** · **resultados** (S1a, S1b, S1, S2, ST y percentiles) · **auditoría** (eventos del intento).

### 2.9 Fuera del alcance del módulo
Login y generación del CIF (IQTest) · otros módulos de la batería (Vocabulario, Razonamiento, Numérico…) · compuestos EIC/EIG/AE (los calcula el Dashboard) · agenda de entrevistas · decisión final de admisión.

---

## 3. Arquitectura propuesta

### 3.1 Elementos obligatorios del diagrama (checklist de la rúbrica)
- [ ] **Cliente** (Cliente Web IQTest: Estudiante, Evaluador, Administrador)
- [ ] **API REST BFA Espacial — Spring Boot**
- [ ] Capa **Controller** (presentación)
- [ ] Capa **Service** (aplicación)
- [ ] Capa **Lógica de negocio**
- [ ] Capa **Repository** (acceso a datos)
- [ ] Capa **Infraestructura**
- [ ] **Base de datos** BFA Espacial
- [ ] Flecha cliente ↔ API rotulada **HTTP/HTTPS + JSON**
- [ ] Flecha API ↔ BD rotulada **JPA / JDBC**
- [ ] Flechas de **ida** (solicitud) **y de regreso** (respuesta)
- [ ] Responsabilidad de cada capa visible dentro de su caja

Regla: es un diagrama de **arquitectura**, no un UML de clases. Nada de 40 clases.

### 3.2 Las capas y su responsabilidad

Flujo: **`Cliente → Controller → Service → Lógica de negocio → Repository → Base de datos`**. Cada capa solo habla con la de al lado.

| Capa | Responsabilidad | Ejemplos (propuestos) | Regla BFA que la toca |
|---|---|---|---|
| **Cliente Web IQTest** | Interfaz de estudiante, evaluador y administrador. Hace las solicitudes HTTP; **no** accede a la base de datos ni a la lógica | Pantalla del subtest, panel de resultados | Muestra el reloj (solo visual) |
| **Controller** (presentación) | Recibe la solicitud HTTP, valida el formato del JSON, llama al Service y devuelve la respuesta JSON con su código HTTP | `IntentoController`, `RespuestaController`, `ResultadoController` | RN-08: nunca serializa la respuesta correcta al aspirante |
| **Service** (aplicación) | Coordina cada caso de uso: llama a la lógica de negocio y a los repositorios, define la transacción | `IntentoService`, `EvaluacionService`, `CalificacionService`, `ResultadoService` | Orquesta «registrar respuesta» y «calificar intento» |
| **Lógica de negocio** | Las **reglas del BFA**, independientes de HTTP y de la base de datos | intento único · control de tiempo · no retroceso · cálculo S1/S2/ST · conversión a percentiles | RN-01, 03, 04, 05, 06, 10 |
| **Repository** (acceso a datos) | Lee y guarda datos; oculta el SQL al resto | `IntentoRepository`, `RespuestaRepository`, `ReactivoRepository`, `BaremoRepository`, `ResultadoRepository` | — |
| **Infraestructura** | Servicios técnicos: Spring Data JPA, Hibernate, configuración de la BD, integración con IQTest, seguridad | Config. de conexión, cliente de sesión IQTest, Spring Security | Identidad (CIF) viene de IQTest |
| **Base de datos** | Persistencia: intentos, respuestas, reactivos, formularios, baremos, resultados, auditoría | PostgreSQL | UNIQUE(CIF, periodo) refuerza RN-01 |

**Por qué se separa así (respuesta corta):** para no mezclar la comunicación HTTP, las reglas psicométricas y el acceso a datos. Así cada parte se cambia, se prueba y se explica por separado.

### 3.3 Comunicación
- **Cliente ↔ API:** HTTP sobre TLS (**HTTPS**), cuerpo en **JSON**. La API expone recursos con URLs y verbos HTTP (REST). El cliente jamás toca la base de datos.
- **API ↔ Base de datos:** Repository → **Spring Data JPA / Hibernate** → **JDBC** → base de datos. Solo la API accede a la BD.
- **Con IQTest:** el aspirante llega ya autenticado (CIF); el Dashboard consulta resultados por un endpoint protegido con token de servicio.

### 3.4 Flujo de solicitud y respuesta

**Solicitud:** `Cliente Web IQTest → HTTP/HTTPS + JSON → Controller → Service → Lógica de negocio → Repository → Base de datos`
**Respuesta:** `Base de datos → Repository → Service → Controller → Respuesta HTTP + JSON → Cliente Web IQTest`

Ejemplo `POST /api/intentos/25/respuestas` (el aspirante marca una opción):
1. El cliente envía la respuesta en JSON por HTTPS.
2. `RespuestaController` recibe y valida el formato.
3. `EvaluacionService` coordina el caso de uso.
4. Lógica de negocio: ¿el intento está activo? ¿el subtest es el vigente? ¿queda tiempo? (RN-03, 04, 05).
5. `RespuestaRepository` guarda/actualiza la respuesta.
6. La BD confirma; la confirmación vuelve por Repository → Service → Controller.
7. El controller responde `201`/`200` en JSON (o `409` si el subtest ya cerró). El cliente actualiza la pantalla.

### 3.5 Tecnologías propuestas
Java 21 · Spring Boot 3 (Spring Web MVC para REST) · Spring Data JPA + Hibernate · Spring Security · PostgreSQL · JSON (Jackson) · Maven.

---

## 4. Servicios REST propuestos

> Son **propuestas**, no endpoints implementados. Usen exactamente estos nombres en todos los documentos y en el diagrama.

### 4.1 Tabla base

| Funcionalidad | Método | Endpoint | Descripción |
|---|---|---|---|
| Iniciar intento | `POST` | `/api/intentos` | Crea el intento BFA del estudiante (o devuelve el existente) |
| Consultar intento | `GET` | `/api/intentos/{id}` | Estado del intento |
| Obtener subtest | `GET` | `/api/intentos/{id}/subtests/{subtest}` | Contenido del subtest activo (sin la clave) |
| Registrar respuestas | `POST` | `/api/intentos/{id}/respuestas` | Registra respuestas del estudiante |
| Finalizar subtest | `POST` | `/api/intentos/{id}/subtests/{subtest}/finalizar` | Cierra el subtest actual |
| Consultar resultado | `GET` | `/api/resultados/{id}` | S1, S2, ST y percentiles (`{id}` = id del intento) |
| Consultar reactivos | `GET` | `/api/reactivos` | Reactivos del módulo (administrador) |
| Registrar reactivo | `POST` | `/api/reactivos` | Crea un reactivo |
| Consultar baremos | `GET` | `/api/baremos` | Tablas de conversión |
| Actualizar baremo | `PUT` | `/api/baremos/{id}` | Actualiza un baremo (autorizado) |

`{subtest}` toma los valores `S1A`, `S2`, `S1B`.

### 4.2 Los tres servicios mínimos y por qué forman el flujo
1. **`POST /api/intentos`** → **inicio**: crea el intento.
2. **`POST /api/intentos/{id}/respuestas`** → **realización de la evaluación**: cada respuesta se registra.
3. **`GET /api/resultados/{id}`** → **resultado**: se consulta lo calculado.

Juntos cubren el ciclo completo: `inicio → realización de la evaluación → resultado`. Se usa **`POST`** cuando el cliente **crea o registra** algo nuevo en el servidor (un intento, una respuesta) y **`GET`** cuando solo **lee** sin modificar nada (un resultado). Todos intercambian **JSON** por **HTTPS**.

### 4.3 Ejemplos de mensajes (ilustrativos)

`POST /api/intentos` — sin cuerpo; el CIF viene del contexto de IQTest, **no** del JSON.
```json
201 Created
{ "id": 25, "estado": "ACTIVO", "periodo": "2026-I", "versionFormulario": 2, "subtestActual": "S1A" }
```

`POST /api/intentos/25/respuestas`
```json
{ "subtest": "S1A", "reactivoId": 12, "opcionId": 48 }
→ 201 Created   { "registrada": true, "tiempoRestanteSeg": 141 }
→ 409 Conflict  { "error": "El subtest ya está cerrado" }
```

`GET /api/resultados/25` (valores de ejemplo)
```json
200 OK
{ "intentoId": 25, "pdS1a": 16, "pdS1b": 5, "pdS1": 21, "pdS2": 18, "pdSt": 39,
  "percS1": 80, "percS2": 75, "percSt": 80 }
```

### 4.4 Reglas del BFA relacionadas con cada endpoint

| Endpoint | Reglas relacionadas |
|---|---|
| `POST /api/intentos` | intento único por periodo (RN-01) · contexto válido de IQTest · asignación de versión de formulario (RN-02) |
| `POST /api/intentos/{id}/respuestas` | intento activo · subtest activo (RN-03) · tiempo disponible (RN-04) · impedir retroceso/modificar subtest cerrado (RN-05) · respuestas correctas no expuestas (RN-08) |
| `GET /api/resultados/{id}` | resultado ya calculado (RN-06, 10) · acceso autorizado (solo evaluador) · protección de información psicométrica |

### 4.5 Convenciones
URLs con sustantivos en plural; el verbo lo da el método HTTP · códigos: `200` OK, `201` creado, `400` dato inválido, `401` sin sesión, `403` sin permiso, `404` no existe, `409` conflicto de negocio (subtest cerrado, intento duplicado) · JSON UTF-8.

---

## 5. Defensa: las 9 preguntas de la rúbrica y su respuesta modelo

| # | Pregunta | Respuesta modelo | Responsable |
|---|---|---|---|
| 1 | ¿Qué proyecto usamos de referencia? | BFA Espacial, un módulo de IQTest para aplicar pruebas psicométricas espaciales; lo analizamos y diseñamos en Ingeniería de Software I y ahora proponemos convertirlo en una API REST con Spring Boot | Maura |
| 2 | ¿Qué problema resuelve? | Digitaliza y automatiza la prueba BFA Espacial: tres subtests con tiempo controlado, intento único, calificación automática y conversión a percentiles con baremos oficiales | Joseph |
| 3 | ¿Quién es el cliente que consume la API? | Principalmente el frontend web de IQTest. Los usuarios (aspirante, evaluador, administrador) interactúan con ese frontend y este hace las solicitudes HTTP a la API. También la consume el Dashboard de IQTest | Joseph |
| 4 | ¿Cuál es el papel central de la API REST? | Es la única puerta de entrada al módulo: expone las funcionalidades como servicios HTTP/JSON, aplica las reglas del BFA y protege la base de datos y la lógica interna. Separa el cliente del servidor | Davide |
| 5 | ¿Qué responsabilidad tiene cada capa? | Ver §3.2. Controller: HTTP/JSON · Service: casos de uso · Lógica de negocio: reglas psicométricas · Repository: datos · Infraestructura: JPA, BD, IQTest, seguridad. Separamos para no mezclar HTTP, reglas y datos | Davide |
| 6 | ¿Cómo se comunica el cliente con la API? | Con solicitudes HTTP sobre TLS (HTTPS) a los endpoints REST; el cuerpo de solicitud y respuesta va en JSON; el servidor responde con códigos de estado | Alexander |
| 7 | ¿Cómo se conecta la API a la base de datos y accede a ella? | Solo el Repository accede a los datos, mediante Spring Data JPA/Hibernate sobre JDBC. El cliente nunca ve la BD; la API traduce objetos a filas y viceversa | Davide |
| 8 | ¿Qué funcionalidades se convierten en servicios REST? | Iniciar intento, registrar respuestas, consultar resultado (mínimos) y además consultar intento, obtener/finalizar subtest, y administrar reactivos y baremos — todas salen de los casos de uso del sistema anterior (§4) | Alexander |
| 9 | ¿Por qué se organizó así la arquitectura? | Para mantener separadas las responsabilidades, facilitar el mantenimiento y permitir que BFA Espacial se integre con IQTest sin que el cliente tenga acceso directo a la base de datos ni a la lógica interna. Además el reloj y las reglas viven en el servidor porque el cliente se puede manipular | Maura |

### Preguntas difíciles que puede hacer el profesor
- **¿Por qué REST y no acceso directo a la BD desde el cliente?** Seguridad (credenciales y datos psicométricos sensibles), reglas en un solo lugar y un contrato estable para varios clientes.
- **¿Por qué JSON?** Formato ligero, legible, soportado por cualquier lenguaje y por Spring de forma nativa.
- **¿Para qué HTTPS?** Cifra el tráfico: protege identidad del aspirante, respuestas y resultados.
- **¿Por qué el reloj está en el servidor?** Un cronómetro en el navegador se puede alterar; el servidor guarda la hora de inicio y decide cuándo cerrar (RN-04). Si se cae la red, el tiempo sigue corriendo (RN-09).
- **¿Cómo evitan que el aspirante vea la respuesta correcta?** Las respuestas que se envían al cliente solo llevan el id y la letra de cada opción; la clave nunca sale del servidor (RN-08).
- **¿Qué pasa si se cae la conexión?** Las respuestas quedan guardadas en el cliente y se sincronizan al volver; el reloj del servidor no se detuvo.
- **¿Dónde está la lógica de negocio, si ya hay Service?** Service coordina casos de uso; la lógica de negocio contiene las reglas del BFA. Se distinguen para poder cambiar una prueba o una regla sin tocar los casos de uso.
- **¿Qué es el baremo?** La tabla de las normas nacionales que convierte una puntuación directa en percentil.
- **¿Cómo se integra con IQTest?** IQTest identifica al aspirante y el módulo recibe su CIF; el Dashboard consulta los resultados por un endpoint con token de servicio.
- **¿Ya está implementado?** No: es una propuesta de arquitectura y de servicios; la implementación viene en la siguiente etapa del curso.

---

## 6. Diccionario de términos (para que todos usemos lo mismo)

| Concepto | Nombre oficial en nuestros documentos |
|---|---|
| Cliente | **Cliente Web IQTest** |
| API | **API REST BFA Espacial (Spring Boot)** |
| Capas | **Controller · Service · Lógica de Negocio · Repository · Infraestructura** |
| Base de datos | **Base de datos BFA Espacial** |
| Protocolo / formato | **HTTPS + JSON** (cliente↔API) · **JPA / JDBC** (API↔BD) |
| Subtests | **S1a Figuras Idénticas · S2 Desplazamiento Espacial · S1b Ladrillos-Cubos** (en ese orden) |
| Compuestos | **S1 = S1a + S1b · ST = S1 + S2** |
| Recurso principal | **intento** (una evaluación completa de un aspirante en un periodo) |
| Estados del intento | `ACTIVO`, `COMPLETADO` |
| Estados del subtest | `PENDIENTE`, `EN_CURSO`, `COMPLETADO`, `CERRADO_POR_TIEMPO` |
| Endpoints | Solo los de §4.1; si alguien necesita otro, se agrega **en todos los documentos a la vez** |

## 7. Errores comunes a evitar
1. Decir que los endpoints «ya existen» o «ya están hechos». Son **propuestos**.
2. Dibujar un diagrama de clases UML. Se pide arquitectura.
3. Olvidar la **flecha de regreso** (respuesta) o los rótulos **HTTPS**, **JSON** y **JPA/JDBC**.
4. Poner en el diagrama endpoints o nombres distintos a los de §4/§6.
5. Confundir **Service** (coordina casos de uso) con **Lógica de negocio** (reglas del BFA).
6. Escribir que el cliente accede a la base de datos.
7. Crear carpetas `src/`, controllers o repositories vacíos «para aparentar avance».
8. Copiar todo el análisis del semestre pasado: el contexto debe ser **corto**.

## 8. Glosario
**API REST**: interfaz para usar el sistema mediante URLs y verbos HTTP con datos en JSON. · **Endpoint**: URL + método de un servicio. · **CIF**: código de identificación del aspirante en IQTest. · **Periodo académico**: periodo de admisión (ej. «2026-I»). · **Intento**: evaluación completa de un aspirante. · **Reactivo**: ejercicio del test. · **PD**: puntuación directa (aciertos). · **Percentil**: posición relativa según la norma. · **Baremo**: tabla que convierte PD en percentil. · **SPA / Frontend**: aplicación web que ve el usuario. · **JPA/Hibernate**: tecnología que traduce objetos Java a tablas. · **JDBC**: conexión Java ↔ base de datos.
