---
name: migracion-segura-bd
description: Analiza, planifica y ejecuta migraciones controladas de datos entre bases de datos MySQL/MariaDB de producción y desarrollo para sistemas Laravel, priorizando la conservación íntegra de los datos originales, la trazabilidad y la verificación posterior.
---

# Skill: Migración segura de bases de datos Laravel

## 1. Objetivo

Gestionar migraciones de datos entre una base de datos de producción y una base de datos de desarrollo para aplicaciones Laravel que utilizan MySQL o MariaDB, incluso cuando existan diferencias entre sus esquemas.

La prioridad absoluta es **preservar los datos originales de producción sin alterarlos ni eliminarlos**. La base de producción es la fuente de referencia y debe tratarse como solo lectura. Desarrollo es el destino de la carga y puede requerir adaptación de esquema o transformación de datos, siempre bajo un plan explícito y validado.

La tarea no se considera exitosa solo porque una importación termine sin errores: debe comprobarse la integridad de los datos y emitirse un informe final.

## 2. Principios no negociables

1. **Producción es de solo lectura.** No ejecutar `INSERT`, `UPDATE`, `DELETE`, `REPLACE`, `TRUNCATE`, `ALTER`, `DROP`, `CREATE`, procedimientos con efectos secundarios ni otras operaciones de escritura sobre producción.
2. **Nunca destruir ni sobrescribir datos del destino sin autorización explícita.** Antes de modificar desarrollo, averiguar si contiene información que deba conservarse.
3. No ejecutar operaciones destructivas ni transformaciones potencialmente irreversibles sin aprobación explícita del usuario.
4. No asumir correspondencias entre columnas solo por similitud de nombre. Confirmar la equivalencia semántica mediante esquema, migraciones, código de Laravel, consultas de solo lectura y, si hace falta, preguntas al usuario.
5. Nunca truncar, redondear, recortar, normalizar ni convertir valores silenciosamente.
6. No inventar valores para campos obligatorios, claves foráneas o registros huérfanos. Solicitar una regla de transformación cuando no exista una solución segura y justificada.
7. No omitir filas problemáticas sin registrarlas. Si hay datos incompatibles, detener la etapa afectada, conservar los datos de origen y reportar el motivo.
8. No declarar éxito sin ejecutar y documentar verificaciones posteriores.
9. No mostrar ni registrar contraseñas, tokens, cadenas de conexión completas ni otros secretos.
10. Mantener una separación clara entre **comparación**, **planificación**, **aprobación**, **ejecución** y **verificación**.
11. No crear archivos adicionales, scripts, pruebas o documentación fuera de lo necesario para la migración, salvo que el usuario lo solicite o autorice.
12. Si no es posible verificar la identidad de las conexiones y el entorno, no iniciar escrituras.

## 3. Alcance

La skill trabaja con:

- Laravel y sus migraciones, modelos, relaciones y configuración de base de datos.
- MySQL y MariaDB.
- Esquemas con tablas, columnas, índices, claves primarias, claves únicas, claves foráneas, restricciones y valores predeterminados.
- Datos relacionales que requieren orden de carga.
- Diferencias entre esquemas y datos de producción y desarrollo.

Puede investigar con consultas SQL de solo lectura y revisar el código del proyecto cuando sea necesario.

## 4. Modos de operación

Identificar y comunicar el modo antes de actuar:

### Modo A: Análisis

- Identificar las conexiones y bases de datos de origen y destino.
- Verificar que el origen sea producción y el destino desarrollo.
- Inspeccionar los esquemas y estimar cantidades de registros.
- No modificar nada.

### Modo B: Planificación

- Elaborar una matriz de diferencias.
- Definir correspondencias de columnas, conversiones y orden de carga.
- Identificar riesgos, dependencias, datos ambiguos y posibles conflictos.
- Proponer respaldos y verificaciones.
- No modificar nada.

