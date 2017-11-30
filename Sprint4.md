# Tabla de contenidos
- [Introducción](#introducción)
- [Requerimientos ASR](#requerimientos-asr)
- [Arbol de Ataque](#arbol-de-ataque)
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

En el Sprint 4 el trabajo se enfocara en el atributo de calidad Seguridad, para esto se realizó la priorización de una serie de ASR que se consideraron de gran importancia tanto para los Stakeholder como para el diseño de la arquitectura. Se diseñará un experimento de arquitectura con el objetivo de validar la propuesta de diseño del equipo para satisfacer los ASR.

## Requerimientos ASR

### ASR 20
COMO Gerente de Postventa CUANDO un centro de servicio envía el archivo con información de los servicios y reparaciones hechos a sus clientes QUIERO que la información este segura y no caiga en manos no autorizadas en ningún punto el proceso PARA garantizar que la información de clientes y reparaciones solos sea usada para el beneficio de nuestros clientes y de CCV DADO que el sistema opera normalmente ESTO DEBE suceder el 100% de las veces ya que hemos detectado que personal interno o externo ha podido obtener acceso no autorizado a dicha información y contactado a los clientes para llevarlos a centros de servicio no autorizados disminuyendo los ingresos de CCV y de los concesionarios

### ASR 21
COMO Gerente de Postventa CUANDO la información de nuestros clientes y sus detalles de contacto y vehículo se encuentre en las bases de datos de CCV QUIERO que la información este segura y no caiga en manos no autorizadas en ningún punto el proceso PARA garantizar que la información de clientes y reparaciones solos sea usada para el beneficio de nuestros clientes y de CCV DADO que el sistema opera normalmente ESTO DEBE suceder el 100% de las veces ya que hemos detectado que personal interno ha podido obtener acceso no autorizado a archivos o bases de datos y contactado a los clientes para llevarlos a centros de servicio no autorizados disminuyendo los ingresos de CCV y de los concesionarios

## Arbol de Ataque
Objetivo: Obtener la información de contacto del cliente de CCV
1. Extrae los detalles de la base de datos directamente  
   1.1 Acceder la base de datos directamente
    
       1.1.1 El atacante tiene una cuenta de base de datos antigua

       1.1.2 El atacante tiene acceso a las cuentas del servidor y por ahí accede la base de datos

   1.2 Accede la base de datos vía el password de uno de los miembros del staff

       1.2.1 El atacante obtuvo el usuario y pwd de AWS y RDS de unos de los empleados por medio de mail o en algún archivo

2. Extrae los detalles del archivo trasmitido por los concesionarios y talleres

   2.1 Accede a través de un comprador del la fuente (Concesionario y Taller)

       2.1.1 El atacante toma el archivo de clientes de uno de los computadores

   2.2 Toma el archivo cuando es trasmitido a CCV

       2.2.1 El atacante toma el archivo del canal de transmisión
 
       2.2.2 El atacante toma el archivo cuando llega a CCV e inicia el proceso

       2.2.3 El atacante toma el archivo del file system del servidor de AWS

3. Extrae los detalles de cada una de las cotizaciones enviadas por los vendedores

   3.1 Atacando el canal

        3.1.1 El Atacante usa sniffers en el canal de transmision

        3.1.2 El Atacante intercepta el mensaje antes de que salga del Concesionario o taller

   3.2 Atacando el modulo de ventas

       3.2.1 El atacante tiene usuario / Password del sistema de ventas y postventas

       3.2.2 El atacante usa las credenciales de otro usuario por medio de suplantación
 
       3.2.3 El atacante obtuvo el usuario y pwd de AWS y RDS de unos de los empleados por medio de mail o en algún archivo

Del anterior árbol de ataque podemos empezar a inferir los recursos vulnerables de la aplicación como son:
1. La base de datos de clientes
2. El archivo trasmitido por el taller o concesionario 
3. Accedo directo a la aplicación

Y probablemente la principal brecha de seguridad esta el control que tiene CCV del manejo de credenciales, así como la falta de conocimiento o divulgación de las políticas de seguridad. 
Tampoco tiene un conjunto de reglas de seguridad por defecto.

Como antecedentes históricos de CCV en varias ocasiones empleados internos de CCV o de los concesionarios han tenido acceso no autorizado a la información de los clientes para montar talleres no autorizados con repuestos traídos de la China y contactan los clientes para prometerles un precio menor y servicio equivalente. 

## Arquitectura Propuesta

### General

La propuesta de arquitectura de este sprint comprende la incorporación de un estilo de arquitectura que soporte la seguridad tanto de las comunicaciones como la identidad del usuario que accede la información con los privilegios adecuados sin sacrificar mucho los otros atributos de calidad ya logrados. 

### Estilo de Arquitectura

El estilo de arquitectura usado en este Sprint es una adaptación PACE - C2 sobre la arquitectura ya montada en los sprints anteriores con LAMDA y LMAX

### Tácticas de Arquitectura

#### Usar defaults de Seguridad
CCV no usa defaults de seguridad luego hay mucho personal que tiene acceso tanto aplicativos y base de datos aun cuando no deberían tener dicha autorización 

#### Detectar intrusión
La intrusión la detectaremos por medio de credenciales de usuario complementado con llaves publicas y privadas y firma digital para asegurar que no solo el usuario provee sus credenciales sino que también el dispositivo desde donde se envía es un dispositivo de confianza

#### Encripción de Información 
Aun cuando los canales son asegurados usando HTTPS eso no garantiza 100% que la información no sea extraída en los extremos o aun ya al interior de la aplicación en CCV por lo tanto usaremos encripción de datos sensible lo cual hace que la información no tenga valor para los atacantes aun cuando la han conseguido

#### Audit Trail
Para llevar un registro detallado de lo que hacen los diferentes actores que acceden el sistema y poder recobrarse de posibles ataques haciendo análisis de información y patrones de ataques
Este audit trail sera manejado de forma asincrónica entre el componente de negocio y el componente de auditoria para manejar la información de accesos.

Este audit trail sera configurable para poder subir y bajar el nivel de auditoria pero teniendo un default mínimo de configuración marcado por la táctica: defaults de Seguridad

### Vistas

#### Vista Funcional

Desde un punto de vista funcional observamos que los puntos críticos en la disponibilidad están asociados principalmente a la capa de seguridad (PACE) y a los recursos sensibles como el canal de transmisión y sus extremos, la información de cliente bien sea desde la aplicación y desde la base de datos

En la capa funcional se ve como los componentes manejadores de protocolos, de comunicaciones y de firmas se encargan de asegurar que la persona que entra o envía la información es quien dice ser. Adicionalmente el componente de Autorización en conjunto con el sub-sistema de seguridad se encargan de asignar los privilegios que dicho usuario puede tener sobre la aplicación de esta forma controlamos quien se conecta y que puede hacer con los datos y la aplicación 

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/SeguridadVistaFuncional.png]]


#### Vista de Despliegue

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/Diagrama despliegue Sprint 4.png]]

