# Proyecto-Final-Programacion-2
Desarrollo de una aplicación web con Java, Spring Boot,  Hibernate y MySQL/MariaDB

# Sistema de Alquiler de Herramientas

TP Final - **Programacion 2**. Aplicacion web empresarial para gestionar el
alquiler/prestamo de herramientas: clientes, catalogo de herramientas, control
de stock, registro de prestamos con calculo automatico de importe y devoluciones.

## Tecnologias

- **Java 17**
- **Spring Boot 3.2.5** (Web MVC, Data JPA, Validation, Thymeleaf)
- **Hibernate / JPA** (`jakarta.persistence`)
- **MySQL** (por defecto) o **MariaDB** (alternativa)
- **Thymeleaf** + CSS para la interfaz
- **Maven** como gestor de dependencias

## Arquitectura en capas (SOLID)

```
com.empresa.alquilerherramientas
 |-- modelo         -> entidades JPA + enums (Cliente, Herramienta, Prestamo, ...)
 |-- dto            -> objetos de formulario (PrestamoForm, DetalleForm)
 |-- repositorio    -> interfaces JpaRepository (acceso a datos)
 |-- servicio       -> interfaces de la logica de negocio
 |     '-- impl      -> implementaciones (toda la logica vive aqui)
 |-- controlador    -> controladores web (reciben peticiones, no calculan)
 |-- excepciones    -> excepciones propias + GlobalExceptionHandler
 '-- config         -> datos iniciales (seed) y conversores
```

Cada servicio se define como **interface + implementacion** para aplicar el
principio de inversion de dependencias (DIP): los controladores dependen de la
abstraccion, no de la clase concreta. Adaptado a Spring Boot (en Hibernate nativo serian los controladores
`InterControl<T>` con `Session`/`Transaction` manuales; aqui lo resuelven
`JpaRepository` + `@Transactional`).

## Reglas de negocio implementadas

1. El precio por dia de una herramienta debe ser mayor que cero.
2. El stock (cantidad total) no puede ser negativo.
3. No se puede dar de baja una herramienta con unidades prestadas.
4. Un prestamo debe estar asociado a un cliente.
5. Un prestamo debe incluir al menos una herramienta.
6. La cantidad solicitada de cada herramienta debe ser mayor que cero.
7. No se puede prestar mas stock del disponible (control de stock).
8. La cantidad de dias debe ser mayor que cero.
9. El importe de cada linea = `cantidad x precio_por_dia x dias`.
10. El importe total del prestamo se calcula **automaticamente**.
11. Al registrar un prestamo se **descuenta** el stock disponible.
12. Al registrar la devolucion se **restituye** el stock.
13. El estado de la herramienta cambia a NO_DISPONIBLE cuando su stock llega a 0.

Toda esta logica vive en la capa de servicio (`PrestamoServiceImpl`,
`HerramientaServiceImpl`), nunca en los controladores.

## Como ejecutar en IntelliJ IDEA

1. **Base de datos**: tene MySQL (o MariaDB) corriendo en `localhost:3306`.
   La base se crea sola gracias a `createDatabaseIfNotExist=true`, o podes
   correr `database.sql` a mano desde MySQL Workbench.
2. **Abrir el proyecto**: en IntelliJ, *File -> Open...* y selecciona el
   archivo **`pom.xml`** (abrir como proyecto). IntelliJ bajara las dependencias
   de Maven automaticamente.
3. **Configurar credenciales**: en `src/main/resources/application.properties`
   ajusta `spring.datasource.username` y `spring.datasource.password` segun tu
   instalacion de MySQL.
   - Para usar **MariaDB** en vez de MySQL: comenta el bloque de MySQL y
     descomenta el bloque de MariaDB.
4. **Ejecutar**: corre la clase `AlquilerHerramientasApplication` (boton Run).
5. Abri el navegador en **http://localhost:8080**.

La primera vez que arranca, la app carga datos de ejemplo (categorias,
herramientas, clientes y empleados) si la base esta vacia.

## Modulos / pantallas

- **Inicio**: panel con indicadores (clientes, herramientas, prestamos activos).
- **Clientes**: alta, baja (logica), edicion y busqueda.
- **Herramientas**: alta, edicion, baja y busqueda; asignacion de categoria.
- **Stock**: inventario con total, disponible y prestado por herramienta.
- **Prestamos**: registro de un prestamo con varias herramientas y calculo del
  importe; detalle de cada prestamo.
- **Devoluciones**: lista de prestamos activos y registro de la devolucion
  (restituye stock).

## Requisitos del entorno

- JDK 17 (recomendado 17 o 21; **no** usar JDK muy nuevos como 26, que no
  son compatibles con esta version de Hibernate).
- Maven (el que trae IntelliJ sirve).
- MySQL 8+ o MariaDB 10+.
