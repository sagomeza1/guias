# Guía Maestra: CRUD en MongoDB para Ciencia de Datos (IoT)

Esta guía resume las operaciones fundamentales para gestionar datos de sensores utilizando la flexibilidad de los documentos BSON de MongoDB.

## 🏗️ 1. Modelado de Datos (Estructura Sugerida)

En IoT, priorizamos el **Embedding** (incrustar) para mantener la relación temporal de las lecturas.
```shell
{
  "sensor_id": "TERM-001",
  "timestamp": "2026-02-24T21:30:00Z",
  "lectura": [
    { "valor": 22.5, "unidad": "Celsius", "tipo": "temperatura" },
    { "valor": 45, "unidad": "%", "tipo": "humedad" }
  ],
  "ubicacion": "Planta_A"
}
```
## ➕ 2. CREATE (Crear)

Insertar datos en colecciones flexibles.

### 🐍 PyMongo

```python
from pymongo import MongoClient
from datetime import datetime

client = MongoClient("mongodb://localhost:27017/")
db = client["iot_database"]
coleccion = db["lecturas_sensores"]

nueva_lectura = {
    "sensor_id": "TERM-001",
    "timestamp": datetime.utcnow(),
    "lectura": [
        { "valor": 22.5, "unidad": "Celsius", "tipo": "temperatura" },
        { "valor": 45, "unidad": "%", "tipo": "humedad" }
    ],
    "ubicacion": "Planta_A"
}

coleccion.insert_one(nueva_lectura)
```

### 🐚 Mongo Shell

```shell
db.lecturas_sensores.insertOne({
  sensor_id: "TERM-001",
  timestamp: new Date(),
  lectura: [
    { valor: 22.5, unidad: "Celsius", tipo: "temperatura" },
    { valor: 45, unidad: "%", tipo: "humedad" }
  ],
  ubicacion: "Planta_A"
})
```

## 🔍 3. READ (Leer)

Consultar datos usando filtros y operadores de comparación.

- `$gt` / `$gte`: Mayor que / Mayor o igual.
- `$lt` / `$lte`: Menor que / Menor o igual.

### 🐚 Mongo Shell (Ejemplo: Filtro por temperatura alta)

```shell
db.lecturas_sensores.find({
  "ubicacion": "Planta_A",
  "lectura.valor": { $gte: 25 }
})
```

## 🛠️ 4. UPDATE (Actualizar)

Modificar campos existentes sin sobreescribir el documento completo.

**IMPORTANTE:** Siempre usa el operador `$set` para actualizaciones parciales.

### 🐚 Mongo Shell

```shell
db.lecturas_sensores.updateOne(
  { "sensor_id": "TERM-001" },
  { $set: { "ubicacion": "Planta_B" } }
)
```
## 🗑️ 5. DELETE (Eliminar)

Limpieza de datos erróneos o "outliers".

### 🐚 Mongo Shell

```shell
db.lecturas_sensores.deleteMany({
  "lectura.valor": { $gt: 100 }
})
```

## 🚀 Consejo de Optimización

Para que tus lecturas de datos sean rápidas, especialmente cuando tienes millones de registros de sensores, siempre crea índices en los campos que más consultas (como el ID del sensor o el tiempo):
```shell
// Crear índice ascendente por timestamp
db.lecturas_sensores.createIndex({ "timestamp": 1 })
```
