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
