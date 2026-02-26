# Integración de Python con MongoDB

Ejemplo de código de integración:
```python
import datetime
from pymongo import MongoClient
from bson.objectid import ObjectId

# 1. CONEXIÓN (El Puente)
# Conectamos al servidor local por defecto. 
# En un entorno real, aquí iría tu cadena de conexión (Connection String).
client = MongoClient("mongodb://localhost:27017/")

# Seleccionamos la base de datos y la colección
# Si no existen, MongoDB las creará automáticamente al insertar el primer documento.
db = client["analitica_db"]
coleccion_ventas = db["ventas_diarias"]

# --- ANÁLISIS DE MODELADO ---
# Para ciencia de datos, preferimos documentos que contengan toda la info relevante.
# Ejemplo de estructura de un documento de venta:
ejemplo_documento = {
    "producto": "Laptop Pro 15",
    "categoria": "Electronica",
    "precio": 1200.50,
    "fecha": datetime.datetime.now(),
    "detalles": {  # Esto es "Embedding" (un documento dentro de otro)
        "marca": "TechBrand",
        "stock_previo": 50
    },
    "etiquetas": ["premium", "descuento", "verano"] # Listas nativas
}

# 2. ESCRITURA (Create)
def insertar_datos():
    print("--- Operaciones de Escritura ---")
    
    # Insertar un solo documento
    resultado_uno = coleccion_ventas.insert_one(ejemplo_documento)
    print(f"Documento insertado con ID: {resultado_uno.inserted_id}")

    # Insertar varios documentos (ideal para cargar datasets de Pandas)
    muchas_ventas = [
        {"producto": "Mouse Inalambrico", "categoria": "Accesorios", "precio": 25.0, "fecha": datetime.datetime.now()},
        {"producto": "Monitor 4K", "categoria": "Electronica", "precio": 400.0, "fecha": datetime.datetime.now()},
        {"producto": "Teclado Mecanico", "categoria": "Accesorios", "precio": 80.0, "fecha": datetime.datetime.now()}
    ]
    resultado_muchos = coleccion_ventas.insert_many(muchas_ventas)
    print(f"Se insertaron {len(resultado_muchos.inserted_ids)} documentos adicionales.\n")

# 3. LECTURA (Read)
def leer_datos():
    print("--- Operaciones de Lectura ---")
    
    # Buscar un solo documento (el primero que coincida)
    venta = coleccion_ventas.find_one({"categoria": "Electronica"})
    print(f"Venta encontrada: {venta['producto']} - ${venta['precio']}")

    # Buscar múltiples documentos con un filtro (Query)
    # Buscamos productos de 'Accesorios' con precio mayor a 30
    query = {
        "categoria": "Accesorios",
        "precio": {"$gt": 30} # $gt significa 'Greater Than' (mayor que)
    }
    
    cursor = coleccion_ventas.find(query)
    print("Accesorios caros encontrados:")
    for doc in cursor:
        print(f" - {doc['producto']}: ${doc['precio']}")
    print("")

# 4. ANALÍTICA BÁSICA (Aggregation Pipeline)
# En lugar de traer miles de datos a Python, dejamos que MongoDB los procese.
def realizar_analitica():
    print("--- Pipeline de Agregación (Transformación) ---")
    
    pipeline = [
        # Paso 1: Filtrar (como un WHERE en SQL)
        {"$match": {"precio": {"$exists": True}}},
        
        # Paso 2: Agrupar (como un GROUP BY en SQL)
        {"$group": {
            "_id": "$categoria", 
            "total_ventas": {"$sum": "$precio"},
            "promedio_precio": {"$avg": "$precio"},
            "conteo": {"$sum": 1}
        }},
        
        # Paso 3: Proyectar (limpiar la salida)
        {"$project": {
            "categoria": "$_id",
            "_id": 0,
            "total_ventas": 1,
            "ticket_promedio": "$promedio_precio"
        }}
    ]
    
    resultados = list(coleccion_ventas.aggregate(pipeline))
    for res in resultados:
        print(f"Categoría: {res['categoria']} | Total: ${res['total_ventas']:.2f}")

if __name__ == "__main__":
    insertar_datos()
    leer_datos()
    realizar_analitica()
```

## Conexión

Se usa `MongoClient` para establecer el vínculo.

## Escritura

- `insert_one()`: Para un solo registro.
- `insert_many()`: Para cargas masivas (convertir un DataFrame de pandas usando `df.to_dict('records')`).

## Lectura

`find_one()` y `find()`: Usan diccionarios para filtrar. Recuerda operadores como `$gt` (mayor que), `$lt` (menor que) o `$in` (en una lista).

## Agregación

