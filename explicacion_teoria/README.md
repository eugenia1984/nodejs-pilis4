# Aplicación del clima

El Back End va a conectar con otro servidor, creando una consola interactiva, se hace un llamado a una API. <br>

Se hace una **petición HTTP**, con el producto de la misma se hace una segunda petición HTTP para ver el clima en ese lugar. <br>

Fetch API está en el navegador con JavaScript, pero no está en Nodejs. <br>

Será una aplicación de consola interactiva con las siguientes opciones: <br>

```
==============================
    Seleccione una opcion
==============================

? ¿Qué desea hacer ?
  1. Buscar ciudad
  2. Historial
  0. Salir
```
Voy a tener una lista de ciudades a elegir, y una vez elegida, voy a poder ver de la misma los siguientes datos: <br>
   * Latitud
   * Longitud
   * Temperatura
   * Temperatura Mínima
   * Temperatura Máxima
   * Como está el clima

Puedo ver en el **historial** las últimas seis búsquedas. <br>

Se conecta a la API de **Mapbox** (geocoding) para las coordenadas y a la API **Open Weather Map** para la información dle clima. <br>

---

## Inicio del proyecto

El archivo principal es : **index.js** donde tengo mi función **main**. <br>

En  el directorio **helpers** tengo mi archivo **inquirer.js**. <br>

También voy a tener mi paquete **colors** dentro del directorio **node_modules** . <br>

---

## Menú de la aplicación

El menú va a tener: <br>

```
1. Buscar ciudad
2. Historial
0. Salir
```

En **index.js** voy a importar: <br>
```
const { leerInput, inquirerMenu, pausa } = require('./helpers/inquirer');
## Consumo de APIs
```

Tengo mi constante **main** que es una arrow function aíncrona ( **async** ) . <br>
En la constante **busquedas** voy a instanciar de **Busquedas()** y tengo la variable **opt**. <br>
Porque trabajo con **clases** para poder compartir argumentos y variables entre los métodos. <br>
Por eso tengo mi directorio **models** con el archivo **busquedas.js**.<br>

Voy a tener un **do while**, mientras que la opción sea distinta de 0. <br>

Y al final inicializo la función **main()**. <br>

---

## Llamadas HTTP hacia servidores externos

Hay que hacer la petición HTTP para recibir de algún end point la información. <br>

Al ser Node.js no se puede utilizar **fetch**, por lo que hay que usar un **paquete de terceros**. <br>
Uno es **request**, aunque está deprecado, pero se sigue usando, lo que si es que usa muchos callbacks. <br>
Otro es **fetch** que nos permite trabajar con fetch como en el Front End, pero le faltan características. <br>
El que voy a utilizar es **AXios**. <br>


---

## Paquete request - superficialmente

---

## Paquete Axios


**Axios** trabaja en base a **promesas**, con then, try, catch. Se puede crear una instancia y configurarlo.<br>

En **busquedas.js** importo **axios** . <br>

Me creo una constante **intance** que va a ser instancia de **axios** por lo que uso **.create()**. <br>
Tengo la base de la URL : **https://api.mapbox.com/geocoding/v5/mapbox.places/** que es un **.json**. <br>

Todo lo que viene a continuación del **?** son los **params** y los defino mediante **get paramsMapbox**. <br>

Uso un **try** y **catch** . <br>

---

## Variables de entorno

Son globales a lo largo de toda la aplicación. <br>
En ellas puedo tneer mis API key. <br>

Voy a usar el paquete **dotenv** . <br>

En mi index.js: <br>

```
require('dotenv').config()
```

Tengo mi archivo **.env** . <br>

```
 get paramsMapbox() {
        return {
            'access_token': process.env.MAPBOX_KEY,
            'limit': 5,
            'language': 'es'
        }
    }
```

En produccion  **.env** no se sube a GitHub, algunos ponene axample.env y ponene una key irreal, el archivo .env lo evitan de subir al repositorio, ya que tiene el token de acceso. Se sube un ejemplo para que la gente sepa qué hacer. <br>

---

## Mapbox places para obtener lugares por nombre

Quiero tomar los resultados y solo retornar el **id** , el **place_name** y el **center**que trae la longitud y la latitud. <br>

