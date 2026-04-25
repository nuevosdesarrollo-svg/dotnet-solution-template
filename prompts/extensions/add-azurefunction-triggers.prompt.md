Actúa como un Arquitecto de Software .NET senior con experiencia en arquitecturas serverless y Azure Functions.

OBJETIVO:
Agregar uno o más triggers a un proyecto Azure Functions existente, siguiendo el SDK de funciones aisladas (.NET Isolated Worker Model), las convenciones del proyecto y las reglas arquitectónicas ya establecidas.

PREREQUISITO:
- El proyecto debe tener el inicializador Azure Functions creado (ver prompt `dotnet-base-project`).
- Este prompt no crea el proyecto Azure Functions; agrega triggers a uno existente.

COMPORTAMIENTO:
- Preguntar qué trigger o triggers se desean agregar antes de continuar.
- Presentar la tabla de opciones y explicar el caso de uso de cada tipo.
- No asumir el tipo de trigger sin confirmación del usuario.
- Si el usuario necesita más de un trigger, aplicarlos de uno en uno, verificando compilación tras cada uno.

MODELO DE EJECUCIÓN (OBLIGATORIO):
- Usar exclusivamente el modelo aislado: Microsoft.Azure.Functions.Worker (Isolated Worker Model).
- No usar el modelo in-process (Microsoft.Azure.WebJobs.*) en proyectos nuevos.
- Justificación: el modelo in-process está deprecated desde .NET 8; el modelo aislado es el estándar actual y el único con soporte a largo plazo.

TABLA DE TRIGGERS DISPONIBLES:

| Trigger          | Cuándo usarlo                                                                 | Paquete principal                                      |
|------------------|-------------------------------------------------------------------------------|--------------------------------------------------------|
| HTTP             | Exponer endpoints REST/webhooks sin infraestructura API completa              | Microsoft.Azure.Functions.Worker (incluido en SDK)     |
| Timer            | Ejecutar tareas programadas por expresión cron sin un Worker Service dedicado | Microsoft.Azure.Functions.Worker (incluido en SDK)     |
| Service Bus      | Procesar mensajes de Queue o Topic/Subscription de Azure Service Bus          | Microsoft.Azure.Functions.Worker.Extensions.ServiceBus |
| Queue Storage    | Procesar mensajes de Azure Queue Storage (cargas ligeras, sin sesiones)       | Microsoft.Azure.Functions.Worker.Extensions.Storage    |
| Blob Storage     | Reaccionar a creación/modificación de blobs en Azure Storage                  | Microsoft.Azure.Functions.Worker.Extensions.Storage    |
| Event Grid       | Reaccionar a eventos de la plataforma Azure o eventos personalizados           | Microsoft.Azure.Functions.Worker.Extensions.EventGrid  |

REGLAS DE ELECCIÓN:
- HTTP Trigger: cuando se necesita un endpoint serverless sin levantar Web API completa; no usar para procesamiento asincrónico.
- Timer Trigger: cuando se necesita cron sin historial ni dashboard; para cron + historial usar Hangfire (ver `add-scheduled-tasks`).
- Service Bus Trigger: cuando el sistema ya usa Azure Service Bus o se requieren sesiones, dead-letter nativo y reintentos configurables.
- Queue Storage Trigger: cuando se necesita desacoplamiento ligero sin las garantías de Service Bus (sin sesiones, sin dead-letter avanzado).
- Blob Storage Trigger: cuando el procesamiento debe dispararse ante la llegada de un archivo; no para procesamiento en tiempo real estricto.
- Event Grid Trigger: cuando se reacciona a eventos de la plataforma Azure (creación de recursos, cambios de estado) o a eventos de dominio publicados via Event Grid.
- No mezclar triggers de naturaleza similar sin justificación (ej. Service Bus + Queue Storage para el mismo propósito).

ESTRUCTURA POR TRIGGER:

HTTP Trigger:
- Clase Function en carpeta Functions/ del proyecto Azure Functions.
- Recibe HttpRequestData; retorna HttpResponseData.
- Validación de modelo antes de delegar a Application.
- Responde con ProblemDetails en caso de error.

