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

### ASR 11
COMO Gerente de mercadeo CUANDO para las fechas especiales (Dias de la Madre, Fines de semana de feria), DADO QUE el sistema opera con sobrecarga QUIERO poder registrar y dar respuesta a todas las cotizaciones hechas por los vendedores de concesionario ESTO DEBE suceder en menos de 2 segundos, dado que hay 1000 vendedores adicionales para un total de 1500 con clientes simultáneamente haciendo cotizaciones. Se estima que 1/3 de los vendedores envían cotizaciones en el mismo momento.

### ASR 14 
COMO Gerente de mercadeo CUANDO diferentes concesionarios hacen campañas particularmente en fines de semana DADO QUE el sistema opera normalmente QUIERO poder registrar y dar respuesta a todas las cotizaciones hechas por los vendedores de concesionario ESTO DEBE suceder en menos de 1 segundos, dado que los requerimientos de cotizaciones sobre el sistema pueden aumentar entre un 10% y un 20% sin previo aviso dadas estas campañas sorpresa. En condiciones normales hay 500 vendedores accediendo el sistema, pero en ocasiones cuando se cruzan campañas en varios concesionarios los vendedores pueden aumentar entre 550 y 600 vendedores activos. Se estima que 1/3 de los vendedores envían cotizaciones en el mismo momento.

## Arquitectura propuesta

### General

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

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/Captura%20de%20pantalla%202017-10-21%20a%20la(s)%209.17.14%20a.m..png]]

#### Vista de Despliegue

[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/Despliegue%20escalabilidad.png]]

#### Vista de Concurrencia

## Puntos de Sensibilidad

La decisión critica de diseño esta en combinar el estilo de arquitectura LMAX para garantizar el tiempo de respuesta con el autoescalamiento horizontal de HW y componentes de negocio para poder reaccionar a los eventos especiales ya que genera un aumento considerable en la carga del sistema.

## Experimento

### Problema a Resolver
### Diseño del Experimento
### Implementación del Experimento
### Resultados
### Análisis de Resultados y Respuesta al problema

## Retrospectiva
## Conclusiones
