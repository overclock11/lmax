En el Sprint 1 el atributo de calidad que vamos a iniciar es el de Latencia dada la prioridad de los Stakeholders del proyecto.

# ASR Cubiertos en el Sprint 1: Latencia
Estos ASR fueron clasificados y prioridades entre arquitectura y el grupo de Stakeholders según su nivel de riesgo. Ellos son:

## ASR1
COMO vendedor de concesionario CUANDO ingreso un modelo de vehículo, DADO QUE el sistema opera normalmente QUIERO ver tanto la ficha técnica como la imagen en alta definición PARA mostrarlo al cliente y acelerar el proceso de compra. ESTO DEBE suceder en menos de 10 segundos.

## ASR3
COMO vendedor de concesionario CUANDO ingreso la identificación del prospecto, DADO QUE el sistema opera normalmente QUIERO que el sistema me traiga toda su información PARA consultar el perfil y los intereses mostrados por el cliente anteriormente. ESTO DEBE suceder en menos de 2 segundos por campo. 

## ASR2
COMO vendedor de concesionario CUANDO ingreso cada dato del prospecto, DADO QUE el sistema opera normalmente QUIERO crear lo más rápidamente posible la información del prospecto para dedicarme a la venta PARA poder presentar al cliente las bondades del vehículo y enviarlo luego a su correo. ESTO DEBE suceder en menos de 1 segundo por campo

## ASR13
COMO Gerente de General de CCV CUANDO consulto uno de los KPIs de comportamiento de concesionarios y talleres para una marca específica, DADO QUE el sistema opera normalmente QUIERO ver la gráfica del KPI así como la información relacionada para poder definir tácticas por marca ESTO DEBE ser informado en menos de 1 minuto 

## ASR25
COMO Subgerente de postventa CUANDO consulto el listado de existencias de repuestos y partes QUIERO obtener la información con alertas de stock mínimo PARA poder hacer el restock sin afectar ningún concesionario ESTO DEBE suceder en menos de 5 segundos

## ASR26
COMO Subgerente de postventa CUANDO consulto el listado de existencias de repuestos y partes QUIERO obtener la información con alertas de stock mínimo PARA para poder hacer el restock sin afectar ningún concesionario ESTO DEBE presentarse con información dada por los talleres y que este actualizada hasta la última hora de la consulta 

## ASR38
COMO Subgerente de Ventas de CCV CUANDO pido el reporte de tendencia por concesionario, DADO QUE el sistema opera normalmente QUIERO ver la gráfica de tendencia actualizada hasta el momento de la consulta PARA aplicar promociones y campañas de mercadeo. ESTO DEBE presentarse con información dada por los concesionarios actualizada hasta la última hora de la consulta 

# Puntos de Sensibilidad
## Conocimiento de lo que sucede en los talleres y concesionarios: 
Las directivas de CCV tienen problemas para saber que pasa realmente en concesionarios y talleres, pues en la actualidad esa información les llega con semanas de atraso lo que no les permite tomar decisiones con dicha información. CCV requiere procesar esa información y tenerla disponible para su uso en la toma de decisiones de manera oportuna

## Conocimiento del prospecto y posible cliente: 
En la actualidad CCV no tiene forma de conocer que pasa con un prospecto dentro del concesionario, que vehículo es el preferido bien sea de CCV o de la competencia lo cual los limita para sacar campañas de mercadeo mas focalizadas, para esto CCV requiere proveer de herramientas a los vendedores de los concesionarios para facilitarles la venta a la vez que CCV conoce el comportamiento del mercado