### Modo C: Simulación

- Ejecutar consultas de diagnóstico y validaciones que no alteren datos.
- Estimar inserciones, actualizaciones necesarias, incompatibilidades y conflictos.
- Si se genera un script o archivo de trabajo, no ejecutarlo automáticamente.
- No modificar nada.

### Modo D: Ejecución aprobada

- Solo iniciar después de mostrar el plan y recibir aprobación explícita del usuario.
- Aplicar exclusivamente las operaciones aprobadas.
- Registrar los resultados de cada etapa.
- Detenerse si aparece una condición no contemplada en el plan.

### Modo E: Verificación

- Comprobar que los datos importados y las relaciones sean correctos.
- Verificar los criterios definidos antes de la ejecución.
- Emitir el informe final, incluyendo errores y tareas pendientes.
- No intentar reparar automáticamente una inconsistencia mediante una operación destructiva.

Si el usuario solicita directamente una migración, comenzar por análisis, planificación y simulación. La solicitud general de migrar no autoriza por sí sola a borrar, sobrescribir ni ejecutar cambios irreversibles.

## 5. Confirmación de conexiones y entorno

Antes de consultar o modificar datos:

1. Identificar las conexiones disponibles sin imprimir secretos.
2. Confirmar cuál es la base de producción y cuál la de desarrollo.
3. Verificar host, nombre de base, usuario y permisos de forma segura, ocultando contraseñas y credenciales.
4. Verificar el entorno configurado por Laravel y las variables de conexión, sin asumir que `APP_ENV=production` o `APP_ENV=local` prueba por sí solo la identidad de la base.
5. Comprobar que las credenciales de producción tengan permisos de solo lectura. Si no se puede garantizar, no realizar ninguna operación de escritura ni ejecutar comandos ambiguos contra esa conexión.
6. No ejecutar `php artisan migrate`, `migrate:fresh`, `migrate:refresh`, `db:wipe`, `schema:drop`, `db:seed` u otros comandos que puedan modificar la base de datos sin determinar con certeza a qué conexión apuntan y obtener la aprobación requerida.
7. Nunca cambiar `.env` ni credenciales para “probar” conexiones sin autorización.
8. Si hay dudas sobre el entorno o las conexiones, detenerse y pedir aclaración.

## 6. Respaldo y recuperación

Antes de modificar desarrollo:

1. Determinar si el destino contiene datos y si deben preservarse.
2. Solicitar o verificar un respaldo consistente de desarrollo y comprobar que el archivo se haya creado correctamente.
3. Cuando sea viable, verificar que el respaldo pueda leerse y que incluya las tablas y objetos requeridos.
4. Registrar ubicación, fecha, tamaño y resultado de la verificación del respaldo, sin exponer secretos.
5. No asumir que disponer de un archivo de respaldo significa que la restauración está probada.
6. No restaurar automáticamente el respaldo ni reemplazar una base completa sin aprobación explícita.

No crear ni guardar respaldos de producción en ubicaciones no aprobadas. Si se necesita un respaldo de producción, realizarlo mediante un procedimiento aprobado y sin modificar la base de origen.

## 7. Inspección del esquema

Comparar producción y desarrollo, como mínimo:

- Tablas y vistas.
- Columnas y orden cuando sea relevante.
- Tipo de dato, longitud, precisión y escala.
- `NULL` / `NOT NULL`.
- Valores predeterminados y expresiones predeterminadas.
- `AUTO_INCREMENT`, claves primarias y claves únicas.
- Índices y claves foráneas.
- Juego de caracteres y collation.
- Restricciones y columnas generadas.
- Triggers, eventos y procedimientos relevantes cuando estén dentro del alcance.
- Tablas pivote, relaciones polimórficas y convenciones usadas por Laravel.

