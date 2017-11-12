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
 - [Aspectos Positivos del Sprint y del Equipo](#aspectos-positivos-del-sprint-y-del-equipo)
 - [Aspectos a Mejorar del Sprint y del Equipo](#aspectos-a-mejorar-del-sprint-y-del-equipo)

## Introducción

En el Sprint 3 el trabajo se enfocara en el atributo de calidad Disponibilidad, para esto se realizó la priorización de una serie de ASR que se consideraron de gran importancia tanto para los Stakeholder como para el diseño de la arquitectura. Se diseñará un experimento de arquitectura con el objetivo de validar la propuesta de diseño del equipo para satisfacer los ASR.

## Requerimientos ASR

## Nota Inicial

Para mayor entendimiento de los requerimientos críticos de arquitectura

Cotización: Elemento de negocio que consta de un encabezado y un detalle. 

Encabezado: Datos del cliente, total de la cotización
Detalle: Datos del producto, cantidad, y monto de la línea

El proceso completo nace con el vendedor de concesionario quien ingresa datos mínimos del cliente, y las referencias de el o los vehículos de interés del cliente.

Al enviar la cotización, el sistema almacena el cliente si este no existe o complementa la información del mismo en la cotización, revisa producto por producto del detalle y adiciona el monto de cada linea de la cotización y por ultimo totaliza la cotización y envía de regreso al vendedor de concesionario

### ASR 5
COMO Gerente de Ventas CUANDO un vendedor de concesionario en la feria del automóvil o día de la madre, DADO QUE el sistema opera con sobrecarga QUIERO que todas las cotizaciones sean respondidas por el sistema ya que en estas fechas la probabilidad de venta es tres veces mayor que en cualquier otra fecha del año PARA poder presentar al cliente rápidamente el precio y los descuentos del vehículo de su interés  ESTO DEBE ser respondido el 99,999% de la veces lo cual quiere decir que en todo el fin de semana de feria solo se permite que 0,001% de las cotizaciones en estado de falla 

## Arquitectura Propuesta

### General
La propuesta de arquitectura de este sprint comprende la unificación de las decisiones tomadas en el sprint 1 para favorecer la Latencia con las que el equipo ha tomado para favorecer la Escalabilidad. Las tácticas a aplicar tienen por objetivo el trabajar en conjunto con el estilo arquitectural LMAX para que el sistema se comporte de forma adecuada en momentos de gran estrés.

### Estilo de Arquitectura
Por descomposición dominante nuestros estilos de arquitectura continuar siendo LMAX y LAMBDA definida en el Sprint 1

### Tácticas de Arquitectura

#### Táctica Software

##### Detect Faults - Monitor 
Este monitor verifica cada x segundos que los componentes de BusinessLogic así como su HW asociado estén activos y en linea. Verifica tanto los BusinessLogic, como la conexión a la base de datos de productos y de clientes. También recibe el error de los componentes en caso de falla y lo registra en la base de datos para seguimiento.

##### Caída Graciosa de los componentes
El componente será extendido para que en caso de falla haga cierre formal de todos los recursos adquiridos, de forma que después de su caída el sistema quede en un estado adecuado y no vaya degradando paulatinamente el estado de salud general del sistema. Además de informar el error al monitor de fallas para registro y seguimiento.

#### Táctica Hardware
##### Recover from faults - Redundancia Activa
Con el fin de garantizar la respuesta a los clientes, se usara la redundancia activa de 3 componentes de HW y SW del BusinessLogic de forma que en el evento de falla de uno o dos de ellos, el sistema siga funcional
 
##### Reintroduction - State Resynchronization 
Un elemento de BusinessLogic caído se re-inicia y empieza a operar en conjunto con la redundancia activa del punto anterior. Esta táctica va en conjunto con la redundancia activa y la caída graciosa del componente para que su reintroducción sea suave sin ir degradando el sistema bajo caídas sucesivas

##### Load Balancing 
Se usará el loadbalancer para que balancee las peticiones entre los componentes de BusinessLogic Activos 1 2 o 3 según lo especificado en la redundancia Activa


### Vistas

#### Vista Funcional

Desde un punto de vista funcional observamos que los puntos críticos en la disponibilidad están asociados con los componentes de BusinessLogic y la disponibilidad del catalogo de productos y el subistema de clientes.
**Ver componentes y monitor de salud de componentes. Círculo Azul**. El monitor de saludo usa una táctica de arquitectura Echo - Ping a los diferentes componentes y ademas tiene como función recibir los errores de los componentes y registrarlos en la base de datos

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/DisponibilidadVistaFuncional.png]]


