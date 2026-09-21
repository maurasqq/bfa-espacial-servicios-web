# TALLER #1 — BFA ESPACIAL · Organización del equipo y commits

**Integrantes:** Maura · Joseph · Alexander · Davide
**Antes de empezar, todos leen [`context.md`](context.md)** (10 min). Ahí están los datos del proyecto, las capas, los endpoints con sus nombres oficiales, el diccionario de términos y las respuestas modelo de la defensa.

## Objetivo

Crear la **propuesta de arquitectura** para transformar el proyecto *BFA Espacial* (Ingeniería de Software I) en una **solución basada en una API REST con Spring Boot**.

**No se implementa nada.** No hay endpoints, controllers ni repositories reales. Todo lo que aparece como endpoint o clase es **propuesto**.

Entregable principal: **diagrama de arquitectura · descripción de capas · servicios REST propuestos · preparación de la exposición y defensa.**

### Cómo se califica (25 pts) y quién lo cubre

| Criterio | Pts | Archivo | Responsable |
|---|---|---|---|
| Diseño de la arquitectura | 7 | `diagramas/arquitectura-bfa.*`, `docs/arquitectura.md` | Davide |
| Definición de capas | 6 | `docs/arquitectura.md` | Davide |
| Comunicación entre componentes | 4 | `docs/arquitectura.md` (flujo) | Joseph |
| Servicios REST propuestos | 3 | `docs/servicios-rest.md` | Alexander |
| Defensa y justificación | 5 | `docs/defensa.md` | Todos, integra Maura |

---

## Estructura del repositorio

Maura crea el repositorio `bfa-espacial-servicios-web` con:

```
bfa-espacial-servicios-web/
├── README.md
├── docs/
│   ├── contexto-bfa.md
│   ├── arquitectura.md
│   ├── servicios-rest.md
│   └── defensa.md
└── diagramas/
    ├── arquitectura-bfa.drawio
    └── arquitectura-bfa.png
```

**No crear** `src/`, `pom.xml`, `controller/`, `service/` ni `repository/`: este taller solo pide la propuesta. (`context.md` y `distribucion.md` pueden quedarse en `docs/equipo/` si el profesor no los quiere en la raíz.)

---

## Flujo de Git

`main` es la rama principal y **nadie trabaja directamente en ella**, salvo Maura en el commit inicial.

**Una rama por tarea**, con el nombre `persona/tema`:

| Rama | Contiene |
|---|---|
| `main` | Commit 1 (Maura) |
| `joseph/contexto-bfa` | Commit 2 |
| `alexander/servicios-rest` | Commits 3 y 6 |
| `davide/arquitectura` | Commits 4 y 7 |
| `joseph/flujo-comunicacion` | Commit 5 (**sale de `main` después de que el PR de Davide se una**) |
| `maura/taller1-integracion` | Commit 8 y 9 |

Comandos de cada persona (cambia los nombres):

```bash
git switch main && git pull                 # partir siempre de main actualizado
git switch -c davide/arquitectura           # crear tu rama
# ... editar archivos ...
git add docs/arquitectura.md diagramas/
git commit -m "Diseñar arquitectura Spring Boot para BFA Espacial"
git push -u origin davide/arquitectura      # y abrir Pull Request hacia main
```

Reglas para que no se pisen entre ustedes:

1. **Cada archivo tiene un dueño** (tabla de abajo). Si necesitas cambiar un archivo ajeno, avisa al dueño o hazlo *después* de que su PR esté unido.
2. El PR lo revisa Maura y se une a `main`. Antes de abrir tu PR: `git pull --rebase origin main`.
3. **Commit 5 depende del 4**: Joseph agrega el flujo a `docs/arquitectura.md`, archivo que crea Davide. Joseph espera a que el PR de Davide esté unido, luego crea su rama desde `main`.
4. **Commit 7 va en la misma rama de Davide** (mismos archivos que el 4) para evitar conflictos.
5. Commits pequeños y mensajes exactamente como se indican abajo, para que el historial muestre quién hizo qué.

| Archivo | Dueño |
|---|---|
| `README.md`, `docs/defensa.md` (integración) | Maura |
| `docs/contexto-bfa.md` | Joseph |
| `docs/servicios-rest.md` | Alexander |
| `docs/arquitectura.md`, `diagramas/*` | Davide (Joseph solo añade la sección de flujo, commit 5) |

---

## COMMIT 1 — MAURA · Crear y preparar el repositorio

