# Laboratorio de Programación y Lenguajes 

## Parcial 22.Mayo.2023

Se le solicita a los alumnos del laboratorio de programación y lenguajes de la UNPAZ que realicen el desarrollo de la siguiente api.

* reservas  (/api/reservas) 
* vehiculos (/api/vehiculo)

Ambos recursos trabajan en conjunto y permiten gestionar la reservas de viajes que los solicitantes podrán realizar por distintas plataformas web.

Un solicitante podrá agendar una reserva y el sistema le asignará el primer vehículo que cumpla con las siguientes condiciones.


- [x] el vehículo debe estar habilitado.
- [x] el vechículo debe poder transportar a la cantidad de personas de la reserva.
- [x] el vehículo debe tener la autonomía suficiente para realizar el viaje sin detenerse.


## Consideraciones Generales

El desarrollo deberá realizarse en **_NodeJs_** utilizando la liberería **_express_**. El código **obligatoriamente**  deberá tener rutas y controladores para gestiona los request y response. Revisar la estructura de directorios propuesta en este repositorio.

Toda la información deberá gestionarse en memoria, es recomendable utilizar la librería **_nodemon_** para ir viendo los cambios sin necesidad de cortar el servidor de express y volverlo a levantar.

La entrega deberá ser en un repositorio de [github](https://github.com) donde el README.md deberá al menos incluir el nombre, apellido y dni del alumno, y escribir todas las cosas que asumieron para resolver el ejercicio o consideraciones que crean conveniente realizar.

### Vehículos

Realizar en endpoint **/api/vehiculos** que permita realizar la consulta total de los vehículos, más la creación, modificacion y consulta de un vehículo en particular a través de su número de patente.

* _GET_ **/api/vehiculos** - Recupera el array de vehículos.
* _GET_ **/api/vehiculos/:patente** - Recupera el vehículo de la patente pasada en el path de la URL como parámetro.
* _PUT_ **/api/vehiculos/:patente** - Permite la modificación de solo los atributos **_habilitado, capacidad y autonomiaKms_** del vehículo  de la patente pasada en el path de la URL como parámetro.
* _POST_ **/api/vehiculos** - Registra un nuevo vehículo.

En la registración de un nuevo vehículo _POST_ **/api/vehiculos** hay que considerar:
1. La patente debe tener 7 digitos. Es un punto adicional validar que sea de formato XX999XX.
2. Validar que el vehículo no se encuentre registrado previamente. La comprobación se debé hacer por la patente.
3. El vehículo nunca se ingresa habilitado, siempre tiene que ser **habilitado==false** y luego si es necesario se habilitará usando el _PUT_ **/api/vehiculos/:patente** para cambiar el valor de ese atributo.
4. La **capacidad** cantidad de personas a transportar debe ser un número de 1 a 10 como máximo.
5. La **autonomiaKms** debe ser un numero > 0. 

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

### Reservas

Realizar en endpoint **/api/reservas** que permita realizar la consulta total de las reservas, la creación, borrado y consulta de un reserva a través del identificador único de reserva que genera la api. 

* _GET_ **/api/reservas** - Recupera el array de reservas.
* _GET_ **/api/reservas/:id** - Recupera la reserva con el id pasado en el path de la URL como parámetro.
* _DELETE_ **/api/reservas/:id** - Permite el borrado del la reserva con el id pasado en el path de la URL como parámetro.
* _POST_ **/api/reserva** - Registra un nueva reserva.


En la registración de un nueva reserva _POST_ **/api/reserva** hay que considerar:

1. el id (identeficador único de la reserva) lo deberá calcular el sistema siguiendo una secuencia incremental.
2. Que la cantidad de personas a transportar sean un número entre 1 y 10.
3. Que la distancia nunca supere los 500 kms
4. La fecha debe tener 8 digitos. Es un punto adicional validar que sea de formato AAAADDMM.
    * 20230602 es una fecha válida
    * 20231402 no es una fecha valída porque sería para el mes 14. 
    * 20231236 Nno es una fecha válida porque no existe el día 36 del mes 12
5. El sistema deberá asociar solo el vehículo que cumple las condiciones de la reserva. 


## Condiciones de la reserva
- [x] el vehículo debe estar habilitado.
- [x] el vechiculo debe poder transportar a la cantidad de personas de la reserva.
- [x] el vehículo debe tener la autonomía suficiente para realizar el viaje sin detenerse.

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

El vehículo debera asociarse de acuedo a las condiciones. Si existe algún error en los datos enviados o no es posible asociar el vehículo el sistema deberá retornar el código correspondiente a un **bad request** indicando un mensajes descriptivo del error.

## PUNTO BONUS

Adicionalmente también se deberá poder buscar la última reserva de un cliente.

* GET **/api/reservas/search?cliente=XXXX** - Recupera la ultima la ultima reserva que realizo el cliente. En caso de no encontrar reservas deberá retornar "No encontrado"


## Ejemplos de creación de reservas

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

Si se hicera la siguiente reserva 
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
Si se hicera la siguiente reserva 

``` JSON
{
    "cliente": "Marcos Fiqueroa" ,
    "cantPersonas":6,
    "distancia": 315 ,
    "fecha": 20230601
}
```

El sistema debera informar un ** Bad Request**  porque no hay ningun vehículo habilitado que satisface la reserva.

### Ejemplo 4
Si se hicera la siguiente reserva 

``` JSON
{
    "cliente": "Marcos Fiqueroa" ,
    "cantPersonas":6,
    "distancia": 315 ,
    "fecha": 20231402
}
```

El sistema debera informar un ** Bad Request**  porque el formato de fecha es inválido.