#### Vista de Despliegue

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/Diagrama despliegue Sprint 3.png]]

## Puntos de Sensibilidad

La decisión critica de diseño está en lograr una disponibilidad del 99,999% durante el fin de semana de la feria del automóvil y día de la madre, para esto debemos lograr que en caso de falla de uno de los componentes clave bien sea de Software o de Hardware asociado de la arquitectura como:
El Business Logic
El Catalogo de Productos
El sub-sistema de clientes
Se lleve un seguimiento de la caída vía los monitores y que estos a su vez estén disponibles y se puedan recuperar sin intervención humana todo el tiempo ya que por los requerimientos de los stakeholders solo podemos fallar en el 0,001% de las cotizaciones. 

## Experimento

## Problema a Resolver
Nuestro experimento consiste en asegurar la disponibilidad del ambiente especialmente en sobrecarga en la feria del automóvil. En tal fecha la disponibilidad debe ser del 99,999% por tal motivo efectuaremos ajustes tanto de HW como de SW para garantizar dicho requerimiento.

## Diseño del Experimento
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/ExperimentoDisponibilidad.PNG]]

### A nivel de Hardware
Redundancia Activa: Iniciaremos con 3 componentes de HW y SW del BusinessLogic los cuales estarán recibiendo peticiones todo el tiempo

LoadBalancing: Para distribuir las peticiones entre los 3 componentes de manera equitativa aun cuando algunos de ellos no están en linea en algún momento del tiempo

### A nivel de Software

Caída Graciosa de los Componentes: Atraparemos todas las excepciones para gestionarlas adecuadamente y además informar del hecho mediante el envío de un mensaje a una cola asíncrona que se comunica con los monitores para su seguimiento y control. Además limpiar y cerrar todas las conexiones antes de caer para no degradar paulatinamente el sistema

Monitor: Componente que valida el estatus tanto del Hardware como del Software asociado a los BusinessLogic y los componentes críticos asociados como la conexión a la base de datos de productos y de clientes. De forma que al notar una situación o posible causa de falla (Fault) el administrador pueda sacar el componente de linea y volver a introducirlo sin afectar al usuario final. También este componente es responsable de recibir y gestionar los mensajes de caída de los componentes y registrarlo en la base da datos para análisis de fallas del sistema.

Replicar Información: En nuestro análisis funcional nos dimos cuenta de que el catálogo por si solo es un punto de concurrencia y de falla de todo el sistema, sin acceso al catálogo de productos ningún proceso puede funcionar independientemente de los robustos que sean sus componentes de HW y SW. Por este motivo como el catálogo de productos en una entidad de información relativamente estática se puede replicar en memoria cache en cada uno de los nodos donde se ejecutan los BusinessLogic de forma que cada nodo del grupo de escalamiento sea autónomo en su ejecución (Ver Vista de Despliegue)

El siguiente método muestra como se obtiene la información del producto desde cache o desde la DB cuando la información no se a cargado.
```javascript
getProduct = function (productId, done) {
    let key = "product/" + productId;
    cacheManager.get(key, function (product) {
        if (process.env.CCVAPP_REDIS_URL && product) {
            done(product);
        } else {
            productPersistence.getProduct(productId, (productDB) => {
                cacheManager.set(key, productDB);
                done(productDB);
            });
        }
    });
};
```

