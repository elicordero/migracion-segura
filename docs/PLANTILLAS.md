# Plantillas

Plantillas para copiar en el plan, el registro de ejecución y el informe final. Ajusta las columnas al caso; no elimines las que documentan riesgo o trazabilidad.

## 1. Ficha de contexto

```markdown
# Migración de datos: <proyecto>

- Fecha:
- Responsable:
- Aprobador:
- Proyecto / rama:
- Framework / versión Laravel:
- Motor y versión:
- Origen: <conexión> / <base> (producción, solo lectura)
- Destino: <conexión> / <base> (desarrollo)
- Modo actual: A / B / C / D / E
- Ventana de trabajo:
- Ubicación de respaldos:
```

## 2. Matriz de diferencias

```markdown
| Tabla | Elemento | Producción | Desarrollo | Riesgo | Acción propuesta | Estado |
|---|---|---|---|---|---|---|
| customers | columna tenant_id | no existe | NOT NULL | Alto | requiere regla de población | pendiente |
| orders | columna status | ENUM 5 valores | ENUM 4 valores | Medio | ampliar ENUM | aprobado |
| products | índice sku | único | no único | Medio | crear índice único | pendiente |
```

Estados sugeridos: `pendiente`, `aprobado`, `aplicado`, `omitido`, `bloqueado`.

## 3. Mapa de columnas

```markdown
| Tabla | Origen | Destino | Transformación | Evidencia | Pérdida posible | Validación |
|---|---|---|---|---|---|---|
| customers | full_name | nombre_completo | copia directa | migración de renombre | no | longitudes y muestras |
| customers | — | tenant_id | lookup en tenant_map | regla de negocio 2026-10-18 | no | cero huérfanos |
| orders | total | total | decimal(10,2) -> decimal(12,2) | cambio de tipo | no | rango y suma de control |
| legacy | notes | observaciones | copia, sin recorte | análisis de longitud | no | máximo 500 caracteres |
```

Si una transformación no es reversible, indícalo en `Pérdida posible` y explica la alternativa.

## 4. Orden de carga

```markdown
1. Catálogos sin dependencias: countries, currencies, categories.
2. Entidades padre: customers, products.
3. Entidades dependientes: orders.
4. Detalle: order_items.
5. Tablas pivote: category_product, order_tag.
6. Verificación de relaciones y conteos.
```

## 5. Estrategia de conflictos

```markdown
| Tabla | Clave en conflicto | Política | Justificación | Registro |
|---|---|---|---|---|
| users | email único | omitir fila de producción | preservar usuario local | guardar id de producción |
| products | sku único | actualizar si coincide | catálogo maestro | guardar valores anteriores |
| orders | id primario | insertar con id de origen | preservar referencias | verificar auto_increment |
```

## 6. Plan de ejecución

```markdown
# Plan de migración

## Alcance
- Tablas incluidas:
- Tablas excluidas:
- Filas excluidas:
- Datos locales que se conservan:

## Operaciones
| # | Tabla | Operación | Filas estimadas | Destructiva | Reversible |
|---|---|---|---:|---|---|
| 1 | customers | INSERT | 12.480 | no | sí, borrar por lote |
| 2 | orders | INSERT | 48.902 | no | sí, borrar por lote |
| 3 | orders | ALTER status ENUM | — | no | sí, revertir ENUM |

## Respaldos
- Desarrollo:
- Verificación del respaldo:
- Recuperación:

## Riesgos
| Riesgo | Probabilidad | Impacto | Mitigación |
|---|---|---|---|

## Criterios de aceptación
- Conteos:
- Claves:
- Relaciones:
- Muestras:
- Limitaciones:

## Detención
- Condiciones:
- Estado esperado al detenerse:
- Contacto:
```

## 7. Registro de aprobación

```markdown
# Aprobación

- Plan presentado: <fecha y hora>
- Aprobado por:
- Fecha y hora:
- Tablas aprobadas:
- Operaciones aprobadas:
- Operaciones prohibidas:
- Cambios de esquema aprobados:
- Respaldo aceptado:
- Condiciones:
```

Sin este registro no se inicia el Modo D.

## 8. Registro de ejecución

```markdown
| Hora | Etapa | Tabla | Operación | Filas | Resultado | Incidencia |
|---|---|---|---:|---|---|---|
| 21:05 | 1 | customers | INSERT lote 1 | 1.000 | OK | — |
| 21:07 | 1 | customers | INSERT lote 2 | 1.000 | OK | — |
| 21:20 | 3 | orders | ALTER ENUM | — | OK | commit implícito |
```

Anota las operaciones DDL y cualquier resultado incierto. No reintentes sin consultar el estado real.

## 9. Informe final

```markdown
# Informe de migración

## Resumen
- Fecha y hora:
- Origen:
- Destino:
- Entornos identificados:
- Modo:
- Resultado: completada / completada con incidencias / detenida / fallida

## Tablas afectadas
| Tabla | Acción | Filas de origen | Insertadas | Actualizadas | Omitidas | Errores | Validación |
|---|---|---:|---:|---:|---:|---:|---|
| | | | | | | | |

## Cambios realizados
| Tabla | Columna | Anterior | Nuevo | Transformación | Motivo | Aprobación | Filas | Reversible |
|---|---|---|---|---|---|---|---:|---|
| | | | | | | | | |

## Validaciones
- Conteos:
- Claves primarias y únicas:
- Integridad referencial:
- `NULL`, longitudes y rangos:
- Columnas críticas:
- Sumas de control o hashes:
- Muestras:
- Fechas, decimales y collation:
- Compatibilidad Laravel:
- Limitaciones:

## Incidencias
| Tabla | Descripción | Filas afectadas | Acción tomada | Pendiente |
|---|---|---:|---|---|

## Respaldo y recuperación
- Respaldo:
- Verificación:
- Recuperación recomendada:
- Intervención manual requerida:

## Protección de producción
- Escrituras en producción: ninguna / no verificado
- Evidencia:
```

Si una métrica no se obtuvo, escribe `no verificada`. No inventes cifras.

## 10. Cierre de sesión

```markdown
- [ ] Informe emitido
- [ ] Incidencias declaradas
- [ ] Pendientes asignados
- [ ] Respaldo conservado según política
- [ ] Aprobaciones archivadas
- [ ] Datos temporales eliminados
- [ ] Producción sin cambios o estado reportado
```