Usar `information_schema` y consultas de solo lectura cuando sea apropiado. Revisar también las migraciones de Laravel, modelos, casts, observers, eventos, factories o seeders relevantes; no ejecutar estos últimos como parte de una migración de datos sin autorización.

No asumir que el esquema de producción es necesariamente el más reciente. Identificar cuál esquema corresponde al código que se pretende desplegar y explicar cualquier contradicción.

## 8. Clasificación de diferencias

Registrar cada diferencia en una matriz con estas columnas:

| Tabla | Elemento | Producción | Desarrollo | Riesgo | Acción propuesta | Estado |
|---|---|---|---|---|---|---|

Clasificar al menos los siguientes casos:

### 8.1 Columnas nuevas en desarrollo

- Identificar si admiten `NULL`, tienen valor predeterminado o requieren un valor calculado.
- Para columnas obligatorias sin valor predeterminado, definir una regla de población validada.
- No inventar valores ni rellenar con una constante arbitraria.

### 8.2 Columnas eliminadas en desarrollo

- No descartar los valores de producción.
- Determinar si la columna fue reemplazada, renombrada o eliminada intencionalmente.
- Si no existe destino equivalente, detener esa parte y solicitar decisión.
- No eliminar columnas ni ejecutar `ALTER TABLE` destructivos automáticamente.

### 8.3 Columnas renombradas

- Confirmar la equivalencia semántica con evidencia del código, migraciones o indicaciones del usuario.
- Mapear explícitamente la columna antigua a la nueva.
- Comprobar que no se mezclen datos si ambas columnas existen y contienen valores distintos.

### 8.4 Cambios de tipo, longitud o precisión

- Evaluar el rango real de los valores de origen y el tipo de destino.
- Detectar cadenas que excedan la longitud, desbordamientos numéricos, pérdida de precisión, fechas inválidas y conversiones ambiguas.
- No truncar, redondear ni cambiar la zona horaria silenciosamente.
- Si no es posible una conversión reversible o sin pérdida, detener esa parte y explicar las alternativas.

### 8.5 Campos que dejan de ser obligatorios o pasan a ser obligatorios

- Detectar valores `NULL`, cadenas vacías y datos que no cumplen la nueva restricción.
- No convertir automáticamente `NULL` a `''`, `0` u otro valor.
- Solicitar una regla de negocio para los datos incompatibles.

### 8.6 Claves primarias, índices y claves foráneas

- Detectar duplicados y referencias huérfanas.
- Verificar que el orden de carga respete las dependencias.
- No desactivar restricciones como solución predeterminada.
- No cambiar identificadores para resolver conflictos sin aprobación y sin analizar todas las referencias relacionadas.

### 8.7 Tablas eliminadas, nuevas o con relaciones modificadas

- Confirmar si deben migrarse, conservarse o excluirse.
- Revisar relaciones uno-a-uno, uno-a-muchos, muchos-a-muchos y polimórficas.
- No eliminar una tabla del destino por el hecho de que ya no exista en el origen.

## 9. Correspondencia y transformación de datos

Crear un mapa explícito por tabla, por ejemplo:

- Columna de origen.
- Columna de destino.
- Regla de transformación.
- Evidencia de equivalencia.
- Riesgo de pérdida.
- Validación requerida.

Las transformaciones deben ser deterministas, documentadas y, cuando sea posible, reversibles.

Si hay ambigüedad sobre el significado de un campo, pedir aclaración antes de continuar. Las reglas de negocio no deben deducirse únicamente por el nombre de una columna.

## 10. Estrategia de carga

Elegir una estrategia según el contenido actual del destino y las necesidades confirmadas. No usar un reemplazo total de la base de datos como opción predeterminada.

Antes de escribir:

1. Mostrar las tablas y filas previstas para insertar, actualizar o excluir.
2. Explicar qué sucede con los registros que ya existen en desarrollo.
3. Definir cómo se resolverán las claves primarias duplicadas y las claves únicas en conflicto.
4. Definir el orden de carga según las dependencias.
5. Definir la estrategia de transacciones y recuperación.
6. Obtener aprobación explícita.

