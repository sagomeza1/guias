# Guía de Principios SOLID en Python 🐍

Los principios SOLID son un acrónimo que representa cinco reglas de diseño de software orientado a objetos. Su objetivo es hacer que el código sea más legible, mantenible y escalable.

## 1. S: Single Responsibility Principle (SRP)

*"Una clase debe tener una sola razón para cambiar."*

Cada clase debe encargarse de una única parte de la funcionalidad.

### El Problema (Clase Dios)

Una clase que maneja datos, los guarda en la base de datos y los exporta a archivos es frágil. Si cambia la base de datos, podrías afectar la lógica del negocio.

### La Solución (Separación de Tareas)

Dividimos la clase en "especialistas" usando sustantivos para las clases y verbos para los métodos.

```python
class Libro:
    """Gestiona exclusivamente los datos del libro."""
    def __init__(self, titulo: str, autor: str):
        self.titulo = titulo
        self.autor = autor

    def obtener_detalles(self) -> str:
        return f"{self.titulo} por {self.autor}"

class PersistenciaLibro:
    """Gestiona el guardado en disco o base de datos."""
    def guardar(self, libro: Libro):
        print(f"Guardando '{libro.titulo}' en la BD... 💾")

class ExportadorLibro:
    """Gestiona los formatos de salida (JSON, XML)."""
    def exportar_a_json(self, libro: Libro):
        return {"titulo": libro.titulo, "autor": libro.autor}
```

## 2. O: Open/Closed Principle (OCP)

*"El software debe estar abierto para la extensión, pero cerrado para la modificación."*

No deberías tener que modificar el código que ya funciona para añadir nuevas funcionalidades.

### Implementación en Python

Usamos Clases Base Abstractas `(ABC)` para definir contratos.

```python
from abc import ABC, abstractmethod

class Exportador(ABC):
    @abstractmethod
    def exportar(self, libro: Libro):
        pass

class PDFExportadorLibro(Exportador):
    def exportar(self, libro: Libro):
        print(f"Generando PDF de '{libro.titulo}'... 📄")

class JSONExportadorLibro(Exportador):
    def exportar(self, libro: Libro):
        print(f"Generando JSON de '{libro.titulo}'... 📋")
```

*Si llega un nuevo formato (Excel), creamos una nueva clase sin tocar las anteriores.*

## 3. L: Liskov Substitution Principle (LSP)

*"Las subclases deben poder sustituir a sus clases base sin romper el programa."*

Si el programa espera un objeto `Exportador`, debería funcionar igual de bien si le pasas un `PDFExportador` o un `JSONExportador`. Si una subclase lanza errores inesperados o tiene restricciones que la base no tenía, viola este principio.

## 4. I: Interface Segregation Principle (ISP)

*"Un cliente no debe ser obligado a depender de métodos que no utiliza."*

Es mejor tener muchas interfaces pequeñas y específicas que una sola interfaz gigante (multitarea).

### Ejemplo de segregación

En lugar de una clase Multifuncional, dividimos las capacidades:

```python
class Impresora(ABC):
    @abstractmethod
    def imprimir(self, documento): pass

class Escaner(ABC):
    @abstractmethod
    def escanear(self, documento): pass

# Una impresora simple solo hereda lo que necesita
class ImpresoraBasica(Impresora):
    def imprimir(self, documento):
        print("Imprimiendo...")
```
## 5. D: Dependency Inversion Principle (DIP)

*"Depende de abstracciones, no de clases concretas."*

Los módulos de alto nivel no deben depender de módulos de bajo nivel. Ambos deben depender de abstracciones.

### Ejemplo con Inyección de Dependencias

El `Notificador` no crea el servicio de correo, lo recibe por parámetro (Inyección).

```python
class ServicioMensajeria(ABC):
    @abstractmethod
    def enviar(self, mensaje: str): pass

class Gmail(ServicioMensajeria):
    def enviar(self, mensaje: str):
        print(f"Enviando via Gmail: {mensaje}")

class Notificador:
    def __init__(self, servicio: ServicioMensajeria):
        # Dependemos de la abstracción, no de 'Gmail' directamente
        self.servicio = servicio

    def enviar_alerta(self, contenido: str):
        self.servicio.enviar(contenido)
# Inyección
notificador = Notificador(Gmail())
notificador.enviar_alerta("¡Principio D aplicado!")
```
## Resumen Final para Científicos de Datos

|Letra|Principio|Utilidad Práctica|
|-----|---------|-----------------|
|S|Responsabilidad Única|Separa la limpieza de datos (Pre-procesamiento) de la lógica del modelo.|
|O|Abierto/Cerrado|Permite añadir nuevos modelos de ML sin cambiar el pipeline de evaluación.|
|L|Sustitución de Liskov|Asegura que cualquier modelo (XGBoost, Random Forest) funcione en el mismo predictor.|
|I|Segregación de Interfaz|No obligues a un modelo simple a implementar métodos complejos de Deep Learning.|
|D|Inversión de Dependencias|Facilita cambiar la fuente de datos (CSV a SQL) sin tocar el análisis.|
