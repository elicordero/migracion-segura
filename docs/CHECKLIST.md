# Listas de verificación

Marca cada casilla con evidencia. Una casilla sin evidencia es una casilla no cumplida.

## 0. Preparación

- [ ] Proyecto Laravel identificado y versión registrada.
- [ ] Motor y versión de MySQL/MariaDB registrados.
- [ ] Conexión de origen identificada por nombre.
- [ ] Conexión de destino identificada por nombre.
- [ ] Producción confirmada como solo lectura.
- [ ] Permisos efectivos revisados con `SHOW GRANTS`.
- [ ] Destino confirmado como desarrollo, no réplica de producción.
- [ ] Ventana de trabajo acordada.
- [ ] Responsable de aprobación identificado.
- [ ] Ubicación de respaldos aprobada.
- [ ] Secretos fuera del chat y del repositorio.

## 1. Análisis

- [ ] Tablas y vistas inventariadas en origen y destino.
- [ ] Volumen estimado por tabla.
- [ ] Columnas, tipos, longitudes, precisión y escala comparadas.
- [ ] `NULL` / `NOT NULL` y valores predeterminados comparados.
- [ ] Claves primarias, únicas, índices y foráneas comparadas.
- [ ] Charset y collation comparados.
- [ ] Columnas generadas, triggers y eventos revisados si aplican.
- [ ] Migraciones, modelos, casts y relaciones de Laravel revisadas.
- [ ] Esquema correspondiente al código a desplegar identificado.
- [ ] Tablas operativas y sensibles marcadas para exclusión.
- [ ] Datos locales de desarrollo identificados.
- [ ] Ninguna escritura ejecutada.

## 2. Planificación

- [ ] Matriz de diferencias completa.
- [ ] Correspondencia de columnas documentada.
- [ ] Evidencia de equivalencia registrada para cada renombre.
- [ ] Transformaciones definidas y deterministas.
- [ ] Columnas nuevas obligatorias con regla aprobada.
- [ ] Columnas eliminadas con decisión registrada.
- [ ] Riesgos de truncamiento, redondeo y zona horaria evaluados.
- [ ] Duplicados y huérfanos analizados.
- [ ] Orden de carga por dependencias definido.
- [ ] Política de conflictos de claves definida.
- [ ] Estrategia de transacciones y recuperación definida.
- [ ] Criterios de aceptación definidos antes de ejecutar.
- [ ] Exclusiones y transformaciones que impiden afirmar igualdad total declaradas.

## 3. Simulación

- [ ] Filas a insertar estimadas por tabla.
- [ ] Filas a actualizar estimadas por tabla.
- [ ] Filas a omitir y motivo.
- [ ] Filas rechazadas y motivo.
- [ ] Conflictos de claves primarias y únicas detectados.
- [ ] Referencias huérfanas detectadas.
- [ ] Valores fuera de rango o longitud detectados.
- [ ] Consultas acotadas y eficientes.
- [ ] Sin consultas costosas en horas de alta carga.
- [ ] Scripts de trabajo revisados y no ejecutados automáticamente.
- [ ] Ninguna escritura ejecutada.

## 4. Respaldo de desarrollo

- [ ] Necesidad de preservar datos del destino confirmada.
- [ ] Respaldo generado o localizado.
- [ ] Archivo existente y con tamaño razonable.
- [ ] Base y fecha correctas.
- [ ] Restauración de prueba en base temporal, si es viable.
- [ ] Ubicación, fecha, tamaño y resultado registrados.
- [ ] Procedimiento de recuperación acordado.
- [ ] Respaldo no versionado ni expuesto.

## 5. Aprobación

- [ ] Plan presentado completo.
- [ ] Impacto esperado presentado.
- [ ] Operaciones destructivas o irreversibles señaladas.
- [ ] Aprobación explícita recibida.
- [ ] Tablas incluidas nombradas.
- [ ] Operaciones permitidas nombradas.
- [ ] Operaciones prohibidas nombradas.
- [ ] Alcance de la aprobación registrado con fecha.
- [ ] Si cambió el plan, nueva aprobación obtenida.

