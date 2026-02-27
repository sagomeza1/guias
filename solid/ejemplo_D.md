# Principio **D**ependency Inversion Principle (DIP)

Para este ejemplo, imagina que estamos construyendo un **Sistema de Notificaciones**.

## 1. El Módulo de Alto Nivel: `GestorNotificaciones`

Este módulo contiene la lógica de negocio: "Quiero avisar al usuario". No le importa si es por Email, SMS o Telegram.

## 2. El Módulo de Bajo Nivel: `ServicioEmail`

Este es el detalle técnico: se conecta a un servidor SMTP y envía bytes por la red.

## La Implementación CORRECTA (Principio D)

Aquí, el módulo de alto nivel define un contrato (abstracción) y el de bajo nivel se adapta a él.

```python
from abc import ABC, abstractmethod

# --- ABSTRACCIÓN (El Contrato) ---
# Ninguno de los dos módulos depende del otro, ambos dependen de esto.
class Notificador(ABC):
    @abstractmethod
    def enviar(self, mensaje: str):
        pass

# --- MÓDULO DE BAJO NIVEL (Detalle técnico) ---
class ServicioEmail(Notificador): # Implementa la interfaz
    def enviar(self, mensaje: str):
        print(f"Conectando al servidor SMTP... Enviando Email: {mensaje}")

class ServicioSMS(Notificador): # Podríamos añadir otro fácilmente
    def enviar(self, mensaje: str):
        print(f"Conectando a la red móvil... Enviando SMS: {mensaje}")

# --- MÓDULO DE ALTO NIVEL (Lógica de negocio) ---
class GestorNotificaciones:
    def __init__(self, notificador: Notificador): 
        # INVERSIÓN: Depende de la abstracción 'Notificador', 
        # no de 'ServicioEmail' directamente.
        self.notificador = notificador

    def ejecutar(self, texto: str):
        # Aquí va la lógica compleja de negocio...
        # Y finalmente se apoya en la abstracción:
        self.notificador.enviar(texto)

# --- USO (Inyección de Dependencias) ---
servicio = ServicioEmail() # Creamos el empleado
app = GestorNotificaciones(servicio) # Se lo pasamos al jefe
app.ejecutar("¡Hola, SOLID!")
```

## ¿Por qué esto cumple el Principio D?

1. **Independencia:** Si mañana quieres usar SMS en lugar de Email, no tienes que tocar ni una sola línea de código dentro de `GestorNotificaciones`.

2. **Dirección de las dependencias:** El "Jefe" (Alto Nivel) ya no importa al "Empleado" (Bajo Nivel). Ahora el "Empleado" debe cumplir con las reglas impuestas por la interfaz `Notificador`.