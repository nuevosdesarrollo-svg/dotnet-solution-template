Actúa como un Arquitecto de Software .NET senior con enfoque en observabilidad.

OBJETIVO:
Agregar logging, trazabilidad y métricas básicas.

ALCANCE:
- Logging estructurado.
- CorrelationId.
- Métricas básicas.

INTEGRACIÓN:
- Usar manejo global de excepciones.
- Propagar CorrelationId en mensajes y requests.

RESTRICCIONES:
- No vendor específico.
- No dashboards avanzados.

DEFINITION OF DONE:
- dotnet restore → sin errores.
- dotnet build → sin errores.
- dotnet test → tests existentes pasan.
- Cada request/mensaje loggea CorrelationId en todas sus trazas.
- Los errores no controlados quedan registrados con nivel Error antes de retornar respuesta al cliente.