# Video del experimento
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Videos/CCV-Sprint1.mp4]]
[![Estilo de Arquitectura LMAX](https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/Lmax-architecture.PNG)](https://drive.google.com/open?id=0BzuXVblyDImnSGZDQ3Rwb3N0WEk "Estilo de Arquitectura LMAX")

# Resumen experimento
Para el diseño presentado por el equipo se decidió realizar el experimento de latencia sobre los componentes de venta y postventa los cuales están relacionados con los siguientes ASR:

ASR1: Visualización de información de vehículos por parte de los vendedores en vitrina. Este ASR debe ejecutarse en un tiempo no mayor a 2 segundos.
ASR2: Creación de prospecto de compra de vehículo en el sistema, esto debe suceder en un tiempo no mayor a 1 seg

Estos dos ASR son de gran importancia ya que parte del éxito de ventas y flujo de ingresos de CCV depende del trabajo que los vendedores realizan en concesionarios con posibles cliente, todo esto apoyado en la plataforma tecnológica de la empresa. Esta información debe darse de forma rápida para poder atender la mayor cantidad de clientes posible y generar las mayores oportunidades de venta.

La parte crítica del diseño de este componente se encuentra en la recepción de las peticiones de los clientes web y móviles, estas deben ser procesadas rápidamente ya que en días de bastante ocupación es de gran criticidad una atención oportuna a los clientes.

Se reciben peticiones REST que son atendidas por un componente de estilo arquitectural LMAX. Este permite procesar una gran cantidad de peticiones debido a la cola circular que utiliza, lo cual hace que el sistema pueda atender múltiples peticiones en un solo thread de ejecución. 

## Procedimiento
La prueba fue diseñada para simular un usuario haciendo peticiones POST cada segundo durante un minuto.
Con estas peticiones se pretende experimentar la latencia en la arquitectura seleccionada en este caso LMAX, cada petición está compuesta por un objeto JSON que es recibido por el sistema e insertado en la cola del input disruptor para ser procesado y generar una respuesta.

## Resultados
Como herramienta de medición de resultados utilizamos JMETER, la cual nos permitió obtener los valores de tiempo para las peticiones realizadas por el usuario. 
De acuerdo a la grafica de resultados, se obtuvo un tiempo minimo de 30 ms y un tiempo maximo de 142 ms, esto garantiza que la arquitectura cumplira con las expectativas de los stakeholder en cuanto a Latencia.

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/jmeter1.png]]
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/jmeter4.png]]


# Vistas
## Component & Connector
### Vista Funcional
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/VistaFuncionalLMAX-LAMBDA-MAPRED.png]]

### Vista Información
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/VistaInformacion.png]]

# Estilo de Arquitectura 
## LMAX
Usaremos el estilo de arquitectura LMAX para las transacciones enviadas por los jefes de taller y los vendedores de concesionarios, que esperar una baja latencia pues están caminando con el cliente entre los vehículos o el taller

## LAMBDA
Usaremos el estilo de arquitectura LAMBDA para procesar los archivos en Batch de las ventas y las reparaciones en talleres por los volúmenes de información que llegara y la información en tiempo real que envíen los vendedores y jefes de taller como transacciones individuales en tiempo real. Con este estilo de arquitectura: Se procesará la información en Batch y se complementara con la información en tiempo real de modo que los principales Stakeholder tengan los KPI requeridos para la toma de decisiones con una visión fresca de los movimiento del mercado

# Tácticas de Arquitectura 
## Map Reduce
Usaremos map reduce en el estilo Lambda para crear la capa batch ya que recibimos archivos de dos fuentes: Ventas y Postventas luego deben ser procesados y unidos basado en el cliente para tener una visión 360 del cliente: Lo que le interesa comprar, lo que realmente compro y el servicio que le da a su vehículo. También usaremos la táctica de map reduce en la capa de servicio de lambda para unir la información que viene de la capa Batch, con la información que viene de la capa Speed. 

## Cache de Información
Colocaremos la información de productos (vehículos, partes) en cache de forma que los procesos como Business Logic en LMAX y la capa de procesamiento de LAMBDA que consultan productos se vean beneficiados por la baja latencia ya que la información probablemente se encuentra en la memoria cache

# Retrospectiva trabajo de equipo

## Aspectos que han funcionado
Para el trabajo en el Sprint 1, se realizo una coordinación de trabajo efectiva entre los miembros del equipo, todos los miembros estuvieron involucrados tanto en diseño como en desarrollo de la arquitectura y experimento. La buena comunicación del equipo permitió conseguir unos resultados satisfactorios al final del Sprint 1.

## Aspectos que no han funcionado correctamente

La elección de herramientas para el desarrollo del experimento dificulto la tarea para los miembros del equipo ya que habían ciertos aspectos que el equipo no dominaba en su totalidad, por lo cual fue necesario realizar esfuerzos extra para poder nivelar el conocimiento del equipo y lograr el desarrollo del experimento.

Por lo pronto consideramos que el equipo ya posee el conocimiento suficiente sobre las herramientas escogidas para poder superar las siguientes experimentos.

 