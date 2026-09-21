# BFA Espacial - Servicios Web

Proyecto académico desarrollado para la asignatura **Servicios Web**, como continuación del proyecto BFA Espacial trabajado previamente en la asignatura Ingeniería de Software I.

## Integrantes

* Maura
* Joseph
* Alexander
* Davide

## Descripción del proyecto

**BFA Espacial** es un módulo perteneciente al sistema general **IQTest**, diseñado para administrar una evaluación psicométrica orientada a medir habilidades relacionadas con el razonamiento y la capacidad espacial.

La evaluación está compuesta por distintos subtests cronometrados, entre ellos **Figuras Idénticas**, **Ladrillos-Cubos** y **Desplazamiento Espacial**.

El sistema se encarga de gestionar el proceso completo de evaluación, incluyendo:

* validación e inicio de intentos;
* presentación de los subtests;
* control del tiempo de cada segmento;
* registro de respuestas del estudiante;
* aplicación de reglas como impedir el retroceso a subtests finalizados;
* cálculo de puntuaciones directas;
* cálculo de los valores S1, S2 y ST;
* conversión de puntuaciones a percentiles mediante baremos;
* almacenamiento de resultados;
* consulta y exposición de resultados hacia otros componentes del sistema IQTest.

Durante Ingeniería de Software I el proyecto fue desarrollado principalmente a nivel de análisis, requisitos y arquitectura utilizando Enterprise Architect. En la asignatura Servicios Web se propone transformar dicho diseño en una solución basada en servicios web mediante una API REST desarrollada con Spring Boot.

## Objetivo del Taller #1

Diseñar una propuesta de arquitectura que permita implementar las funcionalidades principales del módulo BFA Espacial mediante una **API REST con Spring Boot**.

La propuesta debe definir la estructura de la API, las responsabilidades de sus diferentes capas, la comunicación con la base de datos y la manera en que un cliente podrá consumir los servicios.

En esta primera etapa el objetivo principal es el diseño de la arquitectura y de los servicios REST que posteriormente serán implementados.

## Arquitectura propuesta

La solución seguirá una arquitectura organizada por capas:

```text
Cliente IQTest
        ↓
Controller
        ↓
Service
        ↓
Lógica de Negocio
        ↓
Repository
        ↓
Base de Datos
```

El cliente se comunicará con la API mediante **HTTP/HTTPS**, utilizando **JSON** como formato principal para el intercambio de información.

Cada capa tendrá una responsabilidad específica:

* **Controller:** recibe solicitudes HTTP y devuelve respuestas al cliente.
* **Service:** coordina los casos de uso y operaciones principales del sistema.
* **Lógica de Negocio:** aplica las reglas específicas del BFA Espacial.
* **Repository:** gestiona el acceso y persistencia de datos.
* **Infraestructura:** contiene elementos técnicos como configuración, conexión a base de datos e integración con otros componentes.
* **Base de Datos:** almacena intentos, respuestas, reactivos, baremos y resultados.

## Servicios REST principales propuestos

Entre los servicios que posteriormente podrán ser implementados se encuentran:

* `POST /api/intentos`
  Iniciar un nuevo intento de evaluación BFA Espacial.

* `POST /api/intentos/{id}/respuestas`
  Registrar las respuestas realizadas por un estudiante durante un intento.

* `GET /api/resultados/{id}`
  Consultar el resultado de una evaluación, incluyendo puntuaciones y percentiles.

Estos endpoints representan el flujo principal del sistema:

**Inicio del intento → Registro de respuestas → Obtención de resultados**

## Estructura del repositorio

```text
bfa-espacial-servicios-web/
│
├── README.md
│
├── docs/
│   ├── contexto-bfa.md
│   ├── arquitectura.md
│   ├── servicios-rest.md
│   └── defensa.md
│
└── diagramas/
    ├── arquitectura-bfa.drawio
    └── arquitectura-bfa.png
```

### `docs/`

Contendrá la documentación relacionada con:

* contexto y funcionalidades del BFA Espacial;
* arquitectura propuesta;
* servicios REST;
* flujo de comunicación;
* preparación de la exposición y defensa.

### `diagramas/`

Contendrá el diagrama editable de arquitectura y su versión exportada para presentación.

## Alcance actual

En el Taller #1 no se requiere implementar completamente la API ni desarrollar todos los endpoints.

El objetivo actual es definir una arquitectura clara y coherente que sirva como base para las siguientes etapas de desarrollo del proyecto utilizando Spring Boot.
