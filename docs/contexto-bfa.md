# Contexto del sistema BFA Espacial

## ¿Qué es BFA Espacial?

BFA Espacial es un módulo del sistema IQTest orientado a la aplicación digital de una evaluación psicométrica espacial. Su objetivo es administrar de forma controlada los subtests espaciales, registrar las respuestas del aspirante y generar los resultados correspondientes.

El módulo forma parte de una plataforma mayor, IQTest, por lo que no funciona completamente de manera aislada. IQTest proporciona el contexto general del usuario y la autenticación, mientras que BFA Espacial se encarga de la lógica propia de la evaluación.

## Problema que resuelve

El sistema busca digitalizar y automatizar la aplicación del test BFA Espacial.

Entre las principales necesidades que resuelve se encuentran:

- controlar el tiempo de cada subtest;
- registrar las respuestas del aspirante;
- impedir que el usuario regrese a subtests que ya fueron cerrados;
- validar que exista un único intento por aspirante y período;
- calcular las puntuaciones obtenidas;
- convertir las puntuaciones a percentiles utilizando baremos;
- almacenar los resultados;
- permitir que usuarios autorizados consulten los resultados;
- integrar los resultados con el sistema general IQTest.

## Actores principales

### Aspirante / Estudiante

Es el usuario que realiza la evaluación BFA Espacial. Puede iniciar o reanudar un intento, realizar los subtests y registrar sus respuestas.

El aspirante no debe tener acceso a las respuestas correctas ni a información interna utilizada para la calificación.

### Evaluador / Psicólogo / Docente

Es un usuario autorizado que puede consultar y revisar los resultados obtenidos por los aspirantes.

### Administrador del módulo

Es responsable de administrar elementos del sistema como los reactivos, versiones de formulario y baremos utilizados durante la evaluación.

### Dashboard IQTest

Es un componente externo del sistema IQTest que consume los resultados producidos por BFA Espacial.

### Sistema de autenticación IQTest

Es el sistema encargado de identificar al aspirante mediante su contexto de acceso. El módulo BFA Espacial no implementa un sistema de autenticación independiente.

## Funcionalidades principales

Las funcionalidades más importantes del módulo son:

- iniciar o reanudar un intento;
- realizar la evaluación BFA Espacial;
- presentar los subtests;
- registrar respuestas;
- controlar el tiempo de cada segmento;
- impedir retroceso entre subtests cerrados;
- recuperar un intento activo;
- calcular puntuaciones;
- convertir puntuaciones a percentiles;
- consultar resultados;
- integrar resultados con IQTest;
- administrar reactivos;
- administrar versiones de formulario;
- administrar baremos;
- mantener trazabilidad y auditoría del intento.

## Subtests

La evaluación BFA Espacial está compuesta por tres subtests principales:

1. **S1a - Figuras Idénticas**
2. **S2 - Desplazamiento Espacial**
3. **S1b - Ladrillos-Cubos**

Los subtests se presentan de forma secuencial y cada uno posee un tiempo determinado para su realización.

Una vez que un subtest finaliza, el aspirante no puede regresar a modificar las respuestas de ese segmento.

## Reglas de negocio principales

Entre las reglas más importantes del sistema se encuentran:

- cada aspirante puede realizar un único intento por período;
- los subtests son cronometrados;
- el sistema debe controlar el cierre de cada segmento;
- no se permite retroceder a un subtest finalizado;
- las respuestas correctas no deben exponerse al aspirante;
- las respuestas registradas deben conservarse;
- las puntuaciones se calculan automáticamente;
- los resultados se convierten a percentiles mediante baremos;
- el acceso a resultados depende de la autorización del usuario;
- los resultados deben quedar disponibles para su integración con IQTest.

## Resultado de la evaluación

Una vez finalizados los subtests, el sistema calcula las puntuaciones correspondientes a la evaluación.

Se utilizan las puntuaciones de los diferentes subtests para obtener valores como:

- S1;
- S2;
- ST.

Posteriormente estas puntuaciones se convierten a percentiles utilizando los baremos definidos para la prueba.

El resultado final se almacena y puede ser consultado por usuarios o componentes autorizados del sistema IQTest.