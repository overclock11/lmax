# Tabla de contenidos
- [Introducción](#introducción)
- [Requerimientos ASR](#requerimientos-asr)
 - [Arquitectura Propuesta](#arquitectura-propuesta)
   - [General](#general)
   - [Estilo de Arquitectura](#estilo-de-arquitectura)
   - [Tácticas de Arquitectura](#tácticas-de-arquitectura)
    - [Vistas](#vistas)
      - [Vista Funcional](#vista-funcional)
      - [Vista de Información](#vista-de-información)
      - [Vista de Despliegue](#vista-de-despliegue)
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

En el Sprint 1 el trabajo se enfocara en el atributo de calidad Latencia, para esto se realizo la priorización de una serie de ASR que se consideraron de gran importancia tanto para los Stakeholder como el diseño de la arquitectura. Se diseñara un experimento de arquitectura con el objetivo de validar la propuesta de diseño del equipo para satisfacer los ASR.

## Requerimientos ASR

### ASR1
COMO vendedor de concesionario CUANDO ingreso un modelo de vehículo, DADO QUE el sistema opera normalmente QUIERO ver tanto la ficha técnica como la imagen en alta definición PARA mostrarlo al cliente y acelerar el proceso de compra. ESTO DEBE suceder en menos de 2 segundos.

### ASR3
COMO vendedor de concesionario CUANDO ingreso la identificación del prospecto, DADO QUE el sistema opera normalmente QUIERO que el sistema me traiga toda su información PARA consultar el perfil y los intereses mostrados por el cliente anteriormente. ESTO DEBE suceder en menos de 1 segundos por campo. 

### ASR2
COMO vendedor de concesionario CUANDO ingreso cada dato del prospecto, DADO QUE el sistema opera normalmente QUIERO crear lo más rápidamente posible la información del prospecto para dedicarme a la venta PARA poder presentar al cliente las bondades del vehículo y enviarlo luego a su correo. ESTO DEBE suceder en menos de 1 segundo por campo

### ASR13
COMO Gerente de General de CCV CUANDO consulto uno de los KPIs de comportamiento de concesionarios y talleres para una marca específica, DADO QUE el sistema opera normalmente QUIERO ver la gráfica del KPI así como la información relacionada para poder definir tácticas por marca ESTO DEBE ser informado en menos de 1 minuto 

### ASR25
COMO Subgerente de postventa CUANDO consulto el listado de existencias de repuestos y partes QUIERO obtener la información con alertas de stock mínimo PARA poder hacer el restock sin afectar ningún concesionario ESTO DEBE suceder en menos de 5 segundos

### ASR26
COMO Subgerente de postventa CUANDO consulto el listado de existencias de repuestos y partes QUIERO obtener la información con alertas de stock mínimo PARA para poder hacer el restock sin afectar ningún concesionario ESTO DEBE presentarse con información dada por los talleres y que este actualizada hasta la última hora de la consulta 

### ASR38
COMO Subgerente de Ventas de CCV CUANDO pido el reporte de tendencia por concesionario, DADO QUE el sistema opera normalmente QUIERO ver la gráfica de tendencia actualizada hasta el momento de la consulta PARA aplicar promociones y campañas de mercadeo. ESTO DEBE presentarse con información dada por los concesionarios actualizada hasta la última hora de la consulta 

## Arquitectura propuesta

### General
A continuación se expone el resultado de aplicar los estilos y tácticas arquitecturales seleccionados para favorecer la Latencia sobre la arquitectura base que el equipo desarrollo en el Sprint 0. Estas tiene por principal objetivo favorecer el rápido procesamiento de las archivos de postventa y las solicitudes de cotización de los diferentes vehículos que se exhiben en los concesionarios.

### Estilos de Arquitectura

#### LMAX
Usaremos el estilo de arquitectura LMAX para las transacciones enviadas por los jefes de taller y los vendedores de concesionarios, los cuales esperan que el sistema responda rápidamente debido a que están caminando con el cliente entre los vehículos o el taller para poder concretar una venta o servicio postventa.

#### LAMBDA
Usaremos el estilo de arquitectura LAMBDA para procesar los archivos en Batch de las ventas y las reparaciones en talleres por los volúmenes de información que llegara y la información en tiempo real que envíen los vendedores y jefes de taller como transacciones individuales en tiempo real. Con este estilo de arquitectura: Se procesará la información en Batch y se complementara con la información en tiempo real de modo que los principales Stakeholder tengan los KPI requeridos para la toma de decisiones con una visión fresca de los movimiento del mercado

### Tácticas de Arquitectura 

#### Map Reduce
Usaremos map reduce en el estilo Lambda para crear la capa batch ya que recibimos archivos de dos fuentes: Ventas y Postventas luego deben ser procesados y unidos basado en el cliente para tener una visión 360 del cliente: Lo que le interesa comprar, lo que realmente compro y el servicio que le da a su vehículo. También usaremos la táctica de map reduce en la capa de servicio de lambda para unir la información que viene de la capa Batch, con la información que viene de la capa Speed. 

#### Cache de Información
Colocaremos la información de productos (vehículos, partes) en cache de forma que los procesos como Business Logic en LMAX y la capa de procesamiento de LAMBDA que consultan productos se vean beneficiados por la baja latencia ya que la información probablemente se encuentra en la memoria cache

### Vistas

#### Vista Funcional
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/VistaFuncionalLMAX-LAMBDA-MAPRED.png]]

#### Vista de Información
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/VistaInformacion.png]]

#### Vista de Despliegue
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/Diagrama%20de%20despliegue.png]]

## Puntos de Sensibilidad

### Conocimiento de lo que sucede en los talleres y concesionarios: 

Se tomó una decisión critica de diseño que consiste en usar el estilo de arquitectura LAMBDA para procesamiento de información pero usando la capa "Speed" con arquitectura LMAX para que a la vez que se sirve a los vendedores y jefes de taller en cotizaciones, información de producto, complementa la información que requieren los directivos de CCV para el apoyo a la toma de desiciones. Ya que las directivas de CCV tienen problemas para saber que pasa realmente en concesionarios y talleres, pues en la actualidad esa información les llega con semanas de atraso lo que no les permite tomar decisiones con dicha información. CCV requiere procesar esa información y tenerla disponible para su uso en la toma de decisiones de manera oportuna.

## Experimento
Video - puntos de sensibilidad : https://youtu.be/U0onXpev9ak
Video - experimento : https://youtu.be/cBwpvaNpJYc

### Problema a Resolver

Para el diseño presentado por el equipo se decidió realizar el experimento de latencia sobre los componentes de venta y postventa los cuales están relacionados con los siguientes ASR:

ASR1: Visualización de información de vehículos por parte de los vendedores en vitrina. Este ASR debe ejecutarse en un tiempo no mayor a 2 segundos.
ASR2: Creación de prospecto de compra de vehículo en el sistema, esto debe suceder en un tiempo no mayor a 1 seg

Estos dos ASR son de gran importancia ya que parte del éxito de ventas y flujo de ingresos de CCV depende del trabajo que los vendedores realizan en concesionarios con posibles cliente, todo esto apoyado en la plataforma tecnológica de la empresa. Esta información debe darse de forma rápida para poder atender la mayor cantidad de clientes posible y generar las mayores oportunidades de venta.

### Diseño del Experimento
La parte crítica del diseño de este componente se encuentra en la recepción de las peticiones de los clientes web y móviles, estas deben ser procesadas rápidamente ya que en días de bastante ocupación es de gran criticidad una atención oportuna a los clientes.
Se reciben peticiones REST que son atendidas por un componente de estilo arquitectural LMAX. Este permite procesar una gran cantidad de peticiones debido a la cola circular que utiliza lo cual permite que el sistema pueda atender múltiples peticiones en un solo thread de ejecución. 

Videos experimento: https://youtu.be/cBwpvaNpJYc
                    https://youtu.be/U0onXpev9ak

[![Estilo de Arquitectura LMAX](https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/Lmax-architecture.PNG)](https://drive.google.com/open?id=0BzuXVblyDImnSGZDQ3Rwb3N0WEk "Estilo de Arquitectura LMAX")

### Implementación del Experimento
La prueba fue diseñada para simular un usuario haciendo peticiones POST cada segundo durante un minuto.
Con estas peticiones se pretende experimentar la latencia en la arquitectura seleccionada en este caso LMAX, cada petición está compuesta por un objeto JSON que es recibido por el sistema e insertado en la cola del input disruptor para ser procesado y generar una respuesta.

## Resultados
Como herramienta de medición de resultados utilizamos JMETER, la cual nos permitió obtener los valores de tiempo para las peticiones realizadas por el usuario. 
De acuerdo a la grafica de resultados, se obtuvo un tiempo minimo de 30 ms y un tiempo maximo de 142 ms, esto garantiza que la arquitectura cumplira con las expectativas de los stakeholder en cuanto a Latencia.

Gráfica de tiempos
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/jmeter1.png]]

Estadísticas de tiempos
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/jmeter4.png]]

### Análisis de Resultados y Respuesta al problema

# Retrospectiva trabajo de equipo

## Aspectos que han funcionado
Para el trabajo en el Sprint 1, se realizo una coordinación de trabajo efectiva entre los miembros del equipo, todos los miembros estuvieron involucrados tanto en diseño como en desarrollo de la arquitectura y experimento. La buena comunicación del equipo permitió conseguir unos resultados satisfactorios al final del Sprint 1.

## Aspectos que no han funcionado correctamente

La elección de herramientas para el desarrollo del experimento dificulto la tarea para los miembros del equipo ya que habían ciertos aspectos que el equipo no dominaba en su totalidad, por lo cual fue necesario realizar esfuerzos extra para poder nivelar el conocimiento del equipo y lograr el desarrollo del experimento.

Por lo pronto consideramos que el equipo ya posee el conocimiento suficiente sobre las herramientas escogidas para poder superar las siguientes experimentos.

 