Voy a extrae con **resp.data.features** y lo paso por el método **.map()** . <br>
Dentro voy a regresar un **objeto de forma implícita** que me de:
   * id
   * nombre
   * longitud (que la obtengo de la primer posicion en lugar.center)
   * latitud (que la obtengo de la segunda posición de lugar.center)

En **inquirer.js** voy a tener la constante **listarLugares** que es un array de lugares. <br>
Tengo el **unshift** para que agrewgue la opción 0 de Cancelar y las preguntas. <br>
Debo exportar **listarLugares** y en **index.js** lo importo para utilizarlo en el case 1 (buscar ciudad) guardándolo en la constante **id** (asi ya selecciono el lugar). <br>

Creo la constante **lugarSel** y utilizo le método **.find()** para que el id sea igual al que la persona busco, asi encuentro el lugar. <br>

Entonces en la parte de mostrar el resultado: <br>

```
 console.log('Ciudad:', lugarSel.nombre.green );
console.log('Latitud:', lugarSel.lat );
console.log('Longitud:', lugarSel.lng );
```

---

## Uso de OpenWeather para obtener el clima

Es un API, se puede usar buscando una ciudad y tendré los datos del clima. <br>

Lo que interesa es el **current weather data** que está basado en la **longitud** y la **latitud** por las **coordenadas geográficas**.
Copio en entry point y lo paso por **Postman** para ve que esté todo bien. <br>
Veo la información JSOn como un objeto de JS y veo que en **main** tengo los datos de la temperatura, pero los tengo en Kelvin, si al API le envio las unidades y me lo pasa a grados Celsius (**units** : **metric**) y puedo confirgurar el idioma a español, para cuando necesito cómo está el clima (**lang** : **es**). <br>

La latitud y la longitud son mandados como parámetros en la URL.

## Obtener información dle clima, del lugar seleccionado

En **busquedas.js** creo un nuevo método asíncrono llamado **climaLugar** que recibe la latitud y la longitud. Va a tener un try (creo la instancia de axios, con la respuesta me extraigo la data) / catch para manejar el error. Y retorno un objeto con la descripción de cómo está el clima, la temperatura mínima, la temperatura máxima y la temperatura normal. <br>

Creo la constante **clima** que me va a traer toda esa información y la muestro. <br>

---

## Aplicación de consola con historial

**Persistencia en las búsquedas** <br>

En **busquedas.js** voy a crear el método **agragarHistorial**. <br>

En **index.js** lo guardo en base de dato. Y en el caso 2 (el historial) lo recorro con un .forEach.<br>

En **busqeudas.js** tengo qeu hacer una validación para no tener valores duplicados. <br>
Primero chequeo que existe con **.includes()**, si existe lo guardo y si no existe lo agrego. <br>
Y hay que guardar en base de datos y leerlo. Para esto me creo los métodos: **guardarDB** y **leerDB**. Para esto voy a necesitar **File System** y me creo la constante llamada **payload** que guarda el historial. <br>

---


### Leer de archivos JSON

En **busqeudas.js** tengo el método **leerDB**, donde si no existe no se hace nada, pero si existe tengo la constante **info** para tener la información, recordar mandar en **encoding** con UTF-8. Luego tengo la constante **data** para parsear la información. Y solo voy a cargar el historial de la data. <br>

```
 leerDB() {

        if( !fs.existsSync( this.dbPath ) ) return;
        
        const info = fs.readFileSync( this.dbPath, { encoding: 'utf-8' });
        const data = JSON.parse( info );

        this.historial = data.historial;
    }
```

Y luego hay que crear el **getter** con el historial capitalizado, utilizando los métodos **.split** , **.map** (para cambiar la primer letra a mayúscula) , **.substring** (para cumarle el resto de la palabra en minuscula) y **.join** (para volver a juntarlas): <br>

```
get historialCapitalizado() {
        return this.historial.map( lugar => {

            let palabras = lugar.split(' ');
            palabras = palabras.map( p => p[0].toUpperCase() + p.substring(1) );

            return palabras.join(' ')

        })
    }
```

En **busquedas-js** en mi método **agregarHistorial** quiero que solo muestre las últimas seis búsquedas, por lo que : <br>

```
this.historial = this.historial.splice(0,5);
```
