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