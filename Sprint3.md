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
COMO Gerente de Ventas CUANDO un vendedor de concesionario en la feria del automóvil o día de la madre, DADO QUE el sistema opera con sobrecarga QUIERO que todas las cotizaciones sean respondidas por el sistema ya que en estas fechas la probabilidad de venta es tres veces mayor que en cualquier otra fecha del año PARA poder presentar al cliente rápidamente el precio y los descuentos del vehículo de su interés  ESTO DEBE ser respondido en menos de 2 segundos y el 99,999% de la veces lo cual quiere decir que en todo el fin de semana de feria solo se permite que 1 cotización no sea respondida o este sobre el tiempo aceptable de respuesta 

## Arquitectura Propuesta

### General
La propuesta de arquitectura de este sprint comprende la unificación de las decisiones tomadas en el sprint 1 para favorecer la Latencia con las que el equipo ha tomado para favorecer la Escalabilidad. Las tácticas a aplicar tienen por objetivo el trabajar en conjunto con el estilo arquitectural LMAX para que el sistema se comporte de forma adecuada en momentos de gran estrés.

### Estilo de Arquitectura

### Tácticas de Arquitectura

### Vistas

#### Vista Funcional

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/VistaFuncionalEscalamiento.png]]

#### Vista de Despliegue

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/DespliegueEscalabilidad.png]]

## Puntos de Sensibilidad

La decisión critica de diseño está lograr una disponibilidad del 99,999% durante el fin de semana de la feria del automóvil y día de la madre, para esto debemos lograr que en caso de falla de uno de los componentes clave de la arquitectura como:
El Business Logic
El Catalogo de Productos
El sub-sistema de clientes
Esten disponibles y se puedan recupera sin intervención humana todo el tiempo ya que por los requerimientos de los stakeholders solo podemos fallar en una cotización durante todo el evento