# week-2-lab-5-iron-hack
Laboratorio de optimización de consultas a bases de datos.


# 1. Implementando y optimizando consultas a base se datos.

## 1. SQL Query Optimization

Participants are given SQL queries and asked to improve them based on theoretical knowledge of database optimization.

### 1.- Provided SQL Queries for Optimization:
Orders Query: Retrieve orders with many items and calculate the total price.
```
SELECT Orders.OrderID, SUM(OrderDetails.Quantity * OrderDetails.UnitPrice) AS TotalPrice
FROM Orders
JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID
WHERE OrderDetails.Quantity > 10
GROUP BY Orders.OrderID;
```
### 2.- Customer Query: Find all customers from London and sort by CustomerName.

```
SELECT CustomerName FROM Customers WHERE City = 'London' ORDER BY CustomerName;
```

Optimization Tasks:
Participants identify inefficiencies and propose modifications to optimize these queries, such as indexing critical columns, restructuring joins, or adjusting filters.
Theoretical application of indexes or rewriting of subqueries is discussed to theoretically reduce computational load.

### Deficiencias Orders

* No se conoce si se tienen los indices OrderDetails.OrderID
* No se conoce si se tienen los indices OrderDetails.Quantity
* No se conoce si se tienen los indices Orders.OrderID
* No se conoce cual es la tabla más grande en el joion.
* Crear indice en OrderDetails para usar select, where y join.
* Si los datos no cambian una vez se escriben, no usa vistas materializadas.
* En el join primero se hace la unión de todos los orders y posteriormente se filtra  OrderDetails.Quantity > 10

### Optimización Orders
* Crear indice para ascelerar consulta join: OrderDetails.OrderID
* Crear indice para ascelerar consulta join: OrderDetails.Quantity
* Crear indice para ascelerar consulta join: Orders.OrderID
* Filtrar primero la tabla más grande ya sea OrderDetails o Orders
* Utilizar una vista materializada en caso de que los datos no sean cambiantes una vez se escriben en bd.
* Si la tabla es muy grande y no se cuenta con procesos de carga historica, considerar usar particiones y shards para mejor performances y realziar consultas especificas a los datos.
* Como es una consulta SELECT, se puede hacer uso de nodos de lectura y evitar meter carga a nodos primarios o transaccionales.
* Si son consultas recurrentes, considerar colocar un cluster de cache frente a la bd.

**Consulta optimizada**

```
-- Creación de indices
CREATE INDEX idx_orderdetails_orderid_quantity ON OrderDetails (OrderID, Quantity);
CREATE INDEX idx_orders_orderid ON Orders (OrderID);

-- Query
SELECT Orders.OrderID, SUM(OrderDetails.Quantity * OrderDetails.UnitPrice) AS TotalPrice
FROM Orders
JOIN OrderDetails ON Orders.OrderID = OrderDetails.OrderID AND OrderDetails.Quantity > 10
GROUP BY Orders.OrderID;
```

### Deficiencias Customers
* No se conoce si se cuenta con indice Customers.City
* No se conoce si se cuenta con indice Customers.CustomerName


### Optimización Customers
* Crear indice: Customers.City
* Crear indice: Customers.CustomerName

**Consulta optimizada**

```
-- Crear indices
CREATE INDEX idx_customers_city_customername ON Customers (City, CustomerName);

-- Query
SELECT CustomerName FROM Customers WHERE City = 'London' ORDER BY CustomerName;

```

## 2.- NoSQL Query Implementation

Participants will optimize provided NoSQL queries theoretically, focusing on efficient data retrieval and minimized latency.

**NoSQL Queries for Review:**
### 1.- User Posts Query: Retrieve the most popular active posts and display their title and like count.
```
db.posts
  .find({ status: "active" }, { title: 1, likes: 1 })
  .sort({ likes: -1 });
```
### 2.- User Data Aggregation: Summarize the number of active users by location.
User Data Aggregation: Summarize the number of active users by location.

```
db.users.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$location", totalUsers: { $sum: 1 } } },
]);

```

### Deficiencias User Posts Query

* No se cuenta o no se conoce si se tiene un indice compuesto  para status y likes.
* No se conoce cuantos posibles documentos puede regresar.
* No usa paginación o un limit de registros a retornar.
* No se conoce si usa una cache frente a bd.

### Optimización User Posts Query
* Agregar indice compuesto  para status y likes.
* Agregar paginación o limit de documentos a retornar.
* Se puede colocar una cache frente a mongo para optimizar consultas frecuentes.

**Se agregan indices**
```
db.posts.createIndex({ status: 1, likes: -1 });
```

**Query con Limit**
```
const limit = 10;

db.posts.find({ status: "active" }, { title: 1, likes: 1 })
  .sort({ likes: -1 })
  .limit(limit);
```
**Query con Paginado**
```
const page = 1; // Número de la página (comienza en 1)
const pageSize = 10; // Tamaño de la página

db.posts.find({ status: "active" }, { title: 1, likes: 1 })
  .sort({ likes: -1 })
  .skip((page - 1) * pageSize)
  .limit(pageSize);
```

### Deficiencias User aggregate

* Se recomienda agregar indice en status.
* Si no queremor retornar todo el documento, se recomienda agregar una projection en la etapa.

### Optimización User aggregate
* Agregar indice en status
* Agregar projection para solo regresar determinados campos del documento.

**Se agrega indice**
```
db.users.createIndex({ status: 1 });

```

**Agregate con projection**


```
db.users.aggregate([
  { $match: { status: "active" } },
  { $group: { _id: "$location", totalUsers: { $sum: 1 } } },
  { $project: { _id: 0, location: "$_id", totalUsers: 1 } }
]);
```

