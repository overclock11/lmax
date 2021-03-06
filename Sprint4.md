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

El estilo de arquitectura usado en este Sprint es una adaptación PACE - C2 sobre la arquitectura ya montada en los sprints anteriores con LAMDA y LMAX. Sin embargo hicimos algunas adaptaciones al modelo PACE ya que no necesitamos algunas de las capas como la capa de datos del modelo PACE con el fin de favorecer la latencia 

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

Este audit trail será configurable para poder subir y bajar el nivel de auditoria pero teniendo un default mínimo de configuración marcado por la táctica: defaults de Seguridad

### Vistas

#### Vista Funcional

Desde un punto de vista funcional observamos que los puntos críticos en la disponibilidad están asociados principalmente a la capa de seguridad (PACE) y a los recursos sensibles como el canal de transmisión y sus extremos, la información de cliente bien sea desde la aplicación y desde la base de datos

En la capa funcional se ve como los componentes manejadores de protocolos, de comunicaciones y de firmas se encargan de asegurar que la persona que entra o envía la información es quien dice ser. Adicionalmente el componente de Autorización en conjunto con el sub-sistema de seguridad se encargan de asignar los privilegios que dicho usuario puede tener sobre la aplicación de esta forma controlamos quien se conecta y que puede hacer con los datos y la aplicación 

Por otro lado también se ven los componentes de auditoria y control que registran eventos sensible sobre el sistema o la base de datos

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

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/SecuritySetOfTests.png]]

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/dise%C3%B1o%20experimento%20seguridad.png]]


### Análisis de vulnerabilidades

[Video Análisis de vulnerabilidades](https://youtu.be/VF_dkj6TwEk)

Realizamos un análisis del aplicativo previo a la definición del experimento para detectar vulnerabilidades que podrían dar indicios de cambios a realizar en la arquitectura o plantearan una re-definición del experimento.
Para ello utilizamos Nessus, en la cual definimos un escaneo avanzado apuntando a la ip local junto con los puertos que corresponden a las instancias de los servicios de CCV.

Una vez finalizado el escaneo, este fue el reporte que se obtuvo.

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/scanNesuss.png]]

Como se observa en el reporte, no se obtuvieron vulnerabilidades mayores a nivel Medium que nos hicieran replantear puntos importantes de la arquitectura o del experimento, por lo cual decidimos proseguir con el planteamiento inicial.

### Prueba del Experimento  
#### Prueba 1- Cross Site Request Forgery

[Video Cross Site Request Forgery y Solución](https://youtu.be/2kqvTtcQAns)

Los servicios de CCV están expuestos mediante el protocolo https  lo cual asegura el canal mediante el cual se transfiere la información de CCV; sin embargo en las premisas del usuario, específicamente  en el browser existe la vulnerabilidad a ataques de cross site scripting, hoy en día una de los ataques más usados en esta modalidad es el Cross Site Request Forgery,  este ataque aprovecha que el browser ya tiene los tokens de autenticación y emite request al servidor de CCV a nombre del usuario autenticado, con lo cual puede obtener información valiosa para CCV para enviarla a al atacante o en el peor de los casos ejecutar acciones en la aplicación de CCV sin que el usuario se entere.

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/CrossSiteRequestForgery.PNG]]



#### Prueba 2- Man-in-the-middle

[Video Cross Man-in-the-middle attack (MITM) y Solución](https://youtu.be/xzoWmqtuNyQ)

Este tipo de ataque aprovecha el acceso que pueda tener el atacante a cualquier elemento de red que se encuentre entre el sistema CCV y el dispositivo del cliente, en un canal de comunicación que no esté cifrado, es fácil para el atacante instalar un sniffer y comenzar robar información valiosa de CCV.

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/MITM1.PNG]]
Para prevenir este ataque nuestro cliente web  recibe los mensajes cifrados y así  el atacante puede obtener la información  pero dado que está cifrada no le será útil.
Además el contar con un canal de comunicación cifrado  al contar un certificado de seguridad en el servidor es mucho más difícil para el atacante poder asimilar la información que pueda interceptar.

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/MITM2.PNG]]

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

