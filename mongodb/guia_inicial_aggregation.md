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