Es el **motor de búsqueda** de MongoDB.

- `$match`: Filtra los datos de entrada.
- `$group`: Crea las métricas que necesitas (sumas, promedios).
- `$project`: Da formato final a los resultados.

## Consejo

Para que las lecturas sean rápidas, siempre se crea **índices** en los campos que se usan frecuentemente en los filtros (find o $match). En PyMongo se hace así: `coleccion_ventas.create_index('categoria')`. Esto evita que MongoDB tenga que revisar cada documento de la colección uno por uno.

# Integración de MongoDB y Pandas

En MongoDB, cuando se usa el método `.find()`, obtenemos un **cursor**, que en Python se comporta de forma muy similar a una lista de diccionarios. Dado que un DataFrame de Pandas puee construirse directamente desde una lista de diccionarios, la integración es casi nativa.

Se presenta una arquitectura para realizar este proceso de forma eficiente, incluyendo cómo manejar los objetos `ObjectId` de MongoDB que a veces causan problema al exportar datos.

```python
import pandas as pd
from pymongo import MongoClient
import datetime

# 1. CONFIGURACIÓN DE CONEXIÓN
def obtener_cliente():
    # Cadena de conexión (usamos local para este ejemplo)
    return MongoClient("mongodb://localhost:27017/")

def cargar_datos_a_dataframe():
    client = obtener_cliente()
    db = client["analitica_db"]
    coleccion = db["ventas_diarias"]

    # --- ANÁLISIS DE MODELADO PARA PANDAS ---
    # Al traer datos para análisis, es una buena práctica usar 'PROYECCIONES'.
    # La proyección nos permite traer solo las columnas que realmente necesitamos,
    # ahorrando memoria RAM en nuestro entorno de Python.
    
    filtro = {"precio": {"$gt": 0}} # Filtramos ventas válidas
    proyeccion = {
        "_id": 1,          # Incluimos el ID por defecto
        "producto": 1, 
        "categoria": 1, 
        "precio": 1, 
        "fecha": 1,
        "detalles.marca": 1 # Accedemos a campos anidados directamente
    }

    # 2. EXTRACCIÓN (De MongoDB a Lista de Dicts)
    cursor = coleccion.find(filtro, proyeccion)
    lista_documentos = list(cursor)

    if not lista_documentos:
        print("No se encontraron datos para cargar.")
        return pd.DataFrame()

    # 3. TRANSFORMACIÓN (De Lista a DataFrame)
    df = pd.DataFrame(lista_documentos)

    # --- LIMPIEZA DE DATOS ESPECÍFICA DE MONGODB ---
    
    # A. Manejo del ObjectId
    # El campo '_id' es un objeto especial de BSON. Para análisis suele ser mejor 
    # convertirlo a string o eliminarlo si no se va a usar como índice.
    if '_id' in df.columns:
        df['_id'] = df['_id'].apply(str)

    # B. Aplanado de datos (Flattening)
    # Si tienes datos anidados (como 'detalles.marca'), Pandas creará una columna 
    # que contiene diccionarios. Podemos usar json_normalize si la estructura es compleja.
    # En este ejemplo simple, si trajimos 'detalles.marca', aparecerá como un dict.
    
    return df

# 4. EJECUCIÓN Y VISTA PREVIA
if __name__ == "__main__":
    df_ventas = cargar_datos_a_dataframe()
    
    if not df_ventas.empty:
        print("¡Datos cargados exitosamente!")
        print(f"Forma del DataFrame: {df_ventas.shape}")
        print("\nPrimeras 5 filas:")
        print(df_ventas.head())
        
        # Ejemplo de análisis rápido con Pandas
        print("\nPromedio de precio por categoría:")
        print(df_ventas.groupby('categoria')['precio'].mean())
```

## La proyección

En lugar de hacer un `SELECT *`, le decimos a MongoDB exactamente qué campos se requieren. Las colecciones pueden tener cientos de campos (muchos de ellos irrelevantes para un modelo específico), esto es vital para no saturar la memoria.

## `list(cursor)`

El cursor de PyMongo es un iterador. Al convertirlo en lista, materializamos los datos en la memoria de Python para que Pandas pueda procesarlos.

## Conversión de `_id`

El tipo `ObjectId` no es un tipo de dato estándar de Python/Pandas. Si se planea exportar el DataFrame a CSV o Excel más adelante, convertirlo a `str` evitará errores de serialización.

## Estructuras Anidadas

Si los documentos tienen muchos niveles de profundidad, investigar `pd.json_normalize(lista_documentos)`, que expande automáticamente los diccionarios anidados en columnas separadas (por ejemplo: `detalles.marca` se convierte en una columna llamada `detalles.marca`).