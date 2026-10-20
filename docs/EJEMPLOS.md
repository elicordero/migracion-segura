# Ejemplos de uso

Casos de referencia. Los nombres, rutas y datos son ficticios.

## Ejemplo 1: esquemas equivalentes, subconjunto de tablas

### Solicitud

```text
Usa migracion-segura-bd.

Proyecto Laravel 11 con MySQL 8.0.
Origen: conexión `produccion`, base `tienda_prod`.
Destino: conexión `desarrollo`, base `tienda_dev`.

Necesito reproducir un bug con datos reales.
Incluye `customers`, `orders`, `order_items` y `products`.
Excluye `sessions`, `jobs`, `cache`, `tokens` y `auditoria`.
No cambies el esquema. No borres datos de desarrollo.

Empieza por Análisis y Planificación. No escribas nada.
```

### Respuesta esperada, resumida

```text
Modo: Análisis y Planificación. Sin escrituras.

Conexiones
- produccion -> tienda_prod (usuario de solo lectura confirmado)
- desarrollo -> tienda_dev (escritura permitida)

Volumen estimado
- customers: 12.480
- products: 3.210
- orders: 48.902
- order_items: 121.744

Diferencias de esquema
- Sin diferencias de columnas en las cuatro tablas.
- `orders.status` usa ENUM distinto: prod incluye 'refunded', dev no.
- Falta índice `orders_customer_id_foreign` en desarrollo.

Riesgos
- 143 pedidos con estado 'refunded' no pueden cargarse tal cual.
- 12 clientes con email duplicado en desarrollo.

Plan propuesto
1. Respaldo verificado de tienda_dev.
2. Cargar productos y clientes.
3. Cargar pedidos por lotes de 1.000.
4. Cargar líneas de pedido.
5. Verificar conteos, claves y muestras.

Pendiente de decisión
- ¿Ampliar el ENUM de desarrollo para incluir 'refunded'?
- ¿Qué hacer con los 12 emails duplicados?
```

### Aprobación

```text
Apruebo ampliar el ENUM de `orders.status` para incluir 'refunded' y cargar las cuatro tablas.
Para los emails duplicados, conserva el registro de desarrollo y omite el de producción, registrando cada omisión.
No borres ninguna fila.
```

La skill debe registrar la decisión, actualizar el plan y volver a pedir aprobación si el cambio de alcance altera el impacto.

## Ejemplo 2: columna nueva obligatoria y columna renombrada

### Contexto

Desarrollo tiene una migración reciente:

```text
production.customers.full_name      -> development.customers.nombre_completo
development.customers.tenant_id     -> NOT NULL, sin valor predeterminado
```

### Lo que no debe hacer la skill

- Asumir que `full_name` y `nombre_completo` son equivalentes solo por el nombre.
- Rellenar `tenant_id` con `1`.
- Convertir clientes sin tenant en `NULL` silenciosamente.

### Lo que debe hacer

```text
Evidencia de equivalencia
- Migración 2026_09_12_rename_customer_name: renombra full_name a nombre_completo.
- Modelo Customer usa nombre_completo en $fillable.
- No hay otras columnas candidatas.

Riesgo
- `tenant_id` es obligatorio y producción no lo tiene.

Pendiente de decisión
- ¿Todos los clientes pertenecen a un tenant conocido?
- ¿Existe una tabla de correspondencia?
- ¿Se excluyen los clientes sin tenant o se detiene la carga?
```

Con la regla aprobada, el mapa queda explícito:

| Origen | Destino | Regla | Evidencia | Riesgo | Validación |
|---|---|---|---|---|---|
| `full_name` | `nombre_completo` | Copia directa | Migración de renombre | Bajo | Comparar longitudes y muestras |
| — | `tenant_id` | Derivar de `tenant_map.customer_id` | Tabla entregada por negocio | Medio | Cero huérfanos |

## Ejemplo 3: carga bloqueada por pérdida de datos

### Diagnóstico

```text
Tabla: products
Columna destino: sku VARCHAR(32)
Columna origen:  sku VARCHAR(64)

Resultado
- 27 valores superan 32 caracteres.
- 3 valores difieren solo por collation.
- 1 valor contiene caracteres fuera de utf8mb4.

Decisión: no truncar ni normalizar.
```