### Prueba del Experimento  
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/ExperimentoDisponibilidad2.PNG]]
Intencionalmente haremos fallar uno de los nodos de busineslogic y seguiremos enviando la carga al sistema de acuerdo al caso de negocio y medir los resultados de disponibilidad, luego haremos fallar dos de los componentes y repetiremos el experimento para validar que aun en esas condiciones podemos seguir operando. Estos nodos que caen debe reiniciarse por si solos y reintroducirse en el sistema sin afectar la operación 

Por ultimo sacaremos por unos segundos la conexión al catálogo de productos y veremos el impacto a los nodos del BusinessLogic contando con que ellos tienen la información de productos en la memoria cache de cada nodo

### Hardware / SW usado

Como componente de SW adicional a los ya usados con los sprints anteriores:
* New Relic
* JMeter
* NodeJs
* AWS
* RDS
* LoadBalancer

Adicionamos la herramienta de HappyApps.io para obtener medidas de disponibilidad (99,999%) e información de fallos de la disponibilidad del sistema 
* HappyApps.io

## Resultados


[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/ExperimentoDisponibilidad3.PNG]]
Para la primera iteración del experimento se obtuvieron resultados satisfactorios, para esta se utilizó la configuración expuesta anteriormente de 3 instancias, bajo redundancia activa, las cuales estan encargadas de atender las peticiones. Cada una de estas instancias sufrió una caída programada y fueron reintroducidas con EPM2, esto con el fin de medir el nivel de respuesta del sistema ante este tipo de situaciones.

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/happyapps (2).PNG]]

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/happyapps (3).PNG]]

Sobre los 30 segundos que se repartieron las 500 peticiones, se lograron satisfacer todas las peticiones y lograr una media de 375 ms.

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/happyapps (4).PNG]]

Ademas mediante el uso de happyApps monitoreamos la disponibilidad del servicio durante el experimento, lo cual demostró que a pesar de las caídas programadas de los componentes de Business Logic, el servicio siempre estuvo disponible para atender todas las peticiones simuladas.

## Análisis de Resultados y Respuesta al problema

Para el experimento solo se realizó una iteración puesto que se observo un comportamiento favorable del sistema bajo las condiciones establecidas para dar visto buenos a las decisiones arquitecturales. Desde la perspectiva de hardware son de gran importancia la redundancia activa y la re sincronización de componentes pues estos nos permitirán en gran medida responder a las peticiones bajo condiciones difíciles. Por el lado de software el monitoreo de los componentes es crucial y seria importante el poder involucrar un sistema de alertas en el futuro para complementar el esquema definido. 

Dado los resultados positivos, con una media de atención de 375 ms consideramos que las desiciones arquitecturales tomadas son correctas y no se realizaran cambios a la propuesta. 
## Retrospectiva

[Video Arquitectura y Puntos de Sensibilidad](https://youtu.be/T1p-Afn1xjs)  

[Video Diseño del Experimento](https://youtu.be/X0nGhSMfRyQ) 

[Video Resultados del Experimento](https://youtu.be/c_mXIl2CUkc) 

## Aspectos Positivos del Sprint y del Equipo

En este sprint desde el punto de vista del equipo con la dinámica que trajimos del sprint 2 el equipo fluyó mayor sincronía, y asertividad. 

Dividimos el equipo en 2 grupos: Frente Funcional y Frente Técnico para iniciar actividades, pero teniendo nuestro Daily Sprint para sincronizar los equipos, cuando el equipo funcional terminó sus labores, paso a apoyar el equipo técnico para culminar el experimento

## Aspectos a Mejorar del Sprint y del Equipo

Creemos como equipo que aun nos falta mucho mas conocimiento técnico de NodeJS para hacer una manejo optimo de Threads, Memoria Compartida, y sincronización de procesos para llevar el estilo de arquitectura LMAX a su máxima expresión. 

 

 