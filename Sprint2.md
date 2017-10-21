# Sprint 2 - Escalabilidad

En el Sprint 2 el atributo de calidad que vamos a iniciar es el de Escalabilidad dada la prioridad de los Stakeholders del proyecto el cual quedó en segundo lugar después de la Latencia.

## ASR

ASR 11: COMO Gerente de mercadeo CUANDO para las fechas especiales (Dias de la Madre, Fines de semana de feria), DADO QUE el sistema opera con sobrecarga QUIERO poder registrar y dar respuesta a todas las cotizaciones hechas por los vendedores de concesionario ESTO DEBE suceder en menos de 2 segundos, dado que hay 1000 vendedores adicionales con clientes simultáneamente haciendo cotizaciones. 


ASR 4: COMO vendedor de concesionario CUANDO ingreso un modelo de vehículo, DADO QUE el sistema opera con sobrecarga QUIERO ver tanto la ficha técnica como la imagen en alta definición PARA mostrarlo al cliente y acelerar el proceso de compra. ESTO DEBE suceder en menos de 2 segundos, dado que hay 500 vendedores con clientes simultáneamente haciendo cotizaciones. 

ASR 8: COMO vendedor de concesionario CUANDO creo una Cotizacion DADO QUE el sistema opera normalmente QUIERO cargar la información del mismo y obtener el total de la cotización PARA que el cliente vea el valor, descuentos, y color en existencia ESTO DEBE suceder en 1 segundo, dado que hay 500 vendedores con clientes simultáneamente haciendo cotizaciones. 

## Arquitectura

## Tácticas de Arquitectura

### Scaling Out 
Permitir auto-escalar instancias del Business Logic para procesar mas cotizaciones por unidad de tiempo 

### Replicación de componentes de negocio
Replicar el componente de procesamiento de negocio y el catalogo de productos por región para poder cumplir con los picos de demanda

### Scaling Up
Para aumentar la capacidad del HW que alberga los Disruptores de entrada y de salida de 50 a 1500 para los días de la madre y ferias de vehículos

## Vistas

### Vista Funcional

Vista Funcional
[[https://github.com/MISO-4206/Grupo-6/blob/master/Documents/Images/Captura%20de%20pantalla%202017-10-21%20a%20la(s)%209.17.14%20a.m..png]]

## Puntos de Sensibilidad

La decisión critica de diseño esta en combinar el estilo de arquitectura LMAX para garantizar el tiempo de respuesta con el autoescalamiento horizontal de HW y componentes de negocio para poder reaccionar a los eventos especiales ya que pueden ocurrir en cualquier momento