Timer Trigger:
- Clase Function con parámetro TimerInfo.
- Expresión cron configurable desde appsettings bajo "ScheduledJobs:{NombreJob}:CronExpression".
- No contiene lógica de negocio; delega a un caso de uso de Application.
- Log al inicio y fin de cada ejecución con CorrelationId único por ejecución.

Service Bus Trigger:
- Clase Function con parámetro ServiceBusReceivedMessage.
- Preguntar si se usará Queue o Topic/Subscription.
- Para Queue: configurar "Messaging:ServiceBus:QueueName".
- Para Topic/Subscription: configurar "Messaging:ServiceBus:TopicName" y "Messaging:ServiceBus:SubscriptionName".
- Deserializar el cuerpo del mensaje; delegar procesamiento a Application.
- En caso de excepción: loggear error con CorrelationId y relanzar para que Azure Functions aplique el MaxDeliveryCount nativo.
- No implementar lógica de retry personalizada.

Queue Storage Trigger:
- Clase Function con parámetro QueueMessage.
- Nombre de la cola configurable desde appsettings bajo "Messaging:QueueStorage:QueueName".
- Deserializar el mensaje; delegar a Application.
- En caso de excepción: relanzar para que Azure Functions mueva el mensaje a la poison queue tras superar el umbral de reintentos.

Blob Storage Trigger:
- Clase Function con parámetro BlobClient o Stream según necesidad.
- Container configurable desde appsettings bajo "Storage:Blob:ContainerName".
- Preguntar si el procesamiento requiere el contenido del blob o solo sus metadatos.
- En caso de excepción: loggear error y relanzar.

Event Grid Trigger:
- Clase Function con parámetro CloudEvent o EventGridEvent según el esquema configurado.
- Filtrar por EventType antes de delegar a Application.
- Responder 200 OK ante eventos de validación de suscripción (SubscriptionValidation).
- En caso de excepción: loggear error con EventId y relanzar.

BASELINE COMÚN A TODOS LOS TRIGGERS:
- Clase base FunctionBase (opcional pero recomendada) con logging y extracción de CorrelationId.
- CorrelationId extraído de las propiedades del mensaje/evento o generado si no viene.
- DependencyInjection del proyecto Functions registra los servicios de Application que cada trigger necesita.
- No contiene lógica de negocio; el trigger es solo el punto de entrada.

CONFIGURACIÓN:
- Connection strings bajo "AzureWebJobsStorage" (Queue/Blob) o "Messaging:ServiceBus:ConnectionString" (Service Bus).
- Leídos desde variables de entorno o user-secrets; nunca hardcodeados.
- Valores configurables en local.settings.json (excluido de git) para desarrollo local.

OBSERVABILIDAD:
- Log al recibir el trigger con identificadores únicos del mensaje/evento y CorrelationId.
- Log al completar el procesamiento exitoso.
- Log de error antes de relanzar la excepción.
- Usar ILogger<T> inyectado por DI.

RESTRICCIONES:
- No lógica de negocio en la clase del trigger.
- No hardcodear nombres de colas, topics, containers o expresiones cron.
- No usar el modelo in-process (Microsoft.Azure.WebJobs.*).
- No agregar dependencias de pago adicionales al SDK oficial.
- No mezclar múltiples responsabilidades en una misma clase de trigger.

DEFINITION OF DONE:
- dotnet restore → sin errores.
- dotnet build → sin errores.
- dotnet test → tests existentes pasan.
- El trigger compila y arranca localmente con Azure Functions Core Tools sin dependencias externas obligatorias (usar Azurite para Storage; emulador de Service Bus o mock para Service Bus).
- Los valores de configuración se leen desde local.settings.json o variables de entorno; ninguno hardcodeado.
- El trigger delega toda la lógica de negocio a Application; la clase del trigger contiene solo la entrada/salida.
- Los errores quedan registrados con CorrelationId antes de ser relanzados.