Preferir cargas por lotes y transacciones cuando el motor y la operación lo permitan. Tener presente que ciertas operaciones DDL en MySQL/MariaDB pueden producir commits implícitos y no revertirse como una transacción ordinaria.

No utilizar `INSERT IGNORE`, `REPLACE INTO` ni `ON DUPLICATE KEY UPDATE` indiscriminadamente: pueden ocultar conflictos, descartar filas o sobrescribir datos. Solo se permiten si su comportamiento exacto se ha analizado, está documentado y forma parte del plan aprobado.

No desactivar `FOREIGN_KEY_CHECKS` por defecto. Si una necesidad excepcional exige modificar restricciones, justificarlo, obtener aprobación y verificar explícitamente todas las relaciones después.

No copiar a ciegas tablas operativas, tablas de auditoría, sesiones, colas, cachés, tokens, notificaciones o datos específicos del entorno. Determinar su alcance con el usuario.

## 11. Integridad y validación posterior

Definir antes de ejecutar los criterios de aceptación. Después de cargar, verificar según corresponda:

- Conteo de filas de origen, filas procesadas, insertadas, actualizadas, omitidas y rechazadas.
- Conteo por tabla y, si es útil, por rangos de claves.
- Claves primarias y únicas.
- Integridad referencial y registros huérfanos.
- `NULL`, longitudes máximas y rangos numéricos.
- Valores de columnas críticas.
- Sumas de control o hashes de conjuntos de datos cuando sean viables.
- Muestras comparadas de origen y destino.
- Consistencia de fechas, decimales, caracteres, codificación y collation.
- Compatibilidad con el esquema y el código de Laravel previsto.

Los conteos iguales no prueban por sí solos que los datos sean idénticos. Elegir verificaciones que detecten cambios de contenido y documentar las limitaciones.

No escribir ni actualizar datos de producción para realizar una verificación. Si una validación falla, detener el proceso, informar el alcance y proponer una recuperación segura. No afirmar que los datos de desarrollo son idénticos a producción si hubo exclusiones o transformaciones; detallarlas.

## 12. Protección especial de producción

La base de producción no debe recibir datos desde desarrollo ni ser modificada por esta skill.

Si el objetivo final es desplegar cambios de esquema o código en producción después de preparar desarrollo:

1. Tratarlo como una operación distinta de la carga de datos.
2. Inspeccionar las migraciones de Laravel y evaluar compatibilidad hacia atrás.
3. Identificar los cambios destructivos, bloqueos, tiempos estimados y riesgos.
4. Proponer un plan de despliegue y recuperación.
5. Obtener aprobación explícita para cada operación de escritura en producción.
6. No ejecutar el despliegue como consecuencia automática de haber terminado la migración de desarrollo.

El usuario debe comprender que preparar desarrollo con datos de producción no significa que sea seguro reemplazar producción con la base de desarrollo: esta última podría contener datos más antiguos, datos locales o registros generados durante las pruebas.

## 13. Consultas SQL y herramientas

Se permite ejecutar consultas de diagnóstico de solo lectura para resolver dudas, como consultas a `information_schema`, conteos, búsquedas de duplicados y detección de registros huérfanos.

Antes de ejecutar una consulta:

- Confirmar a qué conexión se dirige.
- Preferir consultas acotadas y eficientes.
- Evitar consultas costosas sobre producción en horas de alta carga.
- No exponer datos personales o secretos en la salida si no son necesarios.
- Usar `EXPLAIN` u otros mecanismos de análisis cuando resulte útil y seguro.

No ejecutar SQL dinámico, scripts de terceros o comandos del sistema sin revisar su contenido y sus posibles efectos. No enviar datos de producción a servicios externos ni a modelos remotos salvo que el usuario lo autorice y exista una base legítima para hacerlo; por defecto, limitar la inspección a metadatos y muestras mínimas necesarias.

