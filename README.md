# Acme Daily — Modelo de arquitectura C4 - Johan Suarez

## 1. Lo que entendí del C4 Model

Lo que entendí del C4 Model es que es una forma de diseñar y representar la arquitectura de una aplicación con un enfoque que puede estar más orientado hacia el cliente y al mismo tiempo hacia el equipo de desarrollo, dependiendo del nivel en el que se encuentre el diagrama. La idea es ir del contexto a los contenedores y luego a los componentes, haciendo una especie de zoom en cada nivel para especificar y entrar en más detalle sobre las partes que hacen parte del sistema y cómo se relacionan entre sí.

- **C1 – Contexto:** permite entender qué hace el sistema, quiénes lo utilizan y con qué otros sistemas interactúa, sin entrar en detalles técnicos.
- **C2 – Contenedores:** permite entender cómo se divide el sistema en aplicaciones y almacenes de datos, qué tecnologías utiliza y cómo se comunican. También se pueden mostrar servicios de apoyo, diferenciándolos de los contenedores propiamente dichos.
- **C3 – Componentes:** permite entrar en un contenedor específico para identificar sus módulos internos y las responsabilidades de cada uno.

## 2. Propuesta y justificación de la arquitectura

Para Acme Daily decidí utilizar una arquitectura de monolito, ya que el sistema tiene un conjunto de funcionalidades específicas que pueden desarrollarse dentro de una misma aplicación sin necesidad de separarlas en microservicios. La idea es aprovechar Microsoft Teams como interfaz de usuario mediante Adaptive Cards, de manera que los integrantes puedan diligenciar sus reportes y el Scrum Master pueda gestionar los horarios y recibir alertas sin necesidad de utilizar una página web adicional.

Por otro lado, decidí utilizar algunos servicios de AWS para el alojamiento, la programación de tareas y el almacenamiento de información. Esto permite reducir el trabajo relacionado con la administración de infraestructura y concentrar el desarrollo en las funcionalidades del sistema.

## 3. Diagrama C1 — Contexto

Acme Daily se relaciona con dos tipos de usuarios: los integrantes de los equipos Scrum y el Scrum Master. Los primeros diligencian sus reportes y consultan las actualizaciones del equipo, mientras que el segundo configura el proceso y revisa las alertas. Microsoft Teams es el medio de interacción y Microsoft Entra ID proporciona la identidad corporativa. El bot requiere su registro y configuración del canal de Teams.

## 4. Diagrama C2 — Contenedores

| Elemento | Clasificación C4 | Responsabilidad |
|---|---|---|
| Backend Acme Daily | Contenedor de aplicación | Procesa solicitudes, aplica reglas de negocio y se comunica con Teams. |
| Base de datos | Contenedor de datos | Guarda integrantes, equipos, horarios, reportes y estados de envío. |
| Planificador | Servicio administrado de soporte | Activa periódicamente las revisiones de horarios y reportes pendientes. |
| Microsoft Teams | Sistema externo | Presenta formularios y mensajes y permite publicar en canales. |
| Microsoft Entra ID | Sistema externo | Proporciona identidad y credenciales de integración. |

## 5. Diagrama C3 — Componentes del backend

| Componente | Responsabilidad |
|---|---|
| Adaptador Teams | Recibe actividades, construye Adaptive Cards y envía mensajes privados y publicaciones. |
| Controlador de solicitudes | Coordina las solicitudes de Teams y las activaciones del planificador; devuelve acciones al adaptador para su envío. |
| Autorización | Comprueba credenciales, organización, pertenencia al equipo y permisos. |
| Servicio de reportes | Valida campos, registra reportes y comprueba si los tres últimos reportes consecutivos tienen el mismo identificador de tarea. |
| Servicio de seguimiento | Comprueba horarios y ausencias, genera recordatorios y alertas. |
| Servicio de configuración | Gestiona miembros, horas de envío, plazo y canal de publicación. |
| Repositorio de datos | Centraliza las consultas y escrituras en DynamoDB. |

Todos estos módulos pertenecen al mismo backend; no se despliegan como servicios separados.

## 6. Tecnologías seleccionadas

| Tecnología | Justificación |
|---|---|
| Node.js y TypeScript | Permiten implementar la lógica de negocio y la integración de Teams en un solo proyecto, con soporte de tipado y operaciones asíncronas. |
| Microsoft Teams SDK | Facilita el procesamiento de actividades y el envío de mensajes del bot. |
| Adaptive Cards | Permiten construir formularios directamente en Teams sin desarrollar un frontend independiente. |
| Microsoft Entra ID | Aprovecha la identidad corporativa existente y evita mantener contraseñas de usuarios propias. |
| Amazon ECS Express Mode | Alojamiento administrado de una aplicación en contenedor con endpoint HTTPS; evita configurar manualmente toda la infraestructura. |
| Amazon EventBridge | Automatiza activaciones periódicas. Scheduler envía eventos al bus mediante PutEvents y una regla los dirige a un API Destination HTTPS autenticado del backend. |
| Amazon DynamoDB | Base de datos administrada con capacidad bajo demanda. Las consultas pueden organizarse por equipo, integrante y fecha. |