### Respuesta esperada

```text
Ejecución detenida antes de escribir.

Motivo
- `products.sku` no cabe en el destino sin pérdida.
- Truncar produciría SKU inválidos y posibles duplicados.

Opciones
A. Ampliar `products.sku` a VARCHAR(64) en desarrollo, con aprobación.
B. Excluir los 27 productos y registrar la exclusión.
C. Mapear los SKU a un catálogo nuevo, si negocio lo define.

No se modificó ninguna base.
```

## Ejemplo 4: conservar datos locales de desarrollo

### Contexto

Desarrollo tiene usuarios de prueba y catálogos propios que deben conservarse. Producción tiene usuarios reales con los mismos IDs.

### Enfoque correcto

1. Identificar la clave de conflicto: `users.id` y `users.email`.
2. Decidir la política por tabla:
   - Usuarios: conservar los de desarrollo; insertar solo los que no existan por email.
   - Catálogos: conservar los locales; insertar faltantes.
   - Pedidos: reemplazar solo los pedidos de los clientes migrados, con respaldo previo.
3. Documentar qué se actualiza y qué se omite.
4. Verificar que no queden pedidos apuntando a usuarios no migrados.

### Aprobación típica

```text
Apruebo:
- insertar usuarios de producción cuyo email no exista en desarrollo;
- omitir usuarios con email duplicado, registrando el ID de producción;
- insertar catálogos faltantes;
- no actualizar ni borrar usuarios locales;
- cargar pedidos solo para usuarios insertados.

No apruebo cambios de esquema ni borrados.
```

## Ejemplo 5: tabla operativa fuera de alcance

```text
Petición: "trae todo, incluido sessions y cache".

Respuesta esperada
- `sessions`, `cache` y `cache_locks` son datos operativos del entorno.
- `jobs` y `failed_jobs` pueden contener payloads con datos sensibles.
- `personal_access_tokens` permite autenticación y no debe copiarse.
- Se excluyen por defecto y se pide confirmación si se necesitan.
```

## Ejemplo 6: informe final

```text
Resumen
- Fecha: 2026-10-20 21:40 -03:00
- Origen: tienda_prod (producción, solo lectura)
- Destino: tienda_dev (desarrollo)
- Modo: Ejecución aprobada + Verificación
- Resultado: completada con incidencias

Tablas afectadas
| Tabla | Acción | Origen | Insertadas | Actualizadas | Omitidas | Errores | Validación |
|---|---|---:|---:|---:|---:|---:|---|
| customers | Insert | 12.480 | 12.468 | 0 | 12 | OK con omisiones |
| products | Insert | 3.210 | 3.210 | 0 | 0 | OK |
| orders | Insert | 48.902 | 48.902 | 0 | 0 | OK |
| order_items | Insert | 121.744 | 121.744 | 0 | 0 | OK |

Cambios realizados
- `orders.status`: ENUM ampliado para incluir 'refunded'. Aprobado el 2026-10-20.
- Sin otros cambios de esquema.

Validaciones
- Conteos por tabla y por rango de ID: coinciden con lo insertado.
- 0 claves foráneas huérfanas en `orders` y `order_items`.
- 12 usuarios omitidos por email duplicado; IDs registrados en el anexo.
- Muestras de 100 filas por tabla: sin diferencias de contenido.
- Limitación: no se compararon hashes de todas las filas.

Respaldo y recuperación
- Respaldo: respaldo_tienda_dev_2026-10-20.sql, 184 MB, restaurado en base temporal: OK.
- Recuperación: restaurar el respaldo y volver a ejecutar el plan corregido.

Protección de producción
- Producción no recibió escrituras. Verificado mediante usuario de solo lectura y registro de consultas.
```

## Patrones de petición

| Bien | Mal |
|---|---|
| "Analiza y propone plan, sin escribir" | "Hazlo todo ya" |
| "Incluye estas tablas y excluye estas" | "Trae lo que haga falta" |
| "Conserva estos datos locales" | "Borra desarrollo y carga" |
| "Apruebo estas tablas y estas operaciones" | "Ok" |
| "Detente si hay pérdida de datos" | "Arregla lo que salga" |
| "Emite el informe final" | "Si funciona, listo" |
