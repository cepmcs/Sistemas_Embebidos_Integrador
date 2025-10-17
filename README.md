# Sistemas Embebidos – Integrador

Proyecto integrador que captura lecturas de un sensor MPU6050 desde una placa Arduino y las envía en tiempo real a una tabla de Google BigQuery mediante la BigQuery Storage Write API.

## Arquitectura
- **Arduino + MPU6050**: genera lecturas de aceleración (X/Y/Z) y velocidad angular (Roll/Pitch/Yaw).
- **PC con Python**: `integrador.py` abre el puerto serie (`/dev/ttyACM0` por defecto), agrupa los datos en batches de 10 s y los serializa con un esquema Proto generado dinámicamente.
- **BigQuery**: recibe los registros en el dataset `Integrador_Embebidos`, tabla `Sistemas_Embebidos` del proyecto `savvy-climber-426522-e8`.

```
MPU6050 -> Arduino -> USB/Serial -> integrador.py -> BigQuery Storage Write API -> BigQuery
```

## Requisitos
- **Hardware**
  - Placa Arduino compatible (ej. Arduino Uno/Mega) conectada al MPU6050.
  - Cable USB para conexión con el equipo que ejecutará Python.
- **Software**
  - Python 3.9+ con las dependencias:
    ```bash
    pip install google-cloud-bigquery google-cloud-bigquery-storage grpcio-tools pyserial
    ```
  - Arduino IDE con la librería [`MPU6050` de Jarzebski](https://github.com/jarzebski/Arduino-MPU6050) instalada desde el Library Manager.
  - Credenciales de Google Cloud con permisos para BigQuery (BigQuery Admin o BigQuery Data Editor + BigQuery Job User).

## Configuración de Google Cloud
1. Habilita las APIs **BigQuery** y **BigQuery Storage** en tu proyecto.
2. Crea un dataset `Integrador_Embebidos` y, dentro, la tabla `Sistemas_Embebidos` con el siguiente esquema:
   | Campo | Tipo BigQuery | Descripción |
   | --- | --- | --- |
   | `Xac` | FLOAT | Aceleración eje X |
   | `Yac` | FLOAT | Aceleración eje Y |
   | `Zac` | FLOAT | Aceleración eje Z |
   | `Roll` | FLOAT | Velocidad angular eje X |
   | `Pitch` | FLOAT | Velocidad angular eje Y |
   | `Yaw` | FLOAT | Velocidad angular eje Z |
   | `Timestamp` | TIMESTAMP | Marca temporal generada en el script de Python |
3. Descarga un archivo de credenciales de servicio y exporta la ruta, por ejemplo:
   ```bash
   export GOOGLE_APPLICATION_CREDENTIALS=/ruta/al/archivo.json
   ```

## Preparación del Arduino
1. Abre `sketch_integrador/sketch_integrador.ino` con el Arduino IDE.
2. Selecciona la placa y puerto correctos, luego sube el sketch.
3. Abre el Monitor Serie para confirmar que aparece el mensaje `Initialize MPU6050` y que se emiten seis valores separados por guiones bajos.

## Ejecución del script de Python
1. Conecta la placa al PC y valida el puerto serie (en Linux suele ser `/dev/ttyACM0`; ajusta la constante `port` en `integrador.py` si es necesario).
2. Asegúrate de haber instalado las dependencias y configurado `GOOGLE_APPLICATION_CREDENTIALS`.
3. Ejecuta:
   ```bash
   python integrador.py
   ```
4. El script esperará 10 s para acumular lecturas y luego enviará un lote a BigQuery. Verás el mensaje `enviado` en consola cuando se complete cada batch.
