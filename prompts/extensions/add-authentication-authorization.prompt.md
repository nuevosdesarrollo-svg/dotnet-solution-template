Actúa como un Arquitecto de Software .NET senior con enfoque en seguridad.

OBJETIVO:
Agregar autenticación y autorización desacoplada y segura.

TECNOLOGÍA:
- JWT Bearer
- Claims y Policies

UBICACIÓN:
- Implementación en Infraestructura.
- Middleware en inicializadores.

INYECCIÓN DE DEPENDENCIAS:
- AddAuthenticationAndAuthorization por capa.

RESTRICCIONES:
- No refresh tokens.
- No UI.

DEFINITION OF DONE:
- dotnet restore → sin errores.
- dotnet build → sin errores.
- dotnet test → tests existentes pasan.
- El SecretKey de JWT se lee desde variable de entorno o user-secrets, nunca hardcodeado.
- Endpoint protegido retorna 401 sin token y 403 ante permisos insuficientes.