Crea `README.md`, `docs/` y `diagramas/`. El README inicial incluye: nombre del proyecto, asignatura, integrantes, objetivo del Taller #1, breve explicación de BFA Espacial y estructura del repositorio.

Resumen sugerido:
> BFA Espacial es un módulo de IQTest encargado de administrar una evaluación psicométrica espacial compuesta por subtests cronometrados. El sistema gestiona intentos, registra respuestas, calcula puntuaciones, convierte resultados a percentiles mediante baremos y permite integrar los resultados con IQTest.

```bash
git add .
git commit -m "Inicializar repositorio del Taller 1 de BFA Espacial"
git push
```
**Listo cuando:** el repo existe con la estructura de arriba y los cuatro integrantes tienen acceso.

---

## COMMIT 2 — JOSEPH · Contexto del sistema anterior

Archivo: `docs/contexto-bfa.md` (rama `joseph/contexto-bfa`). **Resumido**, sin copiar los diagramas del semestre pasado. Fuente: `context.md` §2. Debe incluir:

- **Problema que resuelve:** digitalizar y automatizar la aplicación del test BFA Espacial.
- **Actores:** aspirante/estudiante · evaluador/psicólogo/docente · administrador del módulo · Dashboard IQTest · sistema de autenticación IQTest.
- **Funcionalidades:** iniciar/reanudar intento · realizar evaluación · presentar subtests · registrar respuestas · controlar tiempo · impedir retroceso · calcular puntuaciones · convertir a percentiles · consultar resultados · integrar con IQTest · administrar reactivos · administrar baremos.
- **Subtests, en orden de aplicación:** S1a Figuras Idénticas → S2 Desplazamiento Espacial → S1b Ladrillos-Cubos.
- **Reglas importantes:** intento único por periodo · subtests cronometrados · no retroceso · respuestas correctas ocultas · resultados mediante baremos oficiales · acceso según autorización.

```bash
git add docs/contexto-bfa.md
git commit -m "Documentar contexto y funcionalidades del BFA Espacial"
git push -u origin joseph/contexto-bfa
```
Pull Request hacia `main`. **Listo cuando:** cabe en una o dos páginas y usa los nombres del diccionario (`context.md` §6).

---

## COMMIT 3 — ALEXANDER · Servicios REST propuestos

Archivo: `docs/servicios-rest.md` (rama `alexander/servicios-rest`). Identifica qué funcionalidades del sistema viejo se convierten en servicios REST. Propuesta base (`context.md` §4.1):

| Funcionalidad | Método | Endpoint | Descripción |
|---|---|---|---|
| Iniciar intento | POST | `/api/intentos` | Crea un nuevo intento BFA para el estudiante |
| Consultar intento | GET | `/api/intentos/{id}` | Recupera el estado del intento |
| Obtener subtest | GET | `/api/intentos/{id}/subtests/{subtest}` | Obtiene el contenido del subtest activo |
| Registrar respuestas | POST | `/api/intentos/{id}/respuestas` | Registra respuestas del estudiante |
| Finalizar subtest | POST | `/api/intentos/{id}/subtests/{subtest}/finalizar` | Cierra el segmento actual |
| Consultar resultado | GET | `/api/resultados/{id}` | Obtiene S1, S2, ST y percentiles (`{id}` = id del intento) |
| Consultar reactivos | GET | `/api/reactivos` | Consulta reactivos del módulo |
| Registrar reactivo | POST | `/api/reactivos` | Registra un reactivo |
| Consultar baremos | GET | `/api/baremos` | Consulta tablas de conversión |
| Actualizar baremo | PUT | `/api/baremos/{id}` | Actualiza un baremo autorizado |

**Los tres mínimos, que sí o sí aparecen destacados:**
```
POST /api/intentos
POST /api/intentos/{id}/respuestas
GET  /api/resultados/{id}
```
Explicar que representan el flujo **inicio → realización de la evaluación → resultado**, por qué se usan `POST` (crear/registrar) y `GET` (leer), y que son **endpoints propuestos, no implementados**. Incluir un ejemplo de mensaje JSON por cada uno (`context.md` §4.3).

```bash
git add docs/servicios-rest.md
git commit -m "Definir servicios REST propuestos para BFA Espacial"
git push -u origin alexander/servicios-rest
```
Pull Request hacia `main`. **Listo cuando:** ≥ 3 endpoints con método, ruta y descripción, y los tres mínimos justificados.

---

## COMMIT 4 — DAVIDE · Arquitectura y diagrama

