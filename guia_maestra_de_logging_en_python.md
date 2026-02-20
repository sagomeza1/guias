# Guía Maestra: Implementación Profesional de Logging en Python 🪵

Esta guía resume los estándares de la industria para el registro de eventos en aplicaciones de Python, con especial énfasis en el ecosistema de Ciencia de Datos.

## 1. Jerarquía de Niveles de Severidad

El sistema de logging funciona como un filtro. Solo los mensajes con un nivel igual o superior al umbral configurado serán procesados.

|Nivel | ValorUso | Recomendado|
|------|----------|------------|
|DEBUG|10|Diagnóstico técnico detallado (ej. valores de gradientes).|
|INFO|20|Confirmación de hitos (ej. "Modelo cargado con éxito").|
|WARNING|30|Eventos inesperados que no impiden la ejecución.|
|ERROR|40|Fallos graves en una funcionalidad específica.|
|CRITICAL|50|Errores fatales que detienen la aplicación completa.|

## 2. Configuración Inicial (Nivel 1: Básico)

Ideal para scripts rápidos o prototipos iniciales donde se requiere un registro centralizado simple.
```Python
import logging

# Configuración básica para salida por consola
logging.basicConfig(
    level=logging.INFO,
    format='%(asctime)s - %(levelname)s - %(message)s',
    datefmt='%H:%M:%S'
)

def run_quick_task(name: str) -> None:
    """
    Ejecuta una tarea simple registrando su inicio.
    
    Args:
        name: Nombre de la tarea.
    """
    logging.info(f"Task {name} started")
```

## 3. Arquitectura Multi-Handler (Nivel 2: Profesional)

En entornos de producción, es una mejor práctica separar el destino de los logs. Este patrón permite enviar logs ligeros a la consola y logs detallados a un archivo.
```python
import logging
import sys
from typing import Optional

def get_production_logger(logger_name: str, log_file: str) -> logging.Logger:
    """
    Crea un logger configurado con doble destino y formatos diferenciados.
    
    Args:
        logger_name: Identificador único del logger.
        log_file: Ruta del archivo donde se guardará el historial detallado.
        
    Returns:
        Instancia de logging.Logger configurada.
    """
    # 1. Definición del Logger principal
    logger = logging.getLogger(logger_name)
    logger.setLevel(logging.DEBUG) # Nivel base para permitir que los handlers filtren

    # 2. Definición de Formatos
    # Formato detallado con archivo y número de línea para depuración
    detailed_formatter = logging.Formatter(
        '%(asctime)s | %(levelname)-8s | %(filename)s:%(lineno)d | %(message)s'
    )
    # Formato simple para consola
    simple_formatter = logging.Formatter('%(levelname)s: %(message)s')

    # 3. Handler de Consola (Solo INFO y superiores)
    console_handler = logging.StreamHandler(sys.stdout)
    console_handler.setLevel(logging.INFO)
    console_handler.setFormatter(simple_formatter)

    # 4. Handler de Archivo (Todo desde DEBUG)
    file_handler = logging.FileHandler(log_file)
    file_handler.setLevel(logging.DEBUG)
    file_handler.setFormatter(detailed_formatter)

    # 5. Integración
    if not logger.handlers:
        logger.addHandler(console_handler)
        logger.addHandler(file_handler)

    return logger

# Ejemplo de uso:
# logger = get_production_logger("ML_Pipeline", "pipeline.log")
# logger.debug("Hyperparameters: {'lr': 0.01}") -> Solo va al archivo
# logger.info("Process Started") -> Va a consola y archivo
```
## 4. Gestión de Archivos y Rendimiento

Para evitar que los archivos de log consuman todo el almacenamiento, se utiliza la rotación automática.

Rotación por Tamaño (`RotatingFileHandler`)

```python
from logging.handlers import RotatingFileHandler

def setup_rotating_logger(filename: str) -> None:
    """
    Configura un registro que rota al alcanzar 10MB, manteniendo 5 respaldos.
    """
    handler = RotatingFileHandler(
        filename, 
        maxBytes=10*1024*1024, # 10MB
        backupCount=5
    )
    # ... resto de la configuración ...
```

## 5. Mejores Prácticas (Pro-Tips)

1. No uses print(): Los prints no tienen niveles de severidad ni marcas de tiempo automáticas.

2. Nomenclatura: Usa `__name__` al crear loggers dentro de módulos (`logging.getLogger(__name__)`) para que el log indique exactamente en qué módulo ocurrió el evento.

3. Manejo de Excepciones: Usa `logger.exception("Mensaje")` dentro de bloques `except`. Esto adjunta automáticamente el Stack Trace (rastreo de error) al mensaje de log.

4. Performance: En bucles críticos de Ciencia de Datos, evita el nivel DEBUG si el volumen de mensajes es masivo (miles por segundo).

5. Seguridad: Nunca registres contraseñas, tokens de API o datos personales sensibles (PII) en los archivos de log.

Manual generado por Python Data Science Mentor.