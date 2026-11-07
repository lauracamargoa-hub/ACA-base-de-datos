# Proyecto ACA – Administración de Bases de Datos con Oracle

## Descripción del proyecto

Este proyecto implementa una estrategia de administración de bases de datos (DBA) para una tienda comercial que gestiona información de clientes, empleados, ventas, productos e inventario.

La solución aplica prácticas de administración, seguridad, respaldo, recuperación, optimización, automatización y monitoreo sobre Oracle Database 21c Express Edition.

El proyecto parte de un diagnóstico inicial que identifica problemas de disponibilidad, ausencia de respaldos registrados, permisos sin separar por función y crecimiento de los movimientos de inventario.

A partir de estos hallazgos, se implementan mecanismos para fortalecer la recuperación de la información, controlar el acceso a los datos y automatizar tareas de mantenimiento.

## Objetivos

### Objetivo general

Implementar una estrategia de administración de bases de datos utilizando Oracle, aplicando mecanismos de respaldo y recuperación, seguridad, optimización, automatización y monitoreo.

### Objetivos específicos

- Diagnosticar el estado inicial de la base de datos.
- Configurar el modo ARCHIVELOG y el área de recuperación rápida (FRA).
- Implementar respaldos y recuperación mediante RMAN.
- Administrar usuarios, roles y permisos bajo el principio de mínimo privilegio.
- Registrar acciones relevantes mediante auditoría.
- Optimizar consultas mediante índices.
- Automatizar la depuración de movimientos antiguos de inventario.
- Monitorear el estado de la base de datos y sus recursos.

## Tecnologías utilizadas

| Tecnología | Uso |
|---|---|
| Oracle Database 21c Express Edition | Sistema gestor de base de datos |
| SQL*Plus | Ejecución de scripts y comandos administrativos |
| Oracle SQL Developer | Consulta y demostración de los objetos de la base de datos |
| RMAN | Respaldo, validación y recuperación |
| PL/SQL | Procedimientos almacenados y automatización |
| DBMS_SCHEDULER | Programación de tareas dentro de Oracle |
| Windows 11 | Sistema operativo del entorno de implementación |
| Programador de tareas de Windows | Programación de scripts del sistema operativo |

## Entorno de implementación

El proyecto se ejecuta en Windows 11 con Oracle Database 21c Express Edition.

| Elemento | Configuración |
|---|---|
| Versión de Oracle | 21.3.0.0.0 |
| Instancia | XE |
| Base de datos enchufable (PDB) | XEPDB1 |
| Usuario propietario de las tablas | TIENDA_APP |
| Tablespace de la aplicación | TS_TIENDA |
| Listener | localhost:1521 |
| Área de recuperación rápida | FRA |
| Tamaño configurado de la FRA | 5 GB |
| Carpeta de trabajo | C:\Proyecto_Oracle\ |

La tienda se encuentra en la PDB `XEPDB1`. El usuario `TIENDA_APP` es propietario de las nueve tablas del proyecto y utiliza el tablespace `TS_TIENDA`.

## Diagnóstico inicial

El diagnóstico se realizó mediante el script `03_diagnostico_recursos.sql`, ejecutado con privilegios SYSDBA.

Se identificaron las siguientes condiciones iniciales:

- La base de datos se encontraba en modo `NOARCHIVELOG`.
- No había respaldos registrados en el control file.
- El usuario `TIENDA_APP` tenía privilegios de creación, pero no contaba con roles separados por función.
- El tablespace `TS_TIENDA` ocupaba 6 MB de un tamaño inicial de 100 MB.
- La tabla `MOVIMIENTOS_INVENTARIO` era el segmento más grande de la aplicación.
- La tabla de movimientos contenía 50.000 registros.

Este diagnóstico permitió establecer las necesidades de respaldo, recuperación, seguridad y mantenimiento de inventario.

## Modelo de datos

La base de datos representa una tienda comercial que administra clientes, empleados, productos, pedidos y movimientos de inventario.

El modelo contiene nueve tablas: siete relacionadas con los procesos del negocio y dos destinadas al registro de operaciones.

### Tablas del proyecto