Archivos: `diagramas/arquitectura-bfa.drawio`, `diagramas/arquitectura-bfa.png`, `docs/arquitectura.md` (rama `davide/arquitectura`). Hecho en Draw.io. **Un diagrama de arquitectura, no un UML de clases.**

Estructura del diagrama:

```
┌─────────────────────────────┐
│     CLIENTE WEB IQTEST      │
│ Estudiante · Evaluador ·    │
│ Administrador               │
└──────────────┬──────────────┘
        HTTPS + JSON   ▲  (respuesta JSON)
               ▼       │
┌─────────────────────────────────────┐
│     API REST BFA ESPACIAL           │
│          SPRING BOOT                │
│  Controller   → recibe HTTP, JSON   │
│  Service      → coordina casos de uso│
│  Lógica de negocio → intento único, │
│     tiempo, no retroceso, S1/S2/ST, │
│     percentiles                     │
│  Repository   → acceso a datos      │
│  Infraestructura → JPA/Hibernate,   │
│     config BD, integración IQTest,  │
│     seguridad                       │
└──────────────┬──────────────────────┘
        JPA / JDBC   ▲  (datos)
               ▼     │
      ┌─────────────────┐
      │ BASE DE DATOS   │
      │  BFA ESPACIAL   │
      └─────────────────┘
```
Debe mostrar también las flechas de regreso: `Base de datos → Repository → Service → Controller → JSON → Cliente`. La flecha cliente↔API se rotula **HTTPS + JSON**; la de API↔BD, **JPA / JDBC**.

`docs/arquitectura.md` describe la responsabilidad de cada capa (`context.md` §3.2): Controller · Service · Lógica de negocio · Repository · Infraestructura · Base de datos, con los ejemplos de clases propuestas.

```bash
git add diagramas/ docs/arquitectura.md
git commit -m "Diseñar arquitectura Spring Boot para BFA Espacial"
git push -u origin davide/arquitectura
```
Pull Request hacia `main` (**este PR se une primero que el commit 5**). **Listo cuando:** se cumple el checklist de `context.md` §3.1.

---

## COMMIT 5 — JOSEPH · Flujo de comunicación

Rama nueva `joseph/flujo-comunicacion`, creada desde `main` **después** de unido el PR de Davide. Agrega a `docs/arquitectura.md`:

**Solicitud:** `Cliente Web IQTest → HTTP/HTTPS + JSON → Controller → Service → Lógica de negocio → Repository → Base de datos`
**Respuesta:** `Base de datos → Repository → Service → Controller → Respuesta HTTP + JSON → Cliente Web IQTest`

y el ejemplo paso a paso `POST /api/intentos/25/respuestas` (`context.md` §3.4): `RespuestaController → EvaluacionService → validar intento + tiempo + subtest activo → RespuestaRepository → Base de datos`, con la respuesta de regreso.

```bash
git add docs/arquitectura.md
git commit -m "Documentar flujo de comunicación de la API"
git push -u origin joseph/flujo-comunicacion
```
Pull Request hacia `main`. **Listo cuando:** se ven ida **y** vuelta, y el ejemplo nombra las capas en el mismo orden que el diagrama.

---

## COMMIT 6 — ALEXANDER · Endpoints ↔ reglas de negocio

En `docs/servicios-rest.md` (su rama, ya unida o rebaseada) añade la sección «Reglas relacionadas» (`context.md` §4.4):

```
POST /api/intentos
  - intento único por período · contexto válido de IQTest · versión de formulario
POST /api/intentos/{id}/respuestas
  - intento activo · subtest activo · tiempo disponible · impedir retroceso · respuestas correctas no expuestas
GET /api/resultados/{id}
  - resultado previamente calculado · acceso autorizado · protección de información psicométrica
```
Esto demuestra que los endpoints no son un CRUD cualquiera sino que provienen del sistema anterior.

```bash
git add docs/servicios-rest.md
git commit -m "Relacionar endpoints con reglas de negocio del BFA"
git push
```
Pull Request hacia `main`.

---

## COMMIT 7 — DAVIDE · Diagrama final

Misma rama `davide/arquitectura`. Revisa y exporta `arquitectura-bfa.png` (y PDF si el profesor lo pide). Checklist: ninguna capa falta · se ve HTTP/HTTPS · aparece JSON · existe el cliente · aparece Spring Boot · aparece la base de datos · las flechas tienen sentido · aparecen solicitud **y** respuesta · las responsabilidades son visibles · los nombres coinciden con `context.md` §6.

```bash
git add diagramas/
git commit -m "Finalizar diagrama de arquitectura del Taller 1"
git push
```

