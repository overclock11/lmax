# Tabla de contenidos
- [Introducción](#introducción)
- [Requerimientos ASR](#requerimientos-asr)
 - [Arquitectura Propuesta](#arquitectura-propuesta)
   - [General](#general)
   - [Estilo de Arquitectura](#estilo-de-arquitectura)
   - [Tácticas de Arquitectura](#tácticas-de-arquitectura)
    - [Vistas](#vistas)
      - [Vista Funcional](#vista-funcional)
      - [Vista de Despliegue](#vista-de-despliegue)
      - [Vista de Concurrencia](#vista-de-concurrencia)
- [Puntos de Sensibilidad](#puntos-de-sensibilidad)
 - [Experimento](#experimento)
   - [Problema a Resolver](#problema-a-resolver)
   - [Diseño del Experimento](#diseño-del-experimento)
   - [Implementación del Experimento](#implementación-del-experimento)
   - [Resultados](#resultados)
   - [Análisis de Resultados y Respuesta al problema](#análisis-de-resultados-y-respuesta-al-problema)
- [Retrospectiva](#retrospectiva)
- [Conclusiones](#conclusiones)


## Introducción

En el Sprint 2 el trabajo se enfocara en el atributo de calidad Escalabilidad, para esto se realizo la priorización de una serie de ASR que se consideraron de gran importancia tanto para los Stakeholder como el diseño de la arquitectura. Se diseñara un experimento de arquitectura con el objetivo de validar la propuesta de diseño del equipo para satisfacer los ASR.

## Requerimientos ASR

## Nota Inicial

Para mayor entendimiento de los requerimientos críticos de arquitectura

Cotización: Elemento de negocio que consta de un encabezado y un detalle. 

Encabezado: Datos del cliente, total de la cotización
Detalle: Datos del producto, cantidad, y monto de la línea

El proceso completo nace con el vendedor de concesionario quien ingresa datos mínimos del cliente, y las referencias de el o los vehículos de interés del cliente.

Al enviar la cotización, el sistema almacena el cliente si este no existe o complementa la información del mismo en la cotización, revisa producto por producto del detalle y adiciona el monto de cada linea de la cotización y por ultimo totaliza la cotización y envía de regreso al vendedor de concesionario

### ASR 11
COMO Gerente de mercadeo CUANDO para las fechas especiales (Dias de la Madre, Fines de semana de feria), DADO QUE el sistema opera con sobrecarga QUIERO poder registrar y dar respuesta a todas las cotizaciones hechas por los vendedores de concesionario ESTO DEBE suceder en menos de 2 segundos, dado que hay 1000 vendedores adicionales para un total de 1500 con clientes simultáneamente haciendo cotizaciones. Se estima que 1/3 de los vendedores envían cotizaciones en el mismo momento.

### ASR 14 
COMO Gerente de mercadeo CUANDO diferentes concesionarios hacen campañas particularmente en fines de semana DADO QUE el sistema opera normalmente QUIERO poder registrar y dar respuesta a todas las cotizaciones hechas por los vendedores de concesionario ESTO DEBE suceder en menos de 1 segundos, dado que los requerimientos de cotizaciones sobre el sistema pueden aumentar entre un 10% y un 20% sin previo aviso dadas estas campañas sorpresa. En condiciones normales hay 500 vendedores accediendo el sistema, pero en ocasiones cuando se cruzan campañas en varios concesionarios los vendedores pueden aumentar entre 550 y 600 vendedores activos. Se estima que 1/3 de los vendedores envían cotizaciones en el mismo momento.

## Arquitectura Propuesta

### General
La propuesta de arquitectura de este sprint comprende la unificación de las decisiones tomadas en el sprint 1 para favorecer la Latencia con las que el equipo ha tomado para favorecer la Escalabilidad. Las tácticas a aplicar tienen por objetivo el trabajar en conjunto con el estilo arquitectural LMAX para que el sistema se comporte de forma adecuada en momentos de gran estrés.

### Estilo de Arquitectura
Por la descomposición dominante ya estamos comprometidos con los estilos de arquitectura

#### LMAX 
Para respuesta rapida a vendedores y jefes de taller.

#### LAMBDA 
Para proceso rapido de información de forma que las directivas de CCV puedan tomar decisiones 

### Tácticas de Arquitectura

#### Scaling Out 
Permitir auto-escalar instancias del Business Logic para procesar mas cotizaciones por unidad de tiempo 

#### Replicación de componentes de negocio
Replicar el componente de procesamiento de negocio y el catalogo de productos por región para poder cumplir con los picos de demanda

#### Scaling Up
Para aumentar la capacidad del HW que alberga los Disruptores de entrada y de salida para que puedan soportar el aumento en el tamaño de las colas de 200 a 500 para los días de la madre y ferias de vehículos
200 slots para dias normales (Incluyendo los aumentos súbitos de demanda ASR14)
a
500 slots para el dia de la madre y feria automotriz
Este aumento temporal de HW se debe realizar tres días antes del evento y durar hasta una semana después del evento dado que las promociones viven un poco mas que el evento (Pre-Feria, Post-Feria) (ASR11)

### Vistas

#### Vista Funcional

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/VistaFuncionalEscalamiento.png]]

#### Vista de Despliegue

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/DespliegueEscalabilidad.png]]

## Puntos de Sensibilidad

La decisión critica de diseño esta en combinar el estilo de arquitectura LMAX para garantizar el tiempo de respuesta con el autoescalamiento horizontal de HW y replicación de componentes de negocio en ese nuevo HW y aumento de la capacidad de los disruptores para poder reaccionar a los eventos especiales (Día de la madre, Feria del Automóvil), ya que estas generan un aumento considerable en la carga del sistema.

## Experimento

### Cuestionamiento a Resolver

Por medio de este experimento queremos eliminar la incertidumbre y validar el punto de sensibilidad de escalar LMAX replicando en HW y SW el componente de negocio BusinessLogic por medio de **auto-escalamiento** y **aumentando de los disruptores en capacidad** de forma que soporte **500 cotizaciones concurrentes**

### Diseño del Experimento
El experimento se basa principalmente en validar los puntos de sensibilidad expuestos en la vista de despliegue basando la estrategia principalmente en: 
1) Auto-escalamiento del componente Business Logic tanto en HW como en SW
2) Aumento de capacidad de los disruptores que redundan en escalamiento de HW
La gráfica siguiente muestra los dos focos de atención del experimento:
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/ExperimentoEscalabilidad2.png]]

Los pasos para el experimento son:

1) Aumentar el tamaño de los disruptores 
2) Crea un grupo de escalamiento en AWS con un umbral de ejecucion
3) Se adicionan nuevas maquinas a dicho grupo de escalamiento
4) En cada maquina se lanza el proceso de BusinessLogic que inicia de inmediato lecturas de los disruptores
5) Se va generando carga aumentando en bloques de a 100 clientes concurrentes hasta encontrar el "Knee" punto de quiebre del sistema donde se dispara la qatencia a puntos no aceptables
6) Validar que dicho "Knee" este por encima de las 500 cotizaciones concurrentes que es el evento que queremos validar con el presente experimento. 

### Implementación del Experimento

#### Hardware y Software

El Hardware y Software usado consta de:
Aplicación: 
1) NodeJS como Application Server
2) Servicio de Amazon para base de datos (RDS)

Todo montado en servidores cloud AWS 

Simulación de Carga:
1) New Relic como componente del servidor para la generación de métricas
2) JMETER como cliente para simular la carga de trabajo

### Resultados

Iteración 1
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/ExperimentoEscalabilidad.png]]

