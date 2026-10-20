# Migración segura de bases de datos Laravel

Skill para OpenCode que analiza, planifica, ejecuta y verifica migraciones de datos entre una base MySQL/MariaDB de producción y una de desarrollo en proyectos Laravel.

La prioridad absoluta es **preservar los datos de producción**. Producción se trata como solo lectura; desarrollo es el único destino de escritura y solo después de un plan aprobado.

## Qué resuelve

- Diferencias de esquema entre producción y desarrollo.
- Correspondencia y transformación explícita de columnas.
- Cargas parciales o totales con orden de dependencias.
- Conflictos de claves primarias, únicas y foráneas.
- Respaldos, validaciones posteriores e informe final.
- Separación estricta entre migrar datos y desplegar cambios en producción.

## Qué no hace

- No escribe, borra ni altera producción.
- No ejecuta `migrate:fresh`, `db:wipe`, `schema:drop` ni restauraciones automáticas.
- No inventa reglas de negocio para columnas nuevas u obligatorias.
- No despliega código ni esquema a producción como parte de la migración.
- No sustituye un respaldo probado ni una revisión humana del plan.

## Requisitos

- OpenCode instalado.
- Proyecto Laravel con MySQL o MariaDB.
- Acceso de red a ambas bases.
- Credenciales de producción con permisos de solo lectura.
- Cliente `mysql`/`mariadb` o el mecanismo de respaldo que use el equipo.
- Espacio y ubicación aprobada para respaldos de desarrollo.

## Instalación rápida

El repositorio ya tiene la estructura correcta. Copia la carpeta del skill a tu proyecto o a tu configuración global.

Instalación por proyecto:

```powershell
New-Item -ItemType Directory -Path ".opencode\skills" -Force
Copy-Item -Recurse -Force "ruta\a\opencode-migracion-segura\.opencode\skills\migracion-segura-bd" ".opencode\skills\"
```

Instalación global en Windows:

```powershell
Copy-Item -Recurse -Force ".opencode\skills\migracion-segura-bd" "$env:USERPROFILE\.config\opencode\skills\"
```

La ruta y el nombre deben coincidir siempre:

```text
.opencode/skills/migracion-segura-bd/SKILL.md
```

El campo `name` del frontmatter es `migracion-segura-bd` y debe ser igual al nombre de la carpeta.

Guía completa: [docs/INSTALACION.md](docs/INSTALACION.md).

## Uso

Abre OpenCode en el proyecto Laravel y pide la migración en lenguaje natural. La skill se activa por su descripción o de forma explícita:

```text
Usa la skill migracion-segura-bd para analizar la migración de producción a desarrollo.
No modifiques nada todavía: quiero el análisis y el plan.
```

Flujo obligatorio, resumido:

1. Confirmar objetivo y entornos.
2. Verificar conexiones y protección de producción.
3. Analizar esquemas y datos.
4. Generar la matriz de diferencias.
5. Definir correspondencias y transformaciones.
6. Preparar respaldo de desarrollo.
7. Simular y presentar el plan.
8. Obtener aprobación explícita.
9. Ejecutar solo lo aprobado.
10. Validar y emitir el informe final.

Guía de uso y ejemplos de conversación: [docs/USO.md](docs/USO.md) y [docs/EJEMPLOS.md](docs/EJEMPLOS.md).

## Modos de operación

| Modo | Nombre | Escribe en desarrollo | Requiere aprobación |
|---|---|---|---|
| A | Análisis | No | No |
| B | Planificación | No | No |
| C | Simulación | No | No |
| D | Ejecución aprobada | Sí | Sí, explícita |
| E | Verificación | No | No |

Una solicitud general como "migra producción a desarrollo" no autoriza borrar, sobrescribir ni ejecutar operaciones irreversibles. La skill empieza por análisis, planificación y simulación.

## Documentación

| Documento | Contenido |
|---|---|
| [docs/INSTALACION.md](docs/INSTALACION.md) | Instalación local y global, permisos, verificación y solución de problemas |
| [docs/USO.md](docs/USO.md) | Modos, flujo, aprobaciones, información que debe aportar el usuario |
| [docs/SEGURIDAD.md](docs/SEGURIDAD.md) | Protección de producción, respaldos, secretos, detención segura |
| [docs/EJEMPLOS.md](docs/EJEMPLOS.md) | Sesiones de ejemplo, casos con diferencias de esquema y casos bloqueados |
| [docs/CHECKLIST.md](docs/CHECKLIST.md) | Listas de verificación por etapa |
| [docs/PLANTILLAS.md](docs/PLANTILLAS.md) | Plan, matriz de diferencias, mapa de columnas e informe final |

## Estructura del repositorio

```text
opencode-migracion-segura/
├─ .opencode/
│  └─ skills/
│     └─ migracion-segura-bd/
│        └─ SKILL.md
├─ docs/
│  ├─ INSTALACION.md
│  ├─ USO.md
│  ├─ SEGURIDAD.md
│  ├─ EJEMPLOS.md
│  ├─ CHECKLIST.md
│  └─ PLANTILLAS.md
└─ README.md
```

`SKILL.md` es la única fuente de comportamiento. El resto de documentos explica cómo instalarlo, usarlo y verificarlo; no duplica reglas nuevas.

## Verificación de la instalación

1. Abre OpenCode en el proyecto.
2. Pregunta: "¿Qué skills tienes disponibles?".
3. Confirma que aparece `migracion-segura-bd`.
4. Pide un análisis sin escritura y comprueba que la skill se detiene antes de ejecutar.

Si no aparece, revisa [docs/INSTALACION.md](docs/INSTALACION.md).

## Mantenimiento

- Trata `SKILL.md` como código: cambios revisados y versionados.
- Mantén `name` igual al nombre de la carpeta.
- No agregues scripts ni dependencias sin necesidad; la skill opera con SQL de solo lectura y herramientas del proyecto.
- Si cambias el flujo o las reglas, actualiza `docs/USO.md`, `docs/CHECKLIST.md` y `docs/PLANTILLAS.md` en el mismo cambio.
- El alcance actual es Laravel con MySQL/MariaDB. Otros motores requieren revisar tipos, DDL transaccional y sintaxis de `information_schema`.
