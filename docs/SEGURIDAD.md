# Seguridad y protección de datos

Este documento desarrolla las reglas de seguridad de `SKILL.md`. No las reemplaza: ante conflicto, manda `SKILL.md`.

## 1. Modelo de riesgo

| Riesgo | Consecuencia | Control |
|---|---|---|
| Escritura accidental en producción | Pérdida o corrupción de datos reales | Conexión de solo lectura y verificación previa |
| Sobrescritura de desarrollo | Pérdida de datos locales o de pruebas | Respaldo y alcance aprobado |
| Correspondencia incorrecta de columnas | Datos válidos en el campo equivocado | Evidencia semántica, no solo nombres |
| Conversión silenciosa | Truncamiento, redondeo o cambio de zona horaria | Validación de rangos y longitudes |
| Conflicto de claves | Filas omitidas o relaciones rotas | Análisis de duplicados y huérfanos |
| Fuga de secretos | Credenciales expuestas en chat o registros | Redacción y variables de entorno |
| Resultado incierto | Reintentos que duplican datos | Consultar estado antes de reintentar |
| Despliegue prematuro | Producción recibe datos de desarrollo | Separar migración de despliegue |

## 2. Producción es solo lectura

Está prohibido ejecutar sobre producción:

- `INSERT`, `UPDATE`, `DELETE`, `REPLACE`, `TRUNCATE`.
- `ALTER`, `DROP`, `CREATE`, `RENAME`.
- Procedimientos, funciones o eventos con efectos secundarios.
- `migrate`, `migrate:fresh`, `migrate:refresh`, `db:wipe`, `schema:drop`, `db:seed`.
- Restauraciones de respaldo.
- Cualquier sentencia con `SELECT ... INTO OUTFILE`, `LOAD DATA` o bloqueos deliberados.

La skill puede ejecutar sobre producción únicamente consultas de lectura: `SELECT`, `SHOW`, `DESCRIBE`, `EXPLAIN` y consultas a `information_schema`.

### Verificación de la conexión

Antes de cualquier escritura en desarrollo:

1. Confirmar host, puerto, base y usuario de cada conexión.
2. Confirmar que el usuario de producción no tiene permisos de escritura.
3. Confirmar que el destino es desarrollo, no una réplica de producción.
4. Ocultar contraseñas y cadenas completas.
5. Detenerse si hay duda.

`APP_ENV=production` o `APP_ENV=local` no prueban por sí solos qué base hay detrás.

### Usuario de solo lectura

Crear un usuario dedicado reduce el riesgo de error humano. Ejemplo para MySQL/MariaDB, sujeto a aprobación del DBA:

```sql
CREATE USER 'migrador_ro'@'%' IDENTIFIED BY '<contraseña-segura>';
GRANT SELECT, SHOW VIEW ON app_prod.* TO 'migrador_ro'@'%';
```

Notas:

- No otorgar `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`, `ALTER`, `FILE` ni `SUPER`.
- Restringir el host de origen siempre que sea posible.
- Usar TLS si la red no es confiable.
- Guardar la contraseña en el gestor de secretos o en una variable de entorno, nunca en el repositorio.
- Revisar los permisos efectivos con `SHOW GRANTS FOR CURRENT_USER();`.

Si no se puede garantizar el solo lectura, no se inicia la ejecución.

## 3. Respaldo de desarrollo

Antes de modificar desarrollo:

1. Determinar si el destino contiene datos que deben conservarse.
2. Generar o verificar un respaldo consistente.
3. Comprobar que el archivo existe, tiene tamaño razonable y pertenece a la base correcta.
4. Cuando sea viable, restaurarlo en una base temporal para probar que es legible.
5. Registrar ruta, fecha, tamaño y resultado de la verificación.

Ejemplo de respaldo de desarrollo:

```bash
mysqldump \
  --host=<host-dev> \
  --user=<usuario-dev> \
  --single-transaction \
  --routines \
  --triggers \
  --events \
  --default-character-set=utf8mb4 \
  <base_dev> > respaldo_dev_2026-10-20.sql
```

Buenas prácticas:

- No incluir la contraseña en la línea de comandos; usar el archivo de opciones del cliente o una variable de entorno.
- No guardar el respaldo en el repositorio ni en carpetas sincronizadas sin autorización.
- No asumir que un archivo existente es un respaldo válido.
- No restaurar automáticamente.
- No reemplazar una base completa sin aprobación explícita.

Un respaldo de producción solo se realiza con el procedimiento aprobado del equipo y sin modificar la base de origen.

## 4. Operaciones destructivas