- **CLIENTES:** información de los clientes.
- **EMPLEADOS:** información de los empleados.
- **CATEGORIAS:** clasificación de los productos.
- **PRODUCTOS:** productos disponibles en la tienda.
- **PEDIDOS:** pedidos realizados por los clientes.
- **DETALLE_PEDIDO:** productos y cantidades asociados a cada pedido.
- **MOVIMIENTOS_INVENTARIO:** registro de movimientos de inventario.
- **AUDITORIA_PERMISOS:** registro de acciones relacionadas con permisos y auditoría.
- **LOG_DEPURACION_INVENTARIO:** registro de las ejecuciones de depuración del inventario.

El modelo cuenta con llaves primarias (PK) y llaves foráneas (FK), que permiten establecer las relaciones entre las entidades y mantener la integridad referencial de la información.

### Diagrama entidad-relación

El siguiente diagrama representa el modelo de datos del proyecto, mostrando las tablas, sus llaves primarias, llaves foráneas y relaciones.

![Diagrama entidad-relación](diagramas/diagrama_entidad_relacion.png)

La imagen se encuentra dentro de la carpeta `diagramas` del proyecto y debe entregarse junto con este README para que pueda visualizarse correctamente.

## Configuración de almacenamiento

### Tablespace TS_TIENDA

Se creó un tablespace específico para almacenar los objetos de la aplicación.

| Parámetro | Valor |
|---|---|
| Nombre | TS_TIENDA |
| Ubicación | XEPDB1 |
| Tamaño inicial | 100 MB |
| Crecimiento automático | 10 MB |
| Tamaño máximo | 500 MB |
| Propietario de los datos | TIENDA_APP |

El archivo de datos se encuentra en el directorio de Oracle correspondiente a la PDB `XEPDB1`.

### Modo ARCHIVELOG

Se habilitó el modo `ARCHIVELOG`, permitiendo conservar los registros de cambios necesarios para realizar recuperación de la base de datos.

La configuración se realizó mediante el script `04_archivelog_y_fra.sql`.

### Área de recuperación rápida (FRA)

Se configuró el área de recuperación rápida con un límite de 5 GB en la ruta:

```text
C:\Proyecto_Oracle\respaldo\fra
```

La FRA almacena respaldos y archivos de registro necesarios para las operaciones de recuperación.

La configuración permite administrar el espacio destinado a recuperación y consultar su utilización mediante el tablero de monitoreo.

## Funcionalidades implementadas

### 1. Seguridad y auditoría

La seguridad se implementa mediante roles y usuarios separados por función, aplicando el principio de mínimo privilegio.

#### Roles y usuarios

| Rol | Usuario | Función |
|---|---|---|
| ROL_VENTAS | USR_VENTAS | Consultar y modificar información relacionada con clientes y pedidos. |
| ROL_INVENTARIO | USR_INVENTARIO | Registrar movimientos y ajustar productos. |
| ROL_AUDITOR | USR_AUDITOR | Consultar información de auditoría y registros del sistema. |

Los tres usuarios utilizan el perfil `PERFIL_TIENDA`, que establece:

- Bloqueo de cuenta después de cinco intentos fallidos durante una hora.
- Cambio de contraseña cada 90 días.
- Máximo de tres sesiones simultáneas.

#### Control de permisos

Se implementan operaciones `GRANT` y `REVOKE` para administrar los permisos de los roles.

Se valida que los usuarios no puedan ejecutar operaciones que no corresponden a su función.

Por ejemplo:

- `USR_VENTAS` puede consultar y actualizar clientes, pero no puede eliminarlos.
- `USR_VENTAS` no puede consultar movimientos de inventario.
- `USR_INVENTARIO` puede registrar movimientos y actualizar productos, pero no puede consultar clientes.
- `USR_AUDITOR` cuenta con permisos de lectura sobre las tablas autorizadas.

#### Auditoría

La auditoría unificada de Oracle registra las acciones relevantes.

Los registros se copian a la tabla `TIENDA_APP.AUDITORIA_PERMISOS`, incluyendo información sobre el usuario, la acción, el objeto y el resultado de la operación.

Se comprobó que un intento no autorizado de eliminación genera el error `ORA-01031` y queda registrado en la auditoría.

### 2. Respaldo y recuperación mediante RMAN