No continuar si la aprobación es ambigua, anterior al plan o no menciona el alcance destructivo.

## 6. Ejecución

- [ ] Solo se ejecutan acciones aprobadas.
- [ ] Conexión de destino confirmada antes de cada lote.
- [ ] Producción no recibe escrituras.
- [ ] Lotes y transacciones según el plan.
- [ ] DDL con commits implícitos tratado como operación especial.
- [ ] `FOREIGN_KEY_CHECKS` no desactivado sin aprobación.
- [ ] Sin `INSERT IGNORE`, `REPLACE INTO` ni `ON DUPLICATE KEY UPDATE` fuera del plan.
- [ ] Resultados por etapa registrados.
- [ ] Detención ante condición no prevista.
- [ ] Sin reintentos ciegos tras resultado incierto.

## 7. Verificación

- [ ] Conteo de filas de origen registrado.
- [ ] Conteo de procesadas, insertadas, actualizadas, omitidas y rechazadas.
- [ ] Conteo por tabla y por rango de claves cuando aplica.
- [ ] Duplicados de claves primarias y únicas: cero o justificados.
- [ ] Huérfanos: cero o justificados.
- [ ] `NULL`, longitudes máximas y rangos numéricos revisados.
- [ ] Columnas críticas comparadas.
- [ ] Sumas de control o hashes cuando son viables.
- [ ] Muestras de origen y destino comparadas.
- [ ] Fechas, decimales, caracteres y collation revisados.
- [ ] Compatibilidad con esquema y código Laravel previsto.
- [ ] Limitaciones de las verificaciones documentadas.
- [ ] Ninguna validación escribió en producción.

## 8. Cierre

- [ ] Informe final emitido.
- [ ] Fecha y hora registradas.
- [ ] Origen y destino identificados sin secretos.
- [ ] Modo de operación registrado.
- [ ] Resultado general declarado.
- [ ] Tablas afectadas con métricas completas o "no verificada".
- [ ] Cambios de esquema y datos listados con aprobación.
- [ ] Reversibilidad indicada.
- [ ] Respaldo y recuperación documentados.
- [ ] Incidencias y pendientes declarados.
- [ ] Protección de producción confirmada o marcada como "no verificada".
- [ ] Aprobaciones y evidencia archivadas en la ubicación del equipo.

## 9. Detención de emergencia

Activar cuando:

- [ ] No se puede confirmar la identidad de las conexiones.
- [ ] Producción permite escritura y no se puede garantizar su uso.
- [ ] Falta respaldo verificable del destino.
- [ ] Se detecta pérdida de datos.
- [ ] Hay conflictos de claves o relaciones sin resolver.
- [ ] El volumen difiere inesperadamente.
- [ ] Aparece una tabla o columna fuera del plan.
- [ ] Se requiere una operación destructiva no aprobada.
- [ ] Se pierde la conexión o el resultado es indeterminado.

Pasos:

- [ ] Detener nuevas escrituras.
- [ ] No reintentar a ciegas.
- [ ] Consultar el estado real del destino.
- [ ] Registrar qué se completó y qué no.
- [ ] Informar el alcance.
- [ ] Proponer recuperación segura.
- [ ] Esperar instrucciones.

## 10. No declarar éxito si

- [ ] No se identificaron las conexiones con certeza.
- [ ] Se ejecutó algo fuera del plan aprobado.
- [ ] Hay transformaciones sin documentar.
- [ ] Hay filas omitidas sin registro.
- [ ] Las validaciones acordadas no se ejecutaron.
- [ ] Hay incidencias pendientes sin declarar.
- [ ] No se puede afirmar el estado de producción.
- [ ] No existe informe final.
