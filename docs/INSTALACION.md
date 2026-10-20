# Instalación

Esta guía instala la skill `migracion-segura-bd` en OpenCode, por proyecto o de forma global.

## 1. Requisitos previos

| Requisito | Comprobación |
|---|---|
| OpenCode | `opencode --version` |
| PHP y Laravel | `php artisan --version` |
| Cliente MySQL/MariaDB | `mysql --version` o `mariadb --version` |
| Acceso a producción | Credenciales de solo lectura |
| Acceso a desarrollo | Credenciales con permiso de escritura |
| Ubicación de respaldos | Ruta aprobada y con espacio suficiente |

No inicies la skill si no puedes confirmar cuál conexión es producción y cuál es desarrollo.

## 2. Estructura que espera OpenCode

OpenCode descubre skills en estas rutas:

- Proyecto: `.opencode/skills/<nombre>/SKILL.md`
- Global: `~/.config/opencode/skills/<nombre>/SKILL.md`
- Compatibilidad Claude: `.claude/skills/<nombre>/SKILL.md` y `~/.claude/skills/<nombre>/SKILL.md`
- Compatibilidad agentes: `.agents/skills/<nombre>/SKILL.md` y `~/.agents/skills/<nombre>/SKILL.md`

Reglas que deben cumplirse:

1. El archivo se llama exactamente `SKILL.md`, en mayúsculas.
2. La carpeta se llama igual que el campo `name` del frontmatter.
3. El nombre solo usa minúsculas, números y guiones simples.
4. `name` y `description` existen en el frontmatter.

En este repositorio:

```text
.opencode/skills/migracion-segura-bd/SKILL.md
```

```yaml
---
name: migracion-segura-bd
description: Analiza, planifica y ejecuta migraciones controladas...
---
```

La carpeta `skill/` en singular **no** se escanea. Si copiaste una versión anterior con `.opencode/skill/migracion-segura/`, muévela a `.opencode/skills/migracion-segura-bd/`.

## 3. Instalación por proyecto

La instalación por proyecto viaja con el repositorio y solo afecta a ese proyecto.

### Windows PowerShell

```powershell
$destino = ".opencode\skills"
New-Item -ItemType Directory -Path $destino -Force | Out-Null
Copy-Item -Recurse -Force "ruta\a\opencode-migracion-segura\.opencode\skills\migracion-segura-bd" $destino
```

### Linux o macOS

```bash
mkdir -p .opencode/skills
cp -R ruta/a/opencode-migracion-segura/.opencode/skills/migracion-segura-bd .opencode/skills/
```

Resultado esperado:

```text
tu-proyecto/
└─ .opencode/
   └─ skills/
      └─ migracion-segura-bd/
         └─ SKILL.md
```

Si el proyecto usa `.claude/skills` o `.agents/skills`, copia la misma carpeta allí. No dupliques la skill en dos ubicaciones del mismo proyecto: los nombres deben ser únicos.

## 4. Instalación global

La instalación global queda disponible para todos los proyectos del usuario.

### Windows

```powershell
$destino = "$env:USERPROFILE\.config\opencode\skills"
New-Item -ItemType Directory -Path $destino -Force | Out-Null
Copy-Item -Recurse -Force "ruta\a\opencode-migracion-segura\.opencode\skills\migracion-segura-bd" $destino
```

### Linux o macOS

```bash
mkdir -p ~/.config/opencode/skills
cp -R ruta/a/opencode-migracion-segura/.opencode/skills/migracion-segura-bd ~/.config/opencode/skills/
```

La ruta global de OpenCode en Windows es `C:\Users\<usuario>\.config\opencode\skills\`.

## 5. Exponer la skill como comando

Opcional. OpenCode puede mostrar la skill como `/migracion-segura-bd` en la TUI. Para ello agrega `metadata` al frontmatter de `SKILL.md`:

```yaml
---
name: migracion-segura-bd
description: Analiza, planifica y ejecuta migraciones controladas de datos entre bases de datos MySQL/MariaDB de producción y desarrollo para sistemas Laravel, priorizando la conservación íntegra de los datos originales, la trazabilidad y la verificación posterior.
metadata:
  opencode/slash: "true"