## Puntos de Sensibilidad
La decisión critica de diseño está en lograr asegurar la información sensible del cliente en su transmisión y su almacenamiento de ataques tanto externos como internos tanto en los canales de transmisión como en la aplicación y en la base de datos, a la vez que requerimos mantener dentro de limites aceptables los atributos de calidad de latencia, escalabilidad y disponibilidad. Por tanto nuestra arquitectura debe manejar los casos de suplantación de identidad, accesos no autorizados a los recursos críticos y asegurar canales débiles en todo el proceso

## Experimento

## Problema a Resolver
Nuestro experimento consiste en asegurar la informacion del sistema tanto en el transporte como en el almacenamiento  para que no pueda ser interceptada por personas externas o internas que de forma no autorizada quieran acceder a ella.
Ya sea directamente en la base de datos o interceptandola mientras es enviada.

## Diseño del Experimento

### Propósito
### Resultados esperados
### Recursos requeridos
### Elementos de arquitectura involucrados
### Esfuerzo estimado
### Resultados
### Acciones a seguir

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/SecuritySetOfTests.PNG]]

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/dise%C3%B1o%20experimento%20seguridad.png]]



### Prueba del Experimento  
#### Prueba 1- Cross Site Request Forgery
Los servicios de CCV están expuestos mediante el protocolo https  lo cual asegura el canal mediante el cual se transfiere la información de CCV; sin embargo en las premisas del usuario, específicamente  en el browser existe la vulnerabilidad a ataques de cross site scripting, hoy en día una de los ataques más usados en esta modalidad es el Cross Site Request Forgery,  este ataque aprovecha que el browser ya tiene los tokens de autenticación y emite request al servidor de CCV a nombre del usuario autenticado, con lo cual puede obtener información valiosa para CCV para enviarla a al atacante o en el peor de los casos ejecutar acciones en la aplicación de CCV sin que el usuario se entere.

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/CrossSiteRequestForgery.PNG]]

Para prevenir este ataque nuestro cliente web  recibe los mensajes cifrados y así  el atacante puede obtener la información  pero dado que está cifrada no le será útil.



[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/dise%C3%B1o%20experimento%20seguridad.png]]

### Hardware / SW usado

Como componente de SW adicional a los ya usados con los sprints anteriores:
* New Relic
* JMeter
* NodeJs
* AWS
* RDS
* LoadBalancer


## Resultados



## Análisis de Resultados y Respuesta al problema


## Retrospectiva



## Aspectos Positivos del Sprint y del Equipo

En este sprint desde el punto de vista del equipo con la dinámica que trajimos del sprint 2 el equipo fluyó mayor sincronía, y asertividad. 

Dividimos el equipo en 2 grupos: Frente Funcional y Frente Técnico para iniciar actividades, pero teniendo nuestro Daily Sprint para sincronizar los equipos, cuando el equipo funcional terminó sus labores, paso a apoyar el equipo técnico para culminar el experimento

## Aspectos a Mejorar del Sprint y del Equipo

Creemos como equipo que aun nos falta mucho mas conocimiento técnico de NodeJS para hacer una manejo optimo de Threads, Memoria Compartida, y sincronización de procesos para llevar el estilo de arquitectura LMAX a su máxima expresión. 

 

 