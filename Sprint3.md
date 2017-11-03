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

### ASR XX

### ASR XX 

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
