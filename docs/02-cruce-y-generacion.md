# Cómo se cruzan los 11 orígenes

El orden de trabajo está en [00-plan-de-trabajo.md](00-plan-de-trabajo.md).

Antioquia. La ciudad es el municipio, con `cod_divipola` de 5 dígitos.

Los 11 orígenes se cruzan por dos espinas:

- **Geografía:** un municipio, un código.
- **Tiempo:** fecha, hora, mes y periodo. Esta espina se construye. No es uno de los 11 orígenes y no se descarga.

El inventario de cada origen (qué se genera y qué se descarga) está en [01-origenes-y-eda.md](01-origenes-y-eda.md).

## La regla

Nada se une por el nombre del municipio. Itagüí, El Carmen de Viboral o San Andrés de Cuerquia se escriben distinto en EPM, SUI, DANE y el mapa. La llave es el código de 5 dígitos.

Dos datos solo se cruzan en el detalle que comparten.

- Subir un dato fino a un total más grueso: sí. Ejemplo: sumar las horas de un cliente para obtener el mes.
- Pegar un dato grueso como contexto del municipio, sabiendo que es del municipio y no del cliente: sí. Ejemplo: la población del DANE al lado de una ciudad.
- Repartir un total municipal entre clientes como si cada uno lo hubiera medido: no.

## Cómo queda el mapa

```text
DANE / DIVIPOLA (códigos)  +  OpenStreetMap (polígonos)
                    |
                    v
              GEOGRAFÍA                         TIEMPO
         cod_divipola, municipio          fecha, hora, mes
                    |                            |
     +--------------+--------------+             |
     |              |              |             |
  Cliente        Activo         Clima -----------+
     |              |              |
  Consumo      Incidente       (JSON de la cabecera,
  Factura      Mantenimiento    con el código ya pegado)

  SUI, población DANE y EPM abierto
  se cuelgan solo de Geografía, en su propio grano.
  No tocan al cliente ni al activo.
```

Lectura de las flechas:

- El cliente se genera ya con `cod_divipola`.
- El consumo no trae ciudad. Llega a ella por `cliente_id`.
- La factura es la suma del consumo de ese cliente en ese mes, más el valor, la tarifa y si pagó.
- El activo se genera como un punto dentro del polígono. Ahí queda su municipio.
- La falla y el mantenimiento se generan sobre un activo. En este dibujo la falla vive donde vive el activo, así que hereda su ciudad.
- El clima llega de Open-Meteo por la coordenada de la cabecera, en JSON y por hora. Al guardarlo se le escribe el `cod_divipola`.
- SUI, la población del DANE y EPM abierto se guardan en su propio grano y solo tocan el municipio.

## Ficha de cruce

| De | Hacia | Se igualan | Antes de unir |
|---|---|---|---|
| Cliente | Geografía | `cod_divipola` | Nada. El código ya viene en el cliente |
| Consumo | Cliente | `cliente_id` | Nada |
| Consumo | Tiempo | `fecha` + `hora` | Nada |
| Consumo de una ciudad | Clima | `cod_divipola` + `fecha` + `hora` | Sumar los kWh del municipio y la hora. El clima no se une al medidor |
| Facturación | Cliente | `cliente_id` | Nada |
| Facturación | Consumo | `cliente_id` + `periodo` | Sumar las horas del mes. Son dos kWh distintos |
| Activo | Geografía | El punto cae dentro del polígono | Una vez, al cargar. Queda guardado el `cod_divipola` |
| Incidente | Activo | `infraestructura_id` | Nada. La ciudad se hereda del activo |
| Mantenimiento | Activo | `infraestructura_id` | Nada |
| Incidentes de una ciudad | Clima | `cod_divipola` + `fecha` + `hora` | Contar las fallas en una fila. La lluvia se queda en su fila |
| Clima | Geografía | `cod_divipola` asignado | El JSON trae latitud y longitud de la cabecera, no el código |
| SUI | Geografía | `cod_divipola` + `periodo` | Solo para comparar el total del municipio. No toca al cliente |
| DANE población | Geografía | `cod_divipola` + año | Contexto del municipio. No es un cliente |
| EPM abierto | Geografía | `cod_divipola` + `periodo` | Contexto. Si la fila es un circuito sin código, no se cruza |

## Ejemplo: lluvia y fallas de la misma hora

Números inventados, para ver las columnas. No vienen de un archivo.