Se implementó una estrategia de respaldo completo mediante Recovery Manager (RMAN).

La configuración incluye:

- Política de retención de siete días.
- Respaldo automático del control file.
- Respaldo completo de la base de datos junto con los archivelogs.
- Validación del respaldo mediante `RESTORE DATABASE VALIDATE`.

El respaldo completo registrado ocupó aproximadamente 2.706 MB y se completó en ocho segundos.

La validación del respaldo terminó sin errores.

#### Recuperación ante una falla simulada

Se realizó una prueba controlada de recuperación del tablespace `TS_TIENDA`.

La prueba consistió en:

1. Crear un respaldo completo de la base de datos.
2. Insertar un cliente después del respaldo.
3. Forzar el archivado de los registros de cambios.
4. Simular la pérdida del archivo de datos de `TS_TIENDA`.
5. Restaurar el tablespace mediante RMAN.
6. Aplicar la recuperación utilizando los archivelogs.
7. Verificar la disponibilidad del tablespace y los datos recuperados.

La recuperación se realizó mediante los comandos:

```sql
RESTORE TABLESPACE XEPDB1:TS_TIENDA;
RECOVER TABLESPACE XEPDB1:TS_TIENDA;
```

#### Resultados de recuperación

| Indicador | Resultado |
|---|---|
| Tiempo de recuperación (RTO) | Aproximadamente 3 minutos y 20 segundos |
| Pérdida de datos en la prueba (RPO) | 0 |
| Estado final de TS_TIENDA | ONLINE |
| Clientes después de la recuperación | 21 |
| Movimientos de inventario después de la recuperación | 50.000 |

El cliente `Cliente post-respaldo`, insertado después del respaldo inicial, se recuperó correctamente gracias a los archivelogs.

La prueba demuestra que la recuperación permitió restablecer la información del tablespace, incluyendo cambios posteriores al respaldo.

### 3. Optimización de consultas

Se optimizaron consultas sobre la tabla `MOVIMIENTOS_INVENTARIO`, que inicialmente contenía 50.000 registros.

Se creó el índice:

```sql
CREATE INDEX ix_mov_fecha_prod
ON movimientos_inventario (fecha_movimiento, id_producto);
```

Se compararon los planes de ejecución y las lecturas lógicas antes y después de la creación del índice.

#### Comparación de resultados

| Consulta | Antes | Después | Mejora |
|---|---:|---:|---:|
| Consulta A: movimientos de un producto en los últimos 30 días | 373 buffers | 40 buffers | 89 % |
| Consulta B: movimientos de los últimos 30 días | 373 buffers | 6 buffers | 98 % |

Antes de la optimización, las consultas utilizaban `TABLE ACCESS FULL`.

Después de crear el índice, Oracle utilizó operaciones como `INDEX RANGE SCAN`, reduciendo las lecturas lógicas.

Los tiempos de ejecución fueron similares debido al tamaño de la base de datos utilizada en la prueba. La mejora se evidencia principalmente en la reducción de lecturas.

### 4. Automatización de la depuración de inventario

Se implementó el procedimiento almacenado:

```text
TIENDA_APP.SP_DEPURAR_MOVIMIENTOS
```

El procedimiento elimina movimientos de inventario con más de 12 meses de antigüedad, registra la cantidad de filas eliminadas y conserva el resultado de la ejecución.

La información de cada ejecución se almacena en `LOG_DEPURACION_INVENTARIO`.

El procedimiento registra:

- Cantidad de registros eliminados.
- Estado de la ejecución.
- Mensaje descriptivo.
- Información del error cuando ocurre una falla.

#### Trabajo programado

Se creó el trabajo:

```text
TIENDA_APP.JOB_DEPURAR_MOVIMIENTOS
```

El trabajo se programa mediante `DBMS_SCHEDULER` para ejecutarse el primer día de cada mes a las 2:00 a. m.

#### Resultados de la depuración

| Indicador | Resultado |
|---|---:|
| Registros iniciales | 50.000 |
| Registros con más de 12 meses | 33.417 |
| Registros restantes | 16.583 |
| Registros eliminados | 33.417 |
| Estado registrado | OK |

La ejecución quedó registrada en el log con el estado `OK`.

### 5. Monitoreo de la base de datos

Se implementó un tablero de monitoreo mediante el script `13_monitoreo_v2.sql`.

El tablero consulta vistas del diccionario de Oracle para mostrar información relacionada con:

- Uso del área de recuperación rápida.
- Historial de respaldos RMAN.
- Archivelogs generados.
- Espacio utilizado por tablespace.
- Segmentos de mayor tamaño.
- Sesiones de usuario.
- Consultas con mayor número de lecturas lógicas.
- Trabajos programados y su último resultado.
- Usuarios de laboratorio.

#### Umbrales de alerta

| Control | Umbral |
|---|---|
| Uso de FRA | Mayor al 80 % |
| Uso de tablespace | Mayor al 80 % |
| Tiempo desde el último respaldo | Mayor a 24 horas |

#### Estado del monitoreo

Durante la ejecución del 29 de octubre de 2026 se obtuvieron los siguientes resultados:

| Indicador | Resultado |
|---|---:|
| Uso de FRA | 54,9 % |
| Espacio utilizado en FRA | 2.812 MB |
| Límite de FRA | 5.120 MB |
| Uso de TS_TIENDA | 1,7 % |
| Estado del trabajo de depuración | SUCCEEDED |

Los controles reportaron estado `OK`.

### 6. Respaldo comprimido programado

Se dispone de un script RMAN para realizar respaldos comprimidos de la base de datos y sus archivelogs.

El script incluye:

- Configuración de respaldos comprimidos.
- Respaldo completo de la base de datos.
- Respaldo de archivelogs.
- Eliminación de respaldos obsoletos según la política configurada.
- Listado del resumen de respaldos.

El archivo de comandos RMAN es:

```text
14_respaldo_diario.txt
```

El lanzador para Windows es:

```text
respaldo_diario.bat
```

El lanzador genera un archivo de log en:

```text
C:\Proyecto_Oracle\evidencias\14_respaldo_diario.log
```

Estos archivos permiten ejecutar el respaldo desde Windows y utilizarlo como parte de una estrategia de respaldo periódico.

## Estructura del proyecto

La organización de los archivos de trabajo es la siguiente:

```text
Proyecto_ACA/
│
├── README.md
│
├── diagramas/
│   └── diagrama_entidad_relacion.png
│
├── sql/
│   ├── 00_entorno.sql
│   ├── 01_modelo.sql
│   ├── 02_datos.sql
│   ├── 03_diagnostico_recursos.sql
│   ├── 04_archivelog_y_fra.sql
│   ├── 05_respaldo_rman.txt
│   ├── 06_preparar_falla.sql
│   ├── 07_simular_falla.sql
│   ├── 08_recuperacion_rman.txt
│   ├── 09_verificar_recuperacion.sql
│   ├── 10_seguridad_auditoria.sql
│   ├── 11_optimizacion.sql
│   ├── 12_automatizacion_v2.sql
│   ├── 13_monitoreo_v2.sql
│   ├── 14_respaldo_diario.txt
│   └── respaldo_diario.bat
│
├── evidencias/
│   ├── 03_diagnostico_antes.txt
│   ├── 04_archivelog.txt
│   ├── 05_respaldo_rman.log
│   ├── 06_preparar_falla.txt
│   ├── 07_simular_falla.txt
│   ├── 08_recuperacion_rman.log
│   ├── 09_verificar_recuperacion.txt
│   ├── 10_seguridad_auditoria.txt
│   ├── 11_optimizacion.txt
│   ├── 12_automatizacion.txt
│   └── 13_monitoreo.txt
│
├── documentacion/
└── respaldo/
    └── fra/
```

Los nombres de archivos deben corresponder con los scripts y evidencias incluidos en la entrega.

## Instalación y ejecución

### Requisitos

- Windows 11.
- Oracle Database 21c Express Edition.
- SQL*Plus.
- Oracle SQL Developer.
- Acceso administrativo a la base de datos.
- Scripts del proyecto.

### Ejecución de los scripts

1. Iniciar SQL*Plus con privilegios administrativos.
2. Conectarse como SYSDBA.
3. Ejecutar los scripts de configuración y creación de objetos en el orden establecido.
4. Cargar los datos de prueba.
5. Ejecutar los scripts de diagnóstico, configuración, seguridad, optimización, automatización y monitoreo.
6. Ejecutar los comandos RMAN desde los archivos de comandos correspondientes.
7. Consultar las evidencias generadas para revisar los resultados.

Ejemplo de ejecución:

```sql
@C:\Proyecto_Oracle\sql\03_diagnostico_recursos.sql
```

La ruta debe ajustarse a la ubicación real de la carpeta del proyecto.

Los comandos RMAN se ejecutan desde SQL*Plus mediante `HOST` o utilizando el lanzador de Windows, según el script correspondiente.

### Consideraciones de ejecución

- Ejecutar los scripts en el orden establecido.
- No repetir indiscriminadamente los scripts de carga de datos, ya que pueden duplicar registros.
- Los scripts de simulación de falla y depuración realizan operaciones sobre los datos y archivos de la base de datos; deben ejecutarse siguiendo las instrucciones incluidas.
- Las rutas deben corresponder a la ubicación real de Oracle y de los archivos del proyecto.
- No incluir contraseñas en el README ni en otros documentos compartidos.
- La simulación de pérdida del archivo de datos debe realizarse únicamente en el entorno de laboratorio y siguiendo el procedimiento de respaldo indicado.

## Evidencias del proyecto

Las evidencias técnicas se encuentran organizadas por fase y permiten consultar los resultados de las operaciones implementadas.

| Fase | Archivo de evidencia |
|---|---|
| Diagnóstico inicial | `03_diagnostico_antes.txt` |
| Configuración ARCHIVELOG y FRA | `04_archivelog.txt` |
| Respaldo RMAN | `05_respaldo_rman.log` |
| Preparación de la falla | `06_preparar_falla.txt` |
| Simulación de falla | `07_simular_falla.txt` |
| Recuperación RMAN | `08_recuperacion_rman.log` |
| Verificación de recuperación | `09_verificar_recuperacion.txt` |
| Seguridad y auditoría | `10_seguridad_auditoria.txt` |
| Optimización | `11_optimizacion.txt` |
| Depuración de inventario | `12_automatizacion.txt` |
| Monitoreo | `13_monitoreo.txt` |

## Resultados obtenidos

Los resultados evidencian mejoras en la recuperación, seguridad, optimización y mantenimiento de la base de datos.

| Indicador | Resultado |
|---|---|
| Modo de base de datos | ARCHIVELOG |
| Respaldo completo RMAN | Validado sin errores |
| Tiempo de recuperación (RTO) | Aproximadamente 3 min 20 s |
| Pérdida de datos en la prueba (RPO) | 0 |
| Movimientos iniciales | 50.000 |
| Movimientos después de la depuración | 16.583 |
| Movimientos depurados | 33.417 |
| Lecturas lógicas de consulta A | De 373 a 40 |
| Lecturas lógicas de consulta B | De 373 a 6 |
| Uso de FRA reportado | 54,9 % |
| Uso de TS_TIENDA reportado | 1,7 % |

La prueba de recuperación permitió restablecer el tablespace y recuperar un registro insertado después del respaldo inicial.

La optimización mediante índices redujo las lecturas lógicas de las consultas evaluadas.

La depuración automatizada eliminó movimientos antiguos y registró el resultado de la ejecución.

## Conclusiones

El proyecto permite aplicar prácticas fundamentales de administración de bases de datos en un entorno Oracle.

La configuración de ARCHIVELOG y FRA, junto con los respaldos y la recuperación mediante RMAN, fortalece la disponibilidad e integridad de la información.

La gestión de roles, perfiles y auditoría mejora el control de acceso y permite registrar operaciones no autorizadas.

La optimización mediante índices reduce las lecturas lógicas de las consultas evaluadas, mientras que la depuración automatizada contribuye al mantenimiento de la tabla de movimientos de inventario.

Finalmente, el tablero de monitoreo facilita la revisión del estado de los recursos y permite identificar condiciones relevantes para la administración de la base de datos.

## Autores

- [Johan Esteban Petro Trejos]
- [Edwin Alexis Paez]
- [Derly Dayana Romero Beltran]
- [Nely Daniela Araque Uzcategui]
- [Laura Cristina Camargo Acevedo]
