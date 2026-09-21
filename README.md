# BFA Espacial - Servicios Web

Proyecto académico de la asignatura Servicios Web.

## Integrantes

- Maura
- Joseph
- Alexander
- Davide

## Objetivo

Diseñar una propuesta de arquitectura para transformar el módulo BFA Espacial en una API REST utilizando Spring Boot.

## Flujo propuesto

Cliente IQTest → Controller → Service → Lógica de Negocio → Repository → Base de Datos

## Servicios REST principales

- POST /api/intentos
- POST /api/intentos/{id}/respuestas
- GET /api/resultados/{id}
