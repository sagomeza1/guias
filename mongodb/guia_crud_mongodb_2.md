# CRUD MONGODB

## Insertar un solo documento

Se crea un archivo `insert.mongodb`.

Mongo es flexible, por lo que no es necesario crear la base de datos, simplemente con decir que se usara `use("store")` la genera de forma automática. Lo mismo ocurre con las colecciones.

```shell
use("store")

db.products.insertOne({
    name: "Bebida",
    price: 10
})

db.products.insertOne({
    name: "Comida",
    price: 11
})
```
Se puede consultar los documentos de las colecciones con `db.products.find()`.

```shell
store> db.products.find()

[
  {
    "_id": {
      "$oid": "699f1495f372b5f7a2a256a2"
    },
    "name": "Bebida",
    "price": 10
  },
  {
    "_id": {
      "$oid": "699f1495f372b5f7a2a256a3"
    },
    "name": "Comida",
    "price": 11
  }
]
```

## Eliminar documentos
Para vaciar la colección usa `db.products.drop()`.

## Insertar varios documentos

Para insertar varios registros a la vez se usa `db.products.insertMany()`:
```shell
use("store")

db.products.drop()

db.products.insertMany([
    {
        name: "Cerveza",
        price: 5,
    },
    {
        name: "Jugo",
        price: 3,
    },
    {
        name: "Hamburguesa",
        price: 12,
    },
    {
        name: "Helado",
        price: 2,
    },
])

db.products.find()
```

## Actualizar documento

### `$set: {}`

Para actualizar uno o varios documentos tenemos que definir un operador, para modificar o agregar atributos se usa `"set:{}`:

```shell
use("store")

db.products.updateOne(
    // query
    {name:"Jugo"},
    //change => operator
    {
        $set: {
            price: 4,
            tax: 0.05,
            tags: ["fruta", "agua", "azucar"]
        }
    }
);
```
Obtenemos:

```shell
store> db.products.find({name:"Jugo"})
[
  {
    "_id": {
      "$oid": "699f24d287c7f026c9025dab"
    },
    "name": "Jugo",
    "price": 4,
    "tags": [
      "fruta",
      "agua",
      "azucar"
    ],
    "tax": 0.05
  }
]
```
### `$inc: {}`

De incrementar, se usa con campos que guardan valores numericos para realizar un incremento:

```shell
use("store")

db.producst.updateOne(
    // query
    {name:"Cerveza"},
    //change => operator
    {
        $inc: {
            price: 1
        }
    }
);
```
Obtenemos:

```shell
store> db.producst.find({name:"Cerveza"})
[
  {
    "_id": {
      "$oid": "699f24d287c7f026c9025daa"
    },
    "name": "Cerveza",
    "price": 6
  }
]
```

Para realizar la actualización a un documento en especifico acudimos a su `ObjectId("699f24d287c7f026c9025daa")`.

```shell
use("store")

db.producst.updateOne(
    // query
    {_id:ObjectId("699f24d287c7f026c9025dac")},
    //change => operator
    {
        $inc: {
            price: 2
        }
    }
);
```
Obtenemos:
```shell
store> db.producst.find({_id:ObjectId("699f24d287c7f026c9025dac")})
[
  {
    "_id": {
      "$oid": "699f24d287c7f026c9025dac"
    },
    "name": "Hamburguesa",
    "price": 14
  }
]
```
### Operadores


Listado de operadores de actualización:
|Operador|Función|
|--------|-------|
|`$set:` | Asigna un valor específico a un atributo. |
|`$inc:` |Incrementa el valor de un atributo numérico en una cantidad específica. |
|`$mul:` |Multiplica el valor de un atributo numérico por un factor específico. |
|`$rename:` |Cambia el nombre de un atributo. |
|`$unset:`| Elimina un atributo de un documento. |
|`$min:` | Actualiza el valor de un atributo con el valor mínimo especificado, sólo si el valor actual es mayor que el valor especificado. |
|`$max:` | Actualiza el valor de un atributo con el valor máximo especificado, sólo si el valor actual es menor que el valor especificado. |
|`$currentDate:` | Establece el valor de un atributo como la fecha y hora actual. |
|`$addToSet:` | Añade un valor a un atributo de tipo conjunto (`array`), sólo si el valor no existe en el conjunto. |
|`$pop:` | Elimina el primer o último elemento de un atributo de tipo conjunto (`array`). |
|`$pull:` | Elimina un valor específico de un atributo de tipo conjunto (`array`). |
|`$push:` | Añade un valor a un atributo de tipo conjunto (`array`). |
|`$pullAll:` | Elimina varios valores específicos de un atributo de tipo conjunto (`array`).|

Para actualizar varios documentos se usa `db.products.updateMany()`:

```shell
use("store")

db.products.updateMany(
    // query
    {price:{$lt:10}},
    // actualizacioń
    {
        $set:{
            tax:0.05
        }
    }
);
```

Para modificar todos los productos, la query debe estar vacia:
```shell
use("store")

db.products.updateMany(
    // query
    {},
    // actualizacioń
    {
        $push:{
            target: {
            $in : ["tag c", "tag f"]
            }
        }
    }
);
```

### UPSERT

Para realizar un upsert:
```shell
use("store")

db.iot.updateOne({
  sensor: "A001",
  date: "2025-01-31"
}, {
  $push:{
    reading: 1212
    }
}, {
  upsert: true
})
```
Si no se presenta el registro, se crea de forma automática

## Eliminar documentos

```shell
// Elimina un doc
db.products.deleteOne({ _id: 1 });
// Elimina de acuerdo al filtro
db.products.deleteMany({ price: 100 });
// Elimina todos los docs
db.products.drop();
```

<!--  -->
