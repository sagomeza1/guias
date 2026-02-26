# Aggregation Framework

El Aggregation Framework de MongoDB es una poderosa herramienta diseñada para realizar análisis de datos profundos. 

El Aggregation Framework se utiliza a través de un comando llamado aggregate, que permite ejecutar múltiples etapas o procesos denominados pipelines. Cada pipeline realiza una operación específica sobre los datos de entrada. 

```shell
db.listingandreviews.aggregate([
    { $match: { amenities: "Wifi" } },
    { $project: { address: 1 } },
    { $group: { _id: "$address.country", count: { $sum: 1 } } }
])
```

# Tutorial de Agregaciones en MongoDB para Ciencia de Datos

En MongoDB, la agregación no es solo "contar cosas". Es un proceso de transformación de documentos. A diferencia de SQL, donde las consultas suelen ser declarativas, en MongoDB las agregaciones son procedurales: el orden de las etapas importa.

## 1. El Concepto de "Embedding" vs. "Referencing"

Antes de procesar, debemos entender cómo están los datos.

- **Embedding (Incrustar):** Guardamos datos relacionados dentro del mismo documento (ej: una lista de etiquetas dentro de un producto). Es ideal para lecturas rápidas de datos que siempre se usan juntos.

- **Referencing (Referenciar):** Guardamos el ID de un documento en otra colección (como una Foreign Key). Se usa cuando los datos crecen mucho o se comparten entre muchas entidades.

## 2. Las "Estaciones" del Pipeline (Etapas Principales)

Para este tutorial, imaginemos una colección de `ventas` con esta estructura:

```shell
{
  "_id": 1,
  "cliente_id": "C123",
  "productos": [
    {"nombre": "Laptop", "precio": 1200, "categoria": "tecnologia"},
    {"nombre": "Mouse", "precio": 25, "categoria": "tecnologia"}
  ],
  "fecha": "2023-10-27T10:00:00Z",
  "estado": "completado",
  "total": 1225
}
```

### Etapa 1: `$match` (El Filtro)

Es el equivalente al `WHERE` en SQL. Siempre intenta ponerlo al principio para reducir la cantidad de datos que viajan por el pipeline.

### Etapa 2: `$unwind` (Descomponer Listas)

Esta es la etapa favorita de los Data Scientists. Si tienes un array de productos, `$unwind` crea un documento individual por cada elemento del array. Es vital para normalizar datos antes de un análisis granular.

### Etapa 3: `$group` (El Pivot)

Equivalente al `GROUP BY`. Aquí es donde ocurre la magia estadística: sumas, promedios, máximos y mínimos.

### Etapa 4: `$project` (La Selección)

Equivalente al `SELECT`. Nos permite limpiar el ruido, renombrar campos o crear campos calculados (como el IVA de un precio).

## 3. Implementación en Mongo Shell (mongosh)

Ideal para pruebas rápidas directamente en la terminal o Compass.
```shell
// Objetivo: Obtener el ingreso total por categoría de productos en ventas completadas
db.ventas.aggregate([
  // 1. Filtramos solo las ventas exitosas
  { $match: { "estado": "completado" } },
  
  // 2. "Explotamos" el array de productos para tratar cada uno por separado
  { $unwind: "$productos" },
  
  // 3. Agrupamos por la categoría que está dentro del objeto producto
  { 
    $group: { 
      "_id": "$productos.categoria", 
      "ingreso_total": { $sum: "$productos.precio" },
      "cantidad_vendida": { $sum: 1 }
    } 
  },
  
  // 4. Ordenamos por ingreso de mayor a menor
  { $sort: { "ingreso_total": -1 } }
]);
```
## 4. Implementación en Python (PyMongo)

Si estás trabajando en un Jupyter Notebook o un script de ingeniería de datos, usarás esta sintaxis.
```python
from pymongo import MongoClient

# Conexión
client = MongoClient("mongodb://localhost:27017/")
db = client.tienda_analitica

# Definición del Pipeline
pipeline = [
    {
        "$match": {
            "estado": "completado"
        }
    },
    {
        "$unwind": "$productos"
    },
    {
        "$group": {
            "_id": "$productos.categoria",
            "promedio_precio": { "$avg": "$productos.precio" },
            "total_items": { "$sum": 1 }
        }
    },
    {
        "$project": {
            "_id": 0, # Ocultamos el _id generado por el group
            "categoria": "$_id", # Renombramos _id a categoria
            "promedio_precio": 1,
            "total_items": 1
        }
    }
]

# Ejecución
resultados = list(db.ventas.aggregate(pipeline))

for res in resultados:
    print(f"Categoría: {res['categoria']} | Promedio: {res['promedio_precio']:.2f}")
```

5. Relacionando Datos con `$lookup` (El Join de NoSQL)

A veces los datos están separados. Supongamos que queremos traer la información del cliente desde la colección `clientes`.

```shell
// Sintaxis en Mongo Shell
db.ventas.aggregate([
  {
    $lookup: {
      from: "clientes",           // Colección a unir
      localField: "cliente_id",   // Campo en 'ventas'
      foreignField: "id_usuario", // Campo en 'clientes'
      as: "info_cliente"          // Nombre del nuevo array resultante
    }
  }
]);
```

## Consejos de Arquitecto para el Rendimiento

- **Índices:** Si tu primera etapa es un `$match` o un `$sort`, asegúrate de tener un índice en esos campos. Sin índices, MongoDB tendrá que leer cada documento del disco (Colscan), lo cual es muy lento en millones de registros.

- **Memoria:** Las etapas de `$group` y `$sort` tienen un límite de memoria de 100MB por defecto. Si trabajas con Big Data, podrías necesitar la opción `{ allowDiskUse: true }`.

- **Proyecta temprano:** Usa `$project` para eliminar campos pesados (como descripciones largas o imágenes en base64) si no los necesitas para el análisis. Menos datos en el pipeline = más velocidad.