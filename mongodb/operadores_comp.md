# Operadores

 Los operadores de comparación nos permiten realizar consultas más complejas y específicas.

## `$qe` y `$ne`

*"igual que y diferente de"*

El operador equal en MongoDB se emplea para buscar documentos en una colección que coincidan con un valor específico en un campo. Aunque este operador trae implícito el equal en las consultas de MongoDB, también se puede usar de una manera explícita para mayor claridad.
```shell
// Consulta implícita
db.inventory.find({ quantity: 20 })

// Consulta explícita
db.inventory.find({ quantity: { $eq: 20 } })
```
MongoDB nos permite realizar consultas en subdocumentos dentro de un documento principal. Para acceder a estos, los nombres de los campos deben especificarse de manera completa usando comillas dobles sin errores tipográficos.

```shell
// Búsqueda en un subdocumento
db.inventory.find({ "item.name": "AB" })
```
A diferencia del operador equal, el operador not equal ($ne) busca la negación en el contexto del valor especificado. Esto significa que devolverá todos los documentos que no coincidan con el valor dado en el campo.
```shell
// Consulta para obtener documentos con quantity distinto a 20
db.inventory.find({ quantity: { $ne: 20 } })
```
Los operadores de comparación pueden integrarse a las instrucciones de actualización como update en MongoDB. Esto resulta muy útil cuando deseamos realizar cambios en los documentos que cumplan con ciertas condiciones.
```shell
// Aumentar el campo quantity por 10 en documentos donde la quantity no es 20
db.inventory.updateMany(
  { quantity: { $ne: 20 } },
  { $inc: { quantity: 10 } }
)
```

## `$gt`, `$gte`, `$lt` y `$lte`
Al trabajar con números, los operadores como mayor que (`$gt`), mayor o igual que (`$gte`), menor que (`$lt`), y menor o igual que (`$lte`), son esenciales para obtener información valiosa. Con estos operadores, puedes construir consultas más sofisticadas y dinámicas, especialmente cuando estás manipulando grandes volúmenes de datos.

### Uso de `GreaterThan` y `GreaterThanOrEqual`
```shell
db.collection.find({ 
  "campo": { 
    $gt: valor 
  } 
})

db.collection.find({ 
  "campo": { 
    $gte: valor 
  } 
})

db.collection.find({ 
  "campo": { 
    {$gte: valor1, $lt: valor2}
  } 
})
```
Uno de los casos prácticos interesantes es la actualización de datos utilizando operadores. Supongamos que queremos actualizar los documentos de sensores que coinciden con cierta condición específica.

Queremos eliminar elementos de un `array` llamado `readings` en documentos que cumplen con una condición, usando el operador `pull`.

```shell
{
  updateMany: {
    query: { sensor_id: "A001" },
    update: {
      $pull: {
        readings: { $gte: 3 }
      }
    }
  }
}
```
## Busqueda con `$regex`

El operador de `regex` es un potente recurso en el mundo del desarrollo de software que permite realizar búsquedas de patrones dentro de un texto. Resulta especialmente útil cuando se trabaja con bases de datos que contienen campos de texto, al facilitar la identificación de documentos que coincidan con ciertas condiciones específicas que no podrían ser detectadas con una búsqueda básica. Esto es vital para mejorar la eficiencia en la búsqueda de datos dentro de la base de datos, haciendo consultas más precisas y efectivas.

a búsqueda con regex es muy versátil y ofrece varias formas de acotar o expandir la búsqueda según tus necesidades. Aquí te explico las formas comunes de buscar patrones de texto:

- **Búsqueda básica de un texto específico:** Puedes buscar si una cierta palabra está dentro de un campo de texto. Si quieres buscar la palabra "line", simplemente debes usar:
```shell
db.inventory.find({ 'item.description': {$regex: /line/ }})

// Insensible a las mayus "i"
db.inventory.find({ 'item.description': {$regex: /line/i }})

// Considera los saltos de línea "m"
db.inventory.find({ 'item.description': {$regex: /^S/m }})
```
## Operadores de `arrays`

|Operador|Función|
|--------|-------|
|`$in` | Para seleccionar documentos que tengan un campo con un valor que coincida con cualquiera de los valores especificados en un array.|
|`$nin`| Selecciona documentos que no tengan un campo con un valor que coincida con ninguno de los valores especificados en un array. (Contrario a `$in`).|
|`$all` | Selecciona documentos que tengan un campo con un array que contenga todos los valores especificados en la consulta.|
|`$elemMatch` | Para seleccionar documentos que tengan un campo con un array que contenga al menos un elemento que cumpla con ciertos criterios de consulta.|
|`$size`| Selecciona documentos que tengan un campo con un array de un tamaño específico.|

- **Operador in:** Este operador te permite buscar valores específicos dentro de un documento. Por ejemplo, si quieres encontrar documentos donde un Array contiene ciertos valores, in es tu herramienta.

- **Operador nin:** Actúa como el contrario de in; encuentra aquellos documentos que no contienen los valores especificados dentro del Array. Este operador garantiza que tus consultas sean precisas y te permite filtrar datos no deseados.
```shell
db.inventory.find({
    quantity: { $in: [20, 25] }
});

db.inventory.find({
    tags: { $nin: ["book", "electronics"] }
});
```

- **Operador all:** Compara `arrays` no por su contenido exacto, sino por si contienen un conjunto específico de elementos, sin importar el orden.
- **Operador size**: Este operador centra su funcionalidad en determinar el tamaño del `array`. Es ideal para cuando necesitas saber cuántos elementos contiene un `array` en particular.
- **Operador elemMatch:** Si trabajas con `arrays` de documentos u objetos, `elemMatch` permite realizar consultas más complejas, exigiendo que se cumplen varias condiciones dentro de un solo elemento del `array`.
```shell
// all
db.inventory.find({
    tags: { $all: ["school", "book"] }
});

// size
db.inventory.find({
    tags: { $size: 2 }
});

// elemMatch
db.surveys.find({
    results: {
        $elemMatch: {
            product: "xyz",
            score: { $gte: 7 }
        }
    }
});
```
## Operadores lógicos

En MongoDB, los operadores lógicos son fundamentales para la construcción de consultas complejas, permitiéndonos filtrar datos y trabajar con múltiples condiciones de búsqueda. Los operadores básicos incluyen `AND`, `OR`, `NOR` y `NOT`. 

```shell
// AND
db.inspection.find({
    $and: [
        { "field1": "value1" },
        { "field2": "value2" }
    ]
    } , {
        result: 1,
        _id: 0
    }
)
```