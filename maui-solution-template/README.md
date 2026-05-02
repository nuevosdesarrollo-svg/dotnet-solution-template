# maui-solution-template

Plantilla base para iniciar soluciones **.NET MAUI** en el ecosistema de **nuevosdesarrollo-svg**.

## ¿Qué incluye este template?

Estructura mínima y gobernada para comenzar de forma ordenada:

- `maui-solution-template.sln` (solución vacía)
- `src/` y `tests/` para código y pruebas
- `.gitignore` optimizado para .NET y MAUI
- `global.json` fijando el SDK `8.0.100` con `rollForward: latestFeature`
- `Directory.Build.props` con propiedades base incluyendo versiones mínimas de OS para cada plataforma MAUI

> Este repositorio **no** incluye carpeta `prompts/` ni workflows locales por defecto.

## Uso del template

1. Usa este repositorio como base para tu nuevo proyecto MAUI.
2. Crea tu proyecto dentro de `src/` con `dotnet new maui -n <NombreProyecto>`.
3. Agrega el proyecto a la solución con `dotnet sln add src/<NombreProyecto>/<NombreProyecto>.csproj`.

## Sincronización de prompts y workflows (obligatoria)

La sincronización de prompts y workflows se gestiona desde el repositorio central:

- <https://github.com/nuevosdesarrollo-svg/dotnet-prompts>

Sigue las instrucciones del README de `dotnet-prompts` y ejecuta el workflow de sincronización para copiar al repositorio objetivo (sección **mobile**):

- Prompts de trabajo del equipo para MAUI
- Workflows de gobernanza/seguridad/health-check
- Plantillas iniciales autorizadas

## Importante para los equipos

- No crees manualmente `prompts/` ni workflows locales en esta plantilla.
- Tras la **primera sincronización**, personaliza este `README.md` con contexto de tu proyecto (objetivo, arquitectura, plataformas objetivo, convenciones y proceso de trabajo).