1. Open-Meteo dice: cabecera del municipio `05001`, 15 de enero de 2026, hora 18, lluvia 8,5 mm.
2. Al guardar el clima se le escribe `cod_divipola = 05001`.
3. El simulador escribe 3 fallas sobre activos de ese mismo municipio, a esa fecha y esa hora.
4. Para compararlas no se pega la lluvia encima de cada falla. Se arman dos filas:
   - Clima: `05001` + `2026-01-15` + hora 18 → 8,5 mm.
   - Fallas: `05001` + `2026-01-15` + hora 18 → 3 incidentes.
5. Esas dos filas se igualan por código, fecha y hora.

Si se sumara la lluvia una vez por cada falla, un día con 40 fallas contaría la misma lluvia 40 veces.

La lluvia no se une al medidor. El consumo se une al cliente, y el cliente a la ciudad. Para ver consumo y clima juntos, primero se suman los kWh de todos los clientes de ese municipio en esa hora.

## Ejemplo: medidor y factura

Un cliente tiene 24 lecturas en un día y cerca de 720 en un mes. La factura es una sola fila de ese mes.

Primero se suman las horas de ese `cliente_id` en ese `periodo`. Esa suma es el kWh del mes. La factura usa ese total y le agrega valor, tarifa y estado de pago (pagada, pendiente o en mora).

Los dos campos se llaman `consumo_kwh` y no se suman entre sí: uno es la hora, el otro es el mes.

## En qué orden se crean los datos

Primero existe el lugar real. Después se cuelga lo público. Al final se inventa la empresa, usando esos códigos y esos totales.

| Paso | Qué se hace | De dónde sale |
|---|---|---|
| 1 | Lista de municipios de Antioquia, departamento `05` | DANE / DIVIPOLA. Una fila, un código de 5 dígitos |
| 2 | Polígono de cada municipio pegado a ese código | OpenStreetMap. Si el mapa solo trae el nombre, se traduce al código y no se vuelve a usar el nombre |
| 3 | Calendario de fechas y horas del periodo elegido | Se construye. No se descarga |
| 4 | Clima horario de la cabecera de cada municipio que entre al corte | Open-Meteo, JSON. Al guardar se le escribe el `cod_divipola` |
| 5 | Población, totales del sector, tarifas e interrupciones publicadas | DANE, SUI y EPM. Se guardan en su grano. Sirven de contexto y de calibración |
| 6 | Clientes, cada uno con un `cod_divipola` de la lista | Se genera. La cantidad por ciudad se orienta con los suscriptores de SUI, si el archivo los trae. No se copia la población del DANE como si fueran clientes |
| 7 | Activos con latitud y longitud dentro del polígono de su municipio | Se genera. El punto se confirma contra el polígono y queda guardado el código |
| 8 | Consumo de cada cliente, cada hora | Se genera, con estación y franja horaria. Lleva `cliente_id`, `fecha` y `hora` |
| 9 | Factura del mes | Se calcula sumando el consumo de ese cliente en ese mes. Encima se agregan valor, tarifa y si pagó |
| 10 | Fallas sobre activos | Se genera. Hay más fallas si el activo es viejo y si el clima de esa ciudad y esa hora trae lluvia o calor |
| 11 | Mantenimientos sobre activos | Se genera. Un trabajo, un activo, una fecha |

## Qué no se cruza

| Intento | Por qué se queda aparte |
|---|---|
| Demanda de XM con un transformador | XM mide el sistema eléctrico entero en esa hora. El transformador es un equipo de un municipio. No comparten código. XM no entra en este corte |
| kWh de la hora con kWh de la factura | Uno es el medidor en una hora. El otro es lo cobrado en el mes |
| DANE o SUI con un cliente | El censo y el SUI resumen el municipio. Ninguno trae `cliente_id` |
| Interrupciones publicadas de EPM con la tabla de incidentes | EPM abierto es contexto. La falla simulada es otra tabla |
| Lluvia sumada sobre cada incidente | La misma lluvia se contaría una vez por cada falla |

## Lo que el archivo real todavía puede cambiar

- Si SUI viene por año y no por mes, la calibración es municipio × año, no municipio × mes.
- Si EPM trae un circuito y no trae `cod_divipola`, esa fila se queda fuera hasta tener una tabla circuito → municipio.
- Cuántos municipios entran al consumo horario. El mapa de códigos puede ser Antioquia entera. El detalle de cliente × hora se acota cuando la muestra diga el volumen.