| Operación | Aprobación | Requisitos adicionales |
|---|---|---|
| `INSERT` en desarrollo | Plan aprobado | Alcance y conteos definidos |
| `UPDATE` en desarrollo | Plan aprobado | Filas objetivo identificadas |
| `DELETE` en desarrollo | Explícita y específica | Respaldo verificado y respaldo de filas afectadas |
| `TRUNCATE` | Explícita y específica | Respaldo verificado; justificar por qué no un `DELETE` acotado |
| `DROP` de tabla o columna | Explícita y específica | Análisis de dependencias y recuperación |
| `ALTER` de tipo o restricción | Explícita y específica | Evaluar bloqueos y pérdida de datos |
| Desactivar `FOREIGN_KEY_CHECKS` | Explícita y específica | Justificación y verificación posterior de todas las relaciones |
| Restaurar respaldo | Explícita y específica | Confirmar destino y ventana |

"Aprobado el plan" no autoriza por sí solo un `DROP`, `TRUNCATE` o `DELETE` que no estuviera descrito en ese plan.

## 5. Transformación de datos

- No truncar cadenas para que quepan.
- No redondear decimales sin regla de negocio.
- No convertir `NULL` en `''`, `0` o una fecha arbitraria.
- No cambiar zonas horarias sin documentarlo.
- No inventar valores para claves foráneas.
- No reutilizar identificadores para "hacer que cuadre".
- No normalizar mayúsculas, acentos o collation de forma implícita.

Toda transformación debe ser determinista, documentada y verificable. Si no es reversible o sin pérdida, se detiene esa parte y se presentan alternativas.

## 6. Secretos y datos personales

- No mostrar contraseñas, tokens, cadenas de conexión completas ni claves de aplicación.
- No volcar `.env` completo en el chat.
- Redactar la salida de errores antes de compartirla.
- No enviar datos de producción a servicios externos ni a modelos remotos sin autorización.
- Limitar la inspección a metadatos y muestras mínimas.
- Enmascarar datos personales en ejemplos e informes.
- No incluir datos reales en el plan, el informe ni los documentos del repositorio.
- No versionar respaldos, volcados ni archivos `.sql` con datos.

Si un secreto se expone durante la sesión, se debe rotar; borrar el mensaje no es suficiente.

## 7. Integridad referencial

- Cargar primero las tablas padre y después las hijas.
- No desactivar restricciones como solución por defecto.
- Detectar huérfanos antes de crear claves foráneas.
- Revisar relaciones uno a uno, uno a muchos, muchos a muchos y polimórficas.
- Verificar tablas pivote y sus claves compuestas.
- No cambiar identificadores sin analizar todas las referencias.

Si una relación no se puede resolver con los datos disponibles, se detiene la tabla afectada y se pide una regla.

## 8. Errores y resultados inciertos

Detenerse cuando:

- No se puede confirmar qué conexión es producción o desarrollo.
- No hay respaldo verificable del destino.
- Se detecta una conversión con pérdida.
- Hay conflictos de claves o relaciones sin resolver.
- El volumen difiere inesperadamente.
- Aparece una tabla o columna fuera del plan.
- Se requiere una operación destructiva no aprobada.
- La conexión de origen permite escritura y no se puede garantizar su uso seguro.
- Se pierde la conexión o el resultado queda indeterminado.

Ante un resultado incierto:

1. No reintentar a ciegas.
2. Consultar el estado real del destino.
3. Comparar conteos y claves afectadas.
4. Determinar qué se completó.
5. Reportar el estado y pedir instrucciones.

## 9. Verificación

Los conteos iguales no prueban que los datos sean idénticos. Se deben elegir verificaciones que detecten cambios de contenido:

- Conteos por tabla y por rangos de claves.
- Claves primarias y únicas duplicadas.
- Integridad referencial y huérfanos.
- Longitudes máximas, `NULL` y rangos numéricos.
- Sumas de control o hashes cuando sean viables.
- Muestras comparadas de origen y destino.
- Consistencia de fechas, decimales, codificación y collation.
- Compatibilidad con el esquema y el código Laravel previsto.

Las validaciones nunca escriben en producción. Si una validación falla, se detiene el proceso y se propone una recuperación segura.

## 10. Despliegue a producción

Preparar desarrollo con datos de producción no autoriza a desplegar en producción.

Si se requiere un despliegue:

1. Tratarlo como una operación distinta.
2. Revisar migraciones de Laravel y compatibilidad hacia atrás.
3. Identificar cambios destructivos, bloqueos y tiempos.
4. Preparar plan de despliegue y recuperación.
5. Obtener aprobación explícita para cada escritura en producción.
6. No ejecutarlo como consecuencia automática de la migración de desarrollo.

## 11. Lista mínima antes de escribir

- [ ] Producción identificada y de solo lectura.
- [ ] Desarrollo identificado y respaldado.
- [ ] Plan presentado y aprobado con alcance.
- [ ] Matriz de diferencias revisada.
- [ ] Correspondencias y transformaciones documentadas.
- [ ] Duplicados y huérfanos analizados.
- [ ] Orden de carga definido.
- [ ] Criterios de aceptación definidos.
- [ ] Secretos fuera del chat y de los archivos.
- [ ] Estrategia de detención y recuperación acordada.
