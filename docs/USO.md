# Guía de uso

Esta guía explica cómo trabajar con la skill `migracion-segura-bd` sin poner en riesgo los datos.

## 1. Antes de empezar

Ten a mano esta información. No pegues contraseñas en el chat.

| Dato | Ejemplo |
|---|---|
| Conexión de origen | `produccion` |
| Conexión de destino | `desarrollo` |
| Base de origen | `app_prod` |
| Base de destino | `app_dev` |
| Motor y versión | MySQL 8.0 / MariaDB 10.11 |
| Alcance | Todas las tablas o lista concreta |
| Datos a excluir | `sessions`, `jobs`, `cache`, `tokens` |
| Datos locales a conservar | Usuarios de prueba, catálogos propios |
| Ventana de trabajo | Fuera de horas de alta carga |
| Responsable de aprobar | Tú o el rol indicado |
| Respaldo | Ruta y mecanismo aprobados |

La skill puede leer el `.env` y la configuración de Laravel para identificar conexiones, pero no debe mostrar secretos ni modificar credenciales.

## 2. Modos

### Modo A: Análisis

Objetivo: entender el terreno sin tocar nada.

Produce:

- Identificación de conexiones y entornos.
- Confirmación de que producción es solo lectura.
- Inventario de tablas y volumen estimado.
- Primeras diferencias de esquema.

Úsalo cuando:

- Es la primera vez que migras ese proyecto.
- No sabes si los esquemas coinciden.
- Cambió el equipo o la infraestructura.

### Modo B: Planificación

Objetivo: convertir el análisis en un plan revisable.

Produce:

- Matriz de diferencias.
- Correspondencia de columnas y transformaciones.
- Orden de carga por dependencias.
- Riesgos, conflictos y decisiones pendientes.
- Criterios de aceptación.

### Modo C: Simulación

Objetivo: medir el impacto antes de escribir.

Produce:

- Consultas de diagnóstico.
- Conteo de filas a insertar, actualizar u omitir.
- Detección de duplicados, huérfanos y desbordamientos.
- Scripts o archivos de trabajo, sin ejecutarlos automáticamente.

### Modo D: Ejecución aprobada

Objetivo: aplicar exactamente el plan aprobado.

Requisitos:

- Plan mostrado y aprobado de forma explícita.
- Respaldo de desarrollo verificado.
- Alcance cerrado: tablas, filas y transformaciones.
- Criterios de verificación definidos.

Durante la ejecución la skill debe detenerse ante cualquier condición no prevista.

### Modo E: Verificación

Objetivo: demostrar que la carga fue correcta.

Produce:

- Conteos y comparaciones.
- Integridad de claves y relaciones.
- Muestras de origen y destino.
- Informe final con incidencias y limitaciones.

## 3. Flujo con puntos de control

```text
Solicitud
   │
   ├─ A. Análisis ................ sin escritura
   ├─ B. Planificación ........... sin escritura
   ├─ C. Simulación .............. sin escritura
   │
   ├─ [Control 1] Plan presentado
   ├─ [Control 2] Aprobación explícita
   ├─ [Control 3] Respaldo verificado
   │
   ├─ D. Ejecución ............... solo desarrollo
   │
   ├─ [Control 4] Validaciones
   └─ E. Informe final
```

Ningún control se puede omitir porque "ya se probó antes" o porque la carga es pequeña.

## 4. Cómo pedir la migración

### Primera sesión recomendada

```text
Usa la skill migracion-segura-bd.

Contexto: proyecto Laravel 11, MySQL 8.0.
Origen: conexión `produccion`, base `app_prod`.
Destino: conexión `desarrollo`, base `app_dev`.

Objetivo: cargar en desarrollo los datos de producción para reproducir un bug.
Alcance: todas las tablas excepto sessions, jobs, cache, tokens y auditoría.
Conserva los usuarios y catálogos locales de desarrollo.

Haz solo Análisis y Planificación. No escribas en ninguna base.
```

### Continuar a simulación

```text
Con el análisis anterior, pasa a Simulación.
Mide filas, conflictos, huérfanos y desbordamientos.
No ejecutes escrituras.
```

### Aprobar ejecución

```text
Apruebo el plan presentado para las tablas `customers`, `orders` y `order_items`.
No apruebo cambios de esquema ni borrados.
Ejecuta solo inserciones y actualizaciones indicadas.
Detente ante cualquier discrepancia.
```