Dado que no queremos romper con el atributo de escalabilidad en el sistema CCV, decidimos montar en el experimento un sistema de autenticación por medio de un token de sesión mantenido en el cliente y así mantener los componentes del módulo de ventas sin estado.
 El token de autenticación va a ser validado por el back-end de CCV toda vez que un cliente web quiera acceder algún servicio REST, si el token es valido y no ha expirado, entonces la consulta al servicio REST se ejecutará normalmente.

```javascript
app.use(function(req, res, next) {
    authenticationProvider.middlewareVerifyToken(req,res,next);
});

//--------------------------------------------
module.exports.middlewareVerifyToken = function (req,res,next) {
    // check header or url parameters or post parameters for token
    let token = req.body.token || req.query.token || req.headers['x-access-token'];// || req.cookies['session-cookie'];
    // decode token
    if (token) {
        // verifies secret and checks exp
        jwt.verify(token, 'thisisthesupersecretkey', function (err, decoded) {
            if (err) {
                return res.json({success: false, message: 'Failed to authenticate token.'});
            } else {
                // if everything is good, save to request for use in other routes
                req.decoded = decoded;
                next();
            }
        });

    } else {
        // if there is no token
        // return an error
        return res.status(403).send({
            success: false,
            message: 'No token provided.'
        });
    }
};

```
Para obtener el token existe un servicio REST inicial al cual se le proporciona la pareja de credenciales userName y password, estos datos se validan y se devuelve como respuesta el token de autenticación

```javascript

module.exports.authenticateUser = function (request, callback) {

    let registeredUserName = "ccvadmin";
    let registeredPassword = "admin123";
    console.log('data', request);
    // find the user
    let userName = request.userName;
    let userPassword = request.password;


    if (userName !== registeredUserName) {
        callback({success: false, message: 'Authentication failed. User not found.'});
    } else if (userName === registeredUserName) {

        // check if password matches
        if (userPassword !== registeredPassword) {
            callback({success: false, message: 'Authentication failed. Wrong password.'});
        } else {

            // if user is found and password is right
            // create a token with only our given payload
            // we don't want to pass in the entire user since that has the password
            const payload = {
                admin: true
            };
            let token = jwt.sign(payload, 'thisisthesupersecretkey', {
                expiresIn: 86400 // expires in 24 hours
            });

            // return the information including token as JSON
            callback({
                success: true,
                message: 'Enjoy your token!',
                token: token
            });
        }
    }
};

```

Hicimos la validación del tiempo que se incrementa de latencia adicionando esta verificación en cada llamado y encontramos que no se superan los 100ms esto ocurre porque el token que se emite en el primer servicio para el caso del experimento en el proceso de verificación del token no se tiene ningún acceso a capas de persistencia.

## Retrospectiva

[Video Arquitectura y Puntos de Sensibilidad](https://youtu.be/nBeuSxaUNG4)


## Aspectos Positivos del Sprint y del Equipo

En este sprint desde el punto de vista del equipo con la dinámica que trajimos del sprint 3 logramos cumplir los objetivos en el tiempo adecuado  manteniendo un buen ritmo de trabajo.
Los grupos funcional y técnico para iniciamos las actividades, sincronizandonos a través de los Daily, la dinámica fue la misma, cuando el equipo funcional terminó sus labores, pasó a apoyar el equipo técnico para culminar el experimento.

## Aspectos a Mejorar del Sprint y del Equipo

Como equipo nos dimos cuenta que tenemos un gran desconocimiento frente a los ataques que pueden realizarse a una aplicación, lo que nos dificulto hacer el experimento, debemos aprender un poco más sobre los tipos de ataque y como se ejecutan para poder pensar en cómo proteger nuestras aplicaciones

 

 