# Trabajo de Consulta 2 - Conexion con base de datos 
## ¿Qué es JDBC y cuáles son sus componentes?
Java™ Database Connectivity (JDBC) es la especificación JavaSoft de una interfaz de programación de aplicaciones (API) estándar que permite que los programas Java accedan a sistemas de gestión de bases de datos. La API JDBC consiste en un conjunto de interfaces y clases escribas en el lenguaje de programación Java.

Con estas interfaces y clases estándar, los programadores pueden escribir aplicaciones que conecten con bases de datos, envíen consultas escritas en el lenguaje de consulta estructurada (SQL) y procesen los resultados.

Puesto que JDBC es una especificación estándar, un programa Java que utilice la API JDBC puede conectar con cualquier sistema de gestión de bases de datos (DBMS), siempre y cuando haya un controlador para dicho DBMS en concreto.

### Sus componentes son: 
- Driver JDBC:Es un controlador que actúa como intermediario entre la aplicación Java y la base de datos. Traduce las llamadas JDBC en comandos específicos para la DB.
- Conexión (Connection): Representa una sesión entre la aplicación y la base de datos.
- Declaración (Statement): Permite enviar comandos SQL a la base de datos.
- Resultados (ResultSet): Representa los datos devueltos por una consulta SQL.
- Gestión de excepciones (SQLException): Maneja errores que ocurren durante la interacción con la DB. 
- DataSource: Ofrece un enfoque más flexible y optimizado para la gestión de conexiones. 

## Documente 2 librerías de Scala que permitan conectarse a una base de datos relacional. En una tabla resuma sus diferencias.
| **Librería** | **Descripción** | **Ventajas** | **Desventajas** |
|--------------|-----------------|--------------|------------------|
| **Slick**    | Es una librería funcional para interactuar con bases de datos relacionales con Scala. Se basa en un enfoque en funciones y soporte para consultas típicamente tipadas. | - Consultas tipadas que reducen errores en tiempo de compilación.  <br> - API funcional.  <br> - Compatible con varias bases de datos. | - Curva de aprendizaje pronunciada.  <br> - Overhead de configuración inicial. |
| **Doobie**   | Es uan Librería ligera y funcional para gestionar conexiones a bases de datos en Scala. Utiliza Cats Effect para la gestión de la concurrencia. | - Integración con Cats y ZIO.  <br> - Más sencilla que Slick para tareas básicas.  <br> - Diseñada para pruebas fáciles. | - Menos soporte para consultas complejas en comparación con Slick.  <br> - Requiere gestión manual de queries. |

## Documentar cómo establecer una conexión a una base de datos relacional (mysql).

### Genere una base de datos en mysql.
```sql
DROP SCHEMA IF EXISTS pruebaDB;

-- Crear esquema 
CREATE SCHEMA pruebaDB
DEFAULT CHARACTER SET utf8mb4
DEFAULT COLLATE utf8mb4_general_ci;

USE pruebaDB;
```

### Genere una tabla con datos de prueba.
```sql
-- Crear tablas
CREATE TABLE Clientes (
    ID_Cliente INT PRIMARY KEY,
    Nombre VARCHAR(100) NOT NULL,
    Ciudad VARCHAR(50) NOT NULL
);

CREATE TABLE Ventas (
    ID_Venta INT PRIMARY KEY,
    Producto VARCHAR(50) NOT NULL,
    Cantidad INT NOT NULL CHECK (Cantidad > 0),
    Precio_Unitario DECIMAL(10, 2) NOT NULL CHECK (Precio_Unitario > 0),
    Fecha DATE NOT NULL,
    ID_Cliente INT NOT NULL,
    FOREIGN KEY (ID_Cliente) REFERENCES Clientes(ID_Cliente)
);

CREATE TABLE Pedidos (
    ID_Pedido INT PRIMARY KEY,
    ID_Cliente INT NOT NULL,
    Producto VARCHAR(50) NOT NULL,
    Cantidad INT NOT NULL CHECK (Cantidad > 0),
    Fecha DATE NOT NULL,
    FOREIGN KEY (ID_Cliente) REFERENCES Clientes(ID_Cliente)
);

INSERT INTO Clientes (ID_Cliente, Nombre, Ciudad) VALUES
(1, 'Ana Pérez', 'Machala'),
(2, 'Carlos Gómez', 'Loja'),
(3, 'Laura Sánchez', 'Loja'),
(4, 'Luis Ramírez', 'Cuenca');

INSERT INTO Ventas (ID_Venta, Producto, Cantidad, Precio_Unitario, Fecha, ID_Cliente) VALUES
(1, 'Laptop', 10, 800.00, '2024-01-15', 1),
(2, 'Smartphone', 20, 400.00, '2024-02-10', 2),
(3, 'Tablet', 15, 300.00, '2024-03-05', 3),
(4, 'Laptop', 5, 850.00, '2024-04-20', 4),
(5, 'Smartphone', 25, 390.00, '2024-05-10', 3),
(6, 'Tablet', 8, 310.00, '2024-06-15', 3);

INSERT INTO Pedidos (ID_Pedido, ID_Cliente, Producto, Cantidad, Fecha) VALUES
(1, 1, 'Laptop', 2, '2024-01-20'),
(2, 2, 'Smartphone', 3, '2024-02-12'),
(3, 3, 'Tablet', 1, '2024-03-10'),
(4, 4, 'Laptop', 1, '2024-04-25'),
(5, 1, 'Tablet', 2, '2024-05-18'),
(6, 3, 'Smartphone', 5, '2024-06-20');
```
### Desde Scala establezca la conexión a la base datos.
Primeramente se realizan las importaciones para la conexion de la base de datos. 
![image](https://github.com/user-attachments/assets/a22b47fb-25a4-4e40-8a59-363f619e1fe3)
#### Codigo en Scala 
```scala
package conSQL

import slick.jdbc.MySQLProfile.api._

import scala.concurrent.Await
import scala.concurrent.duration._

object ConexionMySQL {

  // Configuración de la base de datos
  val db = Database.forConfig("mysqlDB")

  // Esquema de las tablas
  class Clientes(tag: Tag) extends Table[(Int, String, String)](tag, "Clientes") {
    def id = column[Int]("ID_Cliente", O.PrimaryKey)
    def nombre = column[String]("Nombre")
    def ciudad = column[String]("Ciudad")

    def * = (id, nombre, ciudad)
  }

  class Ventas(tag: Tag) extends Table[(Int, String, Int, BigDecimal, String, Int)](tag, "Ventas") {
    def id = column[Int]("ID_Venta", O.PrimaryKey)
    def producto = column[String]("Producto")
    def cantidad = column[Int]("Cantidad")
    def precioUnitario = column[BigDecimal]("Precio_Unitario")
    def fecha = column[String]("Fecha")
    def idCliente = column[Int]("ID_Cliente")

    def * = (id, producto, cantidad, precioUnitario, fecha, idCliente)
  }

  val clientes = TableQuery[Clientes]
  val ventas = TableQuery[Ventas]

  def main(args: Array[String]): Unit = {
    try {
      // Consulta: Obtener todos los clientes
      val query = clientes.result
      val resultado = Await.result(db.run(query), 10.seconds)

      println("Lista de clientes:")
      resultado.foreach { case (id, nombre, ciudad) =>
        println(s"ID: $id, Nombre: $nombre, Ciudad: $ciudad")
      }

    } finally {
      db.close()
    }
  }
}

```