La aprobación debe nombrar el alcance. "Dale", "adelante" o "hazlo" sin alcance no son aprobaciones suficientes para operaciones destructivas.

### Verificar

```text
Ejecuta el Modo Verificación con los criterios acordados y emite el informe final.
No repares nada automáticamente.
```

## 5. Solicitudes problemáticas

| Solicitud | Problema | Respuesta esperada |
|---|---|---|
| "Copia producción a desarrollo y listo" | No define alcance, respaldo ni conflictos | La skill empieza por análisis y pide decisiones |
| "Haz `migrate:fresh` y carga" | Destruye desarrollo sin autorización | Se detiene y exige plan y aprobación |
| "Borra las tablas de desarrollo" | Operación destructiva | Requiere autorización explícita y respaldo |
| "Desactiva las foreign keys" | Rompe integridad | Solo con justificación y verificación posterior |
| "Usa `REPLACE INTO` para todo" | Oculta conflictos y sobrescribe | Solo si el comportamiento se analizó y aprobó |
| "Rellena la columna nueva con 0" | Inventa datos de negocio | Pide una regla validada |
| "Hazlo igual que la otra vez" | No hay evidencia del contexto actual | Revalida conexiones y esquema |
| "Luego lo pruebas" | Sin verificación no hay éxito | La skill exige validaciones e informe |

## 6. Aprobaciones

Una aprobación válida:

- Se refiere al plan mostrado.
- Indica tablas y operaciones.
- Indica si se permiten actualizaciones, borrados o cambios de esquema.
- Reconoce el respaldo disponible.
- No se extiende a pasos posteriores no descritos.

Una aprobación no válida:

- Silencio.
- "Ok" sin alcance para operaciones destructivas.
- Aprobación dada antes de ver el plan.
- Aprobación de una versión anterior del plan.

Si el plan cambia, se presenta de nuevo y se vuelve a pedir aprobación.

## 7. Qué hacer cuando la skill se detiene

La detención es un resultado correcto, no un fallo. Sigue este orden:

1. Lee el motivo y el alcance afectado.
2. No reintentes escrituras cuyo resultado sea incierto.
3. Consulta el estado real del destino.
4. Responde solo la pregunta pendiente: regla de negocio, exclusión, respaldo o alcance.
5. Vuelve a simular si cambió el plan.

Motivos habituales:

- Conexión de producción ambigua.
- Credenciales de producción con permisos de escritura.
- Respaldo ausente o no verificable.
- Columna obligatoria sin regla de población.
- Pérdida de precisión o longitud.
- Claves huérfanas o duplicadas.
- Volumen distinto al esperado.
- Tabla o columna fuera del plan.

## 8. Alcance y exclusiones

Define por escrito qué se migra y qué no. Como mínimo:

- Tablas incluidas.
- Tablas excluidas y motivo.
- Filas excluidas por rango, fecha o condición.
- Datos locales que se conservan.
- Columnas que no se copian.
- Transformaciones permitidas.

Ejemplo de exclusión:

```text
Excluir `sessions`, `cache`, `jobs`, `failed_jobs`, `password_reset_tokens`
y `personal_access_tokens`. No copiar `users.password` ni `users.remember_token`.
```

La skill no debe copiar a ciegas tablas operativas, de auditoría, colas, cachés o tokens.

## 9. Migración de datos frente a despliegue

Son operaciones distintas:

| Actividad | Destino | Esta skill |
|---|---|---|
| Cargar datos de producción en desarrollo | Desarrollo | Sí, con plan y aprobación |
| Cambiar esquema de desarrollo | Desarrollo | Solo si está en el plan aprobado |
| Desplegar esquema o código a producción | Producción | No, se trata como operación separada |

Preparar desarrollo con datos de producción **no** significa que sea seguro reemplazar producción con desarrollo. Desarrollo puede tener datos antiguos, locales o generados en pruebas.

## 10. Buenas prácticas

- Trabaja fuera de horas de alta carga.
- Usa una conexión de producción dedicada y de solo lectura.
- Migra por lotes cuando el volumen sea alto.
- Empieza por un subconjunto pequeño y verifica.
- Guarda el plan y el informe con la fecha.
- No reutilices aprobaciones de sesiones anteriores.
- No copies datos personales que no necesites.
- Revisa el informe final antes de cerrar la tarea.