## 14. Manejo de errores y detención segura

Detener la ejecución y explicar la situación cuando:

- No se pueda confirmar cuál conexión es producción o desarrollo.
- No exista respaldo verificable del destino cuando sea necesario.
- Se detecte una conversión con posible pérdida.
- Existan conflictos de claves o relaciones no resueltos.
- La cantidad de registros difiera inesperadamente.
- Aparezca una tabla o columna no contemplada en el plan.
- La ejecución requiera una operación destructiva no aprobada.
- La conexión de origen permita escritura y no se pueda garantizar que el proceso no la utilice.
- Se pierda la conexión o el resultado de una operación quede indeterminado.

No reintentar a ciegas operaciones de escritura cuyo resultado sea incierto. Primero consultar el estado real del destino y determinar qué se completó.

## 15. Informe final obligatorio

Al finalizar cada ejecución, presentar un informe en español que incluya:

### Resumen

- Fecha y hora de ejecución.
- Base de origen y base de destino, sin contraseñas ni secretos.
- Entornos identificados.
- Modo de operación.
- Resultado general: **completada**, **completada con incidencias**, **detenida** o **fallida**.

### Tablas afectadas

| Tabla | Acción | Filas de origen | Insertadas | Actualizadas | Omitidas | Errores | Validación |
|---|---|---:|---:|---:|---:|---:|---|

Si una métrica no pudo obtenerse, indicarlo como “no verificada”; no inventar cifras.

### Cambios realizados

Listar cada cambio de esquema o datos, incluyendo:

- Tabla y columna.
- Estructura anterior y nueva.
- Correspondencia o transformación aplicada.
- Motivo y aprobación correspondiente.
- Número de registros afectados.
- Si la operación es reversible y cómo.

### Validaciones

- Conteos y comparaciones ejecutadas.
- Integridad de claves y relaciones.
- Conflictos detectados y su resolución.
- Diferencias pendientes.
- Limitaciones de las verificaciones.

### Respaldo y recuperación

- Respaldo utilizado o generado, si aplica.
- Estado de su verificación.
- Procedimiento de recuperación recomendado.
- Cualquier operación que requiera intervención manual.

### Confirmación de protección

Indicar expresamente si producción permaneció sin modificaciones por parte del proceso. Si no puede comprobarse, declararlo como “no verificado”; nunca afirmarlo sin evidencia.

## 16. Flujo de trabajo obligatorio

Seguir este orden en cada migración:

1. Confirmar el objetivo y los entornos.
2. Verificar las conexiones y la protección de producción.
3. Analizar los esquemas y los datos necesarios.
4. Generar la matriz de diferencias.
5. Definir correspondencias y reglas de transformación.
6. Identificar riesgos y preparar el respaldo de desarrollo.
7. Ejecutar la simulación y preparar el plan de carga.
8. Presentar el plan, el impacto esperado y las acciones destructivas o irreversibles.
9. Esperar aprobación explícita.
10. Ejecutar únicamente las acciones aprobadas.
11. Validar los resultados y detenerse ante cualquier discrepancia no prevista.
12. Emitir el informe final obligatorio.

## 17. Criterio de éxito

Una migración solo puede declararse **completada** cuando:

- Las conexiones y los entornos fueron identificados correctamente.
- Se ejecutó únicamente el plan aprobado.
- Las transformaciones están documentadas.
- Las filas procesadas y los conflictos tienen trazabilidad.
- Las validaciones acordadas fueron satisfactorias.
- Las incidencias pendientes están declaradas.
- La protección de producción se mantuvo o su estado se reportó honestamente.
- Se generó el informe final con las tablas afectadas y los cambios realizados.

Ante cualquier duda, priorizar la conservación de los datos y detenerse antes que realizar una operación potencialmente destructiva.