---
```

Sin ese campo la skill sigue disponible mediante el modelo y se puede invocar por nombre en el mensaje.

Para impedir que el modelo la seleccione automáticamente:

```yaml
metadata:
  opencode/slash: "true"
  opencode/autoinvoke: "false"
```

Con `autoinvoke` en `false`, solo se carga si se pide de forma explícita.

## 6. Permisos

OpenCode permite controlar el acceso a skills desde `opencode.json` (proyecto) o `~/.config/opencode/opencode.json` (global).

```json
{
  "permission": {
    "skill": {
      "*": "allow",
      "migracion-segura-bd": "allow"
    }
  }
}
```

Valores posibles:

| Valor | Comportamiento |
|---|---|
| `allow` | La skill se carga sin preguntar |
| `ask` | Se pide aprobación antes de cargarla |
| `deny` | La skill queda oculta y no se puede cargar |

Para un agente concreto:

```json
{
  "agent": {
    "build": {
      "permission": {
        "skill": {
          "migracion-segura-bd": "allow"
        }
      }
    }
  }
}
```

Si usas un agente personalizado, declara el permiso en su frontmatter:

```yaml
---
permission:
  skill:
    "migracion-segura-bd": "allow"
---
```

## 7. Verificación

1. Abre OpenCode dentro del proyecto Laravel.
2. Pregunta: `¿Qué skills tienes disponibles?`.
3. Confirma que aparece `migracion-segura-bd`.
4. Pide una prueba de solo lectura:

```text
Usa migracion-segura-bd en modo Análisis. No escribas en ninguna base.
Identifica las conexiones y confirma cuál es producción y cuál desarrollo.
```

La respuesta correcta debe describir conexiones y esquemas, y no ejecutar escrituras.

Comprobación directa del archivo:

```powershell
Get-Content ".opencode\skills\migracion-segura-bd\SKILL.md" -TotalCount 5
```

Debe mostrar el bloque `---` con `name` y `description`.

## 8. Solución de problemas

| Síntoma | Causa probable | Solución |
|---|---|---|
| La skill no aparece | Carpeta `skill/` en singular | Renombrar a `.opencode/skills/` |
| La skill no aparece | Archivo `skill.md` en minúsculas | Renombrar a `SKILL.md` |
| La skill no aparece | `name` distinto del nombre de carpeta | Igualar ambos |
| La skill no aparece | Permiso `deny` | Revisar `permission.skill` |
| La skill no aparece | `opencode/autoinvoke: "false"` | Invocarla por nombre o quitar el campo |
| La skill no aparece | Frontmatter inválido | Revisar comillas y sangría YAML |
| Se cargan dos versiones | Copia global y de proyecto | Eliminar una de las dos |
| El agente no la usa | Descripción poco específica o petición ambigua | Pedirla explícitamente por nombre |
| Error de nombre inválido | Mayúsculas, guiones dobles o guion final | Usar `^[a-z0-9]+(-[a-z0-9]+)*$` |

El nombre debe cumplir:

- Entre 1 y 64 caracteres.
- Solo minúsculas alfanuméricas y guiones simples.
- Sin guion al inicio o al final.
- Sin `--`.
- Igual al nombre de la carpeta contenedora.

## 9. Actualización

```powershell
Copy-Item -Recurse -Force "ruta\a\opencode-migracion-segura\.opencode\skills\migracion-segura-bd\*" ".opencode\skills\migracion-segura-bd\"
```

Antes de actualizar, revisa el `CHANGELOG` o el historial del repositorio. Si modificaste la copia local, guarda tus cambios o intégralos en el repositorio de origen.

## 10. Desinstalación

1. Elimina la carpeta `migracion-segura-bd` de `.opencode/skills/` o de la ruta global.
2. Elimina cualquier entrada específica en `permission.skill` si ya no aplica.
3. Reabre OpenCode y confirma que la skill ya no aparece.

No elimines bases de datos, respaldos ni archivos de configuración como parte de la desinstalación.