---

## COMMIT 8 — MAURA · Integración final

Revisa que **contexto ↔ arquitectura ↔ servicios REST ↔ diagrama** sean consistentes: mismos nombres de capas, mismos endpoints (si Alexander propone `POST /api/intentos`, el diagrama no puede decir otra cosa). Actualiza `README.md` con:

**Arquitectura propuesta:** `Cliente IQTest → Controller → Service → Lógica de negocio → Repository → Base de datos`
**Servicios REST principales:**
```
POST /api/intentos
POST /api/intentos/{id}/respuestas
GET  /api/resultados/{id}
```
```bash
git add .
git commit -m "Integrar propuesta final de arquitectura y servicios REST"
git push
```
Último commit funcional antes de preparar la exposición.

---

## COMMIT 9 — TODO EL EQUIPO · Guía de defensa

Archivo `docs/defensa.md`; cada uno prepara su parte con `context.md` §5. **Las 9 preguntas de la rúbrica ya tienen dueño:**

| # | Pregunta | Responsable |
|---|---|---|
| 1 | ¿Qué proyecto usamos de referencia? | Maura |
| 2 | ¿Qué problema resuelve? | Joseph |
| 3 | ¿Quién es el cliente que consume la API? | Joseph |
| 4 | ¿Cuál es el papel central de la API REST? | Davide |
| 5 | ¿Qué responsabilidad tiene cada capa? | Davide |
| 6 | ¿Cómo se comunica el cliente con la API? | Alexander |
| 7 | ¿Cómo se conecta y accede la API a la base de datos? | Davide |
| 8 | ¿Qué funcionalidades se convierten en servicios REST? | Alexander |
| 9 | ¿Por qué se organizó así la arquitectura? (cierre) | Maura |

Orden de exposición: **Maura** (introducción y diagrama general) → **Joseph** (problema y cliente) → **Davide** (capas, papel de la API, acceso a datos) → **Alexander** (REST, HTTPS, JSON) → **Maura** (justificación y cierre).

Frases clave:
- *Cliente (Joseph):* «El cliente directo de la API será principalmente el frontend web de IQTest. Los usuarios interactúan con ese frontend y este realiza las solicitudes HTTP hacia la API BFA Espacial.»
- *Capas (Davide):* «Separamos las responsabilidades para evitar mezclar la comunicación HTTP, las reglas psicométricas y el acceso a datos.»
- *Cierre (Maura):* «La arquitectura se diseñó para mantener separadas las responsabilidades, facilitar el mantenimiento y permitir que BFA Espacial se integre con IQTest sin que el cliente tenga acceso directo a la base de datos o a la lógica interna.»

Todos se estudian además las «preguntas difíciles» de `context.md` §5: cualquiera puede recibir cualquier pregunta.

```bash
git add docs/defensa.md
git commit -m "Preparar guía de exposición y defensa del Taller 1"
git push
```

---

## Orden final de commits en `main`

1. Maura — Inicializar repositorio del Taller 1 de BFA Espacial
2. Joseph — Documentar contexto y funcionalidades del BFA Espacial
3. Alexander — Definir servicios REST propuestos para BFA Espacial
4. Davide — Diseñar arquitectura Spring Boot para BFA Espacial
5. Joseph — Documentar flujo de comunicación de la API
6. Alexander — Relacionar endpoints con reglas de negocio del BFA
7. Davide — Finalizar diagrama de arquitectura del Taller 1
8. Maura — Integrar propuesta final de arquitectura y servicios REST
9. Maura / equipo — Preparar guía de exposición y defensa del Taller 1

El historial demuestra la participación de los cuatro.

## Repartición resumida

| Persona | Responsabilidad principal |
|---|---|
| Maura | Repo, organización, integración final, README y coordinación |
| Joseph | Contexto BFA, problema, actores y flujo de comunicación |
| Alexander | Servicios REST, endpoints y relación con reglas de negocio |
| Davide | Arquitectura, responsabilidades de capas y diagrama Draw.io |

## Definición final del trabajo

Al terminar debe existir: `README.md` (resumen y arquitectura/servicios principales) · `docs/contexto-bfa.md` · `docs/arquitectura.md` · `docs/servicios-rest.md` · `docs/defensa.md` · `diagramas/arquitectura-bfa.drawio` y `.png`.

**Nada más es necesario para este Taller #1.** No creen controllers vacíos ni repositories falsos para aparentar avance: eso vendrá cuando el profesor pida la implementación real de los servicios.
