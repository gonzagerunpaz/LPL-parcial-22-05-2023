# Laboratorio de Programación y Lenguaje - Parcial 22.Mayo.2023

Se le solicita a los alumnos del laboratorio de programación y lenguajes que realicen de desarrollo de las siguintes apis.

* reservas  (/api/reservas) 
* vehiculos (/api/vehiculos)

Ambas Apis trabajan en conjunto y permiten gestionar la reservas de viajes que los solicitantes podrán realizar por distintas plataformas web.

Un solicitante podrá agendar una reserva y el sistema le asignará el primer vehiculo que encuentre que cumpla con las siguientes condiciones

* el vehículo debe estar habilitado.
* el vechiculo debe poder transportar a la cantidad de personas de la reserva.
* el vehículo debe tener la autonomía suficiente para realizar el viaje sin detenerse.

## Vehiculos

Realizar en endpoint **/api/vehiculos** que permita realizar la consulta total de los vehiculos, más la creación, modificacion y consulta de un vehículo en particular a través de su número de patente.

* GET **/api/vehiculos** - Recupera el array de vehiculos
* GET **/api/vehiculos/:patente** - Recupera el vehiculo de la patente pasada en el path de la URL como parámetro.
* PUT **/api/vehiculos/:patente** - Permite la modificación de solo los atributos **[habilitado, capacidad y autonomiaKms]** vehículo  de la patente pasada en el path de la URL como parámetro.
* POST **/api/vehiculos** - Registra un nuevo vehículo

En la registración de un nuevo vehículo **POST** hay que considerar:
* La patente debe tener 7 digitos. Es un punto adicional validar que sea de formato XX999XX
* El vehiculo nunca se ingresa habilitado, siempre tiene que ser **habilitado==false** y luego si es necesario se habilitará usando el PUT para cambiar el valor de ese atributo.
* La **capacidad** cantidad de personas a transportar debe ser un número de 1 a 10 como máximo
* La **autonomiaKms** debe ser un numero >0 

### Estructura del objeto vehículo

``` JSON
    {
        "patente": String,
        "marca": String,
        "modelo": String,
        "habilitado": Boolean,
        "capacidad": Number,
        "autonomiaKms": Number
    }

```

## Reservas

Realizar en endpoint **/api/reservas** que permita realizar la consulta total de las reservas, más la creación, borrado y consulta de un reserva a través del identificador único de reserva que genera la api.

* GET **/api/reservas** - Recupera el array de reservas
* GET **/api/reservas/:id** - Recupera la reserva con el id pasado en el path de la URL como parámetro.
* DELETE **/api/reservas/:id** - Permite el borrado del la reserva con el id pasado en el path de la URL como parámetro.
* POST **/api/reserva** - Registra un nueva reserva


En la registración de un nueva reserva **POST** hay que considerar:

* el id (identeficador único de la reserva) lo deberá calcular el sistema siguiendo una secuencia incremental.
* Que la cantidad de personas a transportar sean un número entre 1 y 10
* Que la distancia nunca supere los 500 kms
* El sistema deberá asociar solo el vehículo que cumple las condiciones de la reserva.  
* La patente debe tener 8 digitos. Es un punto adicional validar que sea de formato YYYYDDMM, Ej 20230602 es una fecha válida, mientras que 20231402 no es una fecha validad porque sería para el mes 14. Otra fecha inválida sería 20231236 porque no existe el día 36 del mes 12


### Condiciones de la reserva
* el vehículo debe estar habilitado.
* el vechiculo debe poder transportar a la cantidad de personas de la reserva.
* el vehículo debe tener la autonomía suficiente para realizar el viaje sin detenerse.

### Estructura del objeto Reserva para los GET
``` JSON
    {
        "id": Number,
        "cliente": String,
        "cantPersonas": Number,
        "distancia": Number,
        "fecha": String,
        "vehiculo":     {
            "patente": String,
            "marca": String,
            "modelo": String,
            "capacidad": Number,
            "autonomiaKms": Number
        }
    }
```
### Estructura del BODY en el POST de reservas
``` JSON
{
    "cliente": String,
    "cantPersonas": Number,
    "distancia": Number,
    "fecha": String,
}
```

El vehículo debera asociarse de acuedo a las condiciones. Si existe algún error ne los datos enviados o no es posible asociar el vehículo el sistema deberá retornar el código correspondiente a un **bad request** indicando un mensajes descriptivo del error.

## Ejemplos

Si tuvieramos los siguientes vehículos hablitados

|Patente|Marca      |modelo|capacidad|autonomiaKms|
|-------|-----------|------|---------|------------|
|AD341PO|Chevrolet  |Onix  |    4    |    280     |
|NK808CR|Volkswagen |Suram |    5    |    240     |
|AA343OU|Toyota     |Etios |    3    |    320     |

### Ejemplo 1
Si se hicera la siguinte reserva 
```
{
    "cliente": "Gerardo Gonzalez" ,
    "cantPersonas":2,
    "distancia": 315 ,
    "fecha": 20230601
}
```
El vehículo asociado sería el 3ro. porque si bien los dos primeros cumplen con la cantidad de personas no cumplen la autonómia. 
La salida sería la siguiente:

``` JSON
    {
        "id": 1,
        "cliente": "Gerardo Gonzalez",
        "cantPersonas": 2,
        "distancia": 315,
        "fecha": "20230601",
        "vehiculo": {
            "patente": "AA343OU",
            "marca": "Toyota",
            "modelo": "Etios",
            "capacidad": 3,
            "autonomiaKms": 320
        }
    }
```
### Ejemplo 2

Si se hicera la siguinte reserva 
``` JSON
{
    "cliente": "Romina Suarez",
    "cantPersonas": 5,
    "distancia": 100,
    "fecha": "20230530",
}
```

El vehículo asociado sería el 2ro. porque es el único que cumple que puede transpotar a 4 personas y que la autonómia es mayor a la distancia solicitada

La salida sería la siguiente:
``` JSON
    {
        "id": 2,
        "cliente": "Romina Suarez",
        "cantPersonas": 5,
        "distancia": 100,
        "fecha": "20230530",
        "vehiculo": {    
            "patente": "NK808CR",
            "marca": "Volkswagen",
            "modelo": "Suram",
            "capacidad": 5,
            "autonomiaKms": 240
        }
    }
```

### Ejemplo 3
Si se hicera la siguinte reserva 

``` JSON
{
    "cliente": "Marcos Fiqueroa" ,
    "cantPersonas":6,
    "distancia": 315 ,
    "fecha": 20230601
}
```

El sistema debera informar un ** Bad Request**  porque no hay ningun vehículo habilitado que satisface la reserva

### Ejemplo 4
Si se hicera la siguinte reserva 

``` JSON
{
    "cliente": "Marcos Fiqueroa" ,
    "cantPersonas":6,
    "distancia": 315 ,
    "fecha": 20231402
}
```

El sistema debera informar un ** Bad Request**  porque el formato de fecha es inválido.