La primera iteración del experimento solo replicamos el componente BusinessLogic y como resultado obtuvimos los resultados siguientes:

El resultado no fue optimo para concurrencia de 500 cotizaciones concurrentemente puesto que se nos creo una situación de contención en los disruptores generando bloqueo de la cola y tiempos de espera de todos los BusinessLogic replicados mientras el BusinessLogic activo tomaba o insertaba en los disruptores de entrada y salida. Adicionalmente apoyados con New Relic encontramos que había unos Queries sub-optimos a la base de datos que generaban aun mas contención.
 
Ver Gráfica Queries Sub-Optimos:
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/NewRelicQueries.png]]

Ver Gráfica 2 Contención Input Disruptor
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/ContencionDisruptorNewRelic.png]]

RESULTADO: ****ITERACION 1 NO ACEPTABLE****

Motivo por el cual redefinimos nuestro experimento:

Iteración 2
Modificamos los queries sub-optimos y ampliamos el grupo de escalamiento para que contenga los dos disruptores y el business logic como se representa en la gráfica siguiente:

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/ExperimentoEscalabilidad2.png]]



### Análisis de Resultados y Respuesta al problema

Para realizar pruebas en el experimento se planteó un set de pruebas en el que se mide la latencia promedio de la respuesta al guardar y calcular una cotización haciendo variación en el número de usuarios concurrentes entre 10, 50, 100, 200, 300, 400 y 500 usuarios concurrentes, siendo este último valor nuestro objetivo.
Esta prueba se repitió para cada iteración agregando instancias del componente a replicar en el ambiente hasta 4 instancias replicadas.
Se configuraron en el código un conjunto de eventos asociados a New Relic que permiten medir el tiempo y arrojar métricas de operaciones claves en los componentes.
```javascript
function processRequest() {   
    let bgGetRequest = nr.createBackgroundTransaction('SalesBLProcessor:processRequest', function () {
        getRequest(function (nextQueueElement) {
            if (nextQueueElement && nextQueueElement.request) {
                let quote = nextQueueElement.request;
                quoteProvider.createOrUpdateCustomer(quote.customer);
                buildQuoteDetail(quote, function (result) {
                    pushResponse(quote, function () {
                        nr.endTransaction();                      
                    });
                    saveQuote(quote,function (result) {                      
                    });
                });

            }
        });
    });
    bgGetRequest();
}
```

###  Iteración 1
 Se probó la arquitectura teniendo un único componente disruptor y replicando el componente Bussines Logic, se encontró que por problemas de contención en el componente disruptor la latencia pasaba a valores inaceptables al pasar los 50 usuarios concurrentes y tal como se mencionó anteriormente se descartó esa configuración.
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/LMAX1.PNG]]


###  Iteración 2
 Se probó la arquitectura replicando una instancia EC2 que contiene a todos los componentes de la arquitectura inicial,  Contoller, Disruptor y BusinessLogic, además se incluyó un balanceador de carga que permitiera distribuir las peticiones entre el número de instancias que se agreguen al grupo de escalamiento,  en esta prueba los resultados también fueron inaceptables, pero gracias a la instrumentación de New Relic se logró identificar que todas las inserciones tenían una verificación de registro incluido hecha a partir de un select lo cual generaba bloqueos en el acceso a la Base de Datos Postgres en RDS.
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/LMAX2.PNG]]


###  Iteración 3
Se corrige el problema identificado en la anterior iteración, los tiempos mejoran, pero debido a la comunicación entre los componentes al interior de cada instancia replicada, aún se mantienen tiempos de latencia que están en valores inaceptables.
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/LMAX3.PNG]]
###  Iteración 4
Buscando un comportamiento de referencia en una configuración diferente a las anteriores se realiza el set de pruebas sobre una arquitectura compuesta por un balanceador de carga y un conjunto de instancias en las cuales se realiza la misma operación de cálculo de cotización y su respectiva persistencia expuesto mediante servicio REST por cada una de las instancias.
En este experimento variando la misma cantidad de instancias se logra obtener tiempos de latencia aceptables, pero modificando completamente la arquitectura inicialmente planteada.

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/LMAXReferencia.PNG]]


Para realizar un resumen se hace un contraste de lo sucedido a medida que se incrementaban instancias para cada uno de los casos, estos fueron los resultados de la latencia promedio para 500 usuarios concurrentes:


[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/lmaxResumen.PNG]]



## Retrospectiva

### Arquitectura y Puntos de Sensibilidad

[Video Arquitectura y Puntos de Sensibilidad](https://youtu.be/in9C1BXz-b0)
## Conclusiones