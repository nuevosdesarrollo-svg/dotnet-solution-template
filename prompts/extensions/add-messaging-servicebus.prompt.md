Actúa como un Arquitecto de Software .NET senior experto en sistemas event-driven y Azure Service Bus.

OBJETIVO:
Agregar un Worker Service para mensajería con Azure Service Bus, siguiendo el SDK oficial y las convenciones del proyecto.

COMPORTAMIENTO:
- Preguntar si se usará Queue o Topic/Subscription y justificar la elección según el caso de uso.
- Explicar la diferencia: Queue para procesamiento punto a punto; Topic/Subscription para distribución a múltiples consumidores.
- No mezclar ambos modelos en el mismo Worker sin justificación.

TECNOLOGÍA:
- Paquete oficial: Azure.Messaging.ServiceBus (no usar el paquete legacy WindowsAzure.ServiceBus).
- ServiceBusClient registrado como singleton en DI.
- ServiceBusProcessor por cada consumidor, configurado en DI.

CONFIGURACIÓN:
- Connection string bajo "Messaging:ServiceBus:ConnectionString" (leído desde variable de entorno o user-secrets, nunca hardcodeado).
- Para Queue: "Messaging:ServiceBus:QueueName".
- Para Topic/Subscription: "Messaging:ServiceBus:TopicName" y "Messaging:ServiceBus:SubscriptionName".

BASELINE:
- SubscriptionBase<TMensaje> encapsulando la lógica de deserialización y despacho.
- Consumidor de ejemplo concreto que hereda de SubscriptionBase.
- Contrato de mensaje explícito (clase o record) en capa de Application o Domain según corresponda.

MANEJO DE ERRORES Y DEAD-LETTER:
- Envolver el procesamiento en try/catch dentro del handler del ServiceBusProcessor.
- En caso de excepción: loggear el error con CorrelationId y llamar AbandonMessageAsync para que Service Bus aplique reintentos automáticos.
- No implementar lógica de retry personalizada; confiar en el MaxDeliveryCount nativo de Service Bus.
- Documentar en el README del Worker cuántos intentos están configurados (MaxDeliveryCount recomendado: 3–5).
- Mensajes que superan MaxDeliveryCount son movidos automáticamente a dead-letter por Service Bus.

INYECCIÓN DE DEPENDENCIAS:
- Clase DependencyInjection del Worker que registra ServiceBusClient, procesadores y consumidores.

OBSERVABILIDAD:
- Log al recibir el mensaje (con MessageId y CorrelationId).
- Log al completar el procesamiento exitoso.
- Log de error antes de abandonar el mensaje.
- CorrelationId extraído de ApplicationProperties del mensaje o generado si no viene.

RESTRICCIONES:
- No retries complejos ni circuit breaker.
- No lógica de negocio en el consumidor.
- No dependencias de pago adicionales al SDK oficial.

DEFINITION OF DONE:
- dotnet restore → sin errores.
- dotnet build → sin errores.
- dotnet test → tests existentes pasan.
- La connection string se lee desde configuración/variables de entorno, nunca hardcodeada.
- Un mensaje procesado exitosamente llama CompleteMessageAsync.
- Un mensaje fallido llama AbandonMessageAsync y registra el error con CorrelationId.