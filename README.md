📌 Proyecto: Pipeline de Ingesta y Validación de Datos – AWS Lambda + SQL Server
📖 Descripción
Este proyecto implementa un pipeline serverless para procesar múltiples archivos de entrada (TXT) desde Amazon S3, validar su calidad y cargar la información en tablas de la capa Silver de un Data Warehouse en SQL Server.
La solución está basada en una Lambda Function en Python que se diseñó para ser reutilizable y escalable, permitiendo agregar nuevos layouts de archivos sin modificar la lógica central.

🏗 Arquitectura
La arquitectura utilizada incluye los siguientes servicios:
AWS Lambda → Procesa los archivos desde S3 y ejecuta las cargas hacia SQL Server.
Amazon S3 → Almacena los archivos de entrada (FILE_CAPTACION.txt, CLIENTES_POR_PRODUCTO.txt, RECO.txt, ENT_CAJA_CAPTACION.txt, SISPAGOS_IO.txt, ENT_TASAS_PASIVAS.txt, entre otros).
AWS SNS → Notificaciones centralizadas de errores o éxito del procesamiento.
AWS CloudWatch → Logs de ejecución y monitoreo de la Lambda.
SQL Server (Capa Silver) → Almacén de datos de destino.
Tabla de Calidad → Registro de validaciones de integridad y calidad de cada carga.
Bucket de Control → Registro de archivos procesados para evitar duplicidades.

🔄 Flujo del Proceso
Carga del archivo en el bucket S3 correspondiente.
Trigger de AWS Lambda que detecta el nuevo archivo.
Identificación del tipo de archivo por nombre (NOMBRE_ARCHIVO) y selección de layout mapeado.

Validación de Calidad:
Coincidencia de columnas con el layout esperado.
Validación de tipos de datos.
Validación de campos obligatorios.
Inserción en tabla destino en SQL Server.
Registro en Tabla de Calidad con fecha, resultado y detalles de errores (si los hay).
Notificación vía SNS del estado del procesamiento.
Registro en tabla LOG_ARCHIVOS_PROCESADOS_LAMBDA para control histórico.

📂 Archivos Procesados
FILE_CAPTACION.txt → Tabla RR.[110_ENT_CAPTACION]
CLIENTES_POR_PRODUCTO.txt → Tabla RR.[007_ENT_CLIENTES_PRODUCTO]
RECO.txt → Tabla RR.[106_RECO]
ENT_CAJA_CAPTACION.txt → Tabla RR.[109_ENT_CAJA_CAPTACION]
SISPAGOS_IO.txt → Tabla RR.[105_SISPAGOS_IO]
ENT_TASAS_PASIVAS.txt → Tabla RR.[102_ENT_TASAS_PASIVAS]

🛠 Tecnologías Usadas
Python 3.9
boto3 → Interacción con AWS S3 y SNS.
pyodbc → Conexión a SQL Server.
pandas → Procesamiento y validación de datos.
numpy → Manipulación numérica y validaciones.
AWS Lambda con runtime Python.
AWS S3 para almacenamiento.
AWS SNS para notificaciones.

📊 Consultas SQL de Validación
Contar registros insertados

SELECT COUNT(*) AS total_registros
FROM RR.[007_ENT_CLIENTES_PRODUCTO]
WHERE FECHA_PROCESO = '2025-08-09';

SELECT TOP 10 *
FROM RR.[007_ENT_CLIENTES_PRODUCTO]
WHERE FECHA_PROCESO = '2025-08-09';

Logs de calidad
SELECT *
FROM RR.LOG_CALIDAD_DATOS_LAMBDA
WHERE FECHA_PROCESO = '2025-08-09'
  AND NOMBRE_ARCHIVO = 'CLIENTES_POR_PRODUCTO.txt';

📌 Consideraciones
La Lambda fue reciclada desde una implementación previa en producción.
Se adaptó para manejar múltiples layouts y tablas sin reescribir código central.
Todas las inserciones son controladas por validaciones de calidad previas.
Es posible reprocesar un archivo simplemente eliminando sus registros en la tabla destino y volviéndolo a subir al bucket S3.


AWS CloudWatch Logs para monitoreo.

SQL Server para almacenamiento final.
