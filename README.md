# dotnet-solution-template

Plantilla base para iniciar soluciones .NET en el ecosistema de **nuevosdesarrollo-svg**.

## ¿Qué incluye este template?

Estructura mínima y gobernada para comenzar de forma ordenada:

- `dotnet-solution-template.sln` (solución vacía)
- `src/` y `tests/` para código y pruebas
- `.gitignore` optimizado para .NET
- `global.json` para fijar SDK recomendado
- `Directory.Build.props` con propiedad común inicial

> Este repositorio **no** incluye carpeta `prompts/` ni workflows locales por defecto.

## Uso del template

1. Usa este repositorio como base para tu nuevo proyecto.
2. Crea tus proyectos dentro de `src/` y `tests/`.
3. Agrega los proyectos a la solución con `dotnet sln add`.

## Sincronización de prompts y workflows (obligatoria)

La sincronización de prompts y workflows se gestiona desde el repositorio central:

- <https://github.com/nuevosdesarrollo-svg/dotnet-prompts>

Sigue las instrucciones del README de `dotnet-prompts` y ejecuta el workflow de sincronización para copiar al repositorio objetivo:

- Prompts de trabajo del equipo
- Workflows de gobernanza/seguridad/health-check
- Plantillas iniciales autorizadas

## Importante para los equipos

- No crees manualmente `prompts/` ni workflows locales en esta plantilla.
- Tras la **primera sincronización**, personaliza este `README.md` con contexto de tu proyecto (objetivo, arquitectura, convenciones y proceso de trabajo).
