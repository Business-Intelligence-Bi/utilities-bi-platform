# Orígenes de Makaia y primer EDA

El orden de trabajo está en [00-plan-de-trabajo.md](00-plan-de-trabajo.md).

Territorio: **Antioquia** (DIVIPOLA, departamento `05`).
Ciudad en este proyecto = **municipio**, con código `cod_divipola` de 5 dígitos.
El tablero podrá filtrar por ese código.

Esta guía no cruza datos y no simula la empresa. Sirve para saber **qué origen se inventa**, **qué origen se descarga** y **qué hay que anotar** la primera vez que se abre cada archivo.

El cruce y el orden para generar los datos están en [02-cruce-y-generacion.md](02-cruce-y-generacion.md).

## Dos cajones

No se mezclan.

| Cajón | Qué es | Para qué sirve |
|---|---|---|
| A. Se genera | El simulador escribe la operación de una empresa ficticia: cliente, hora, activo, falla, factura y mantenimiento | El detalle fino que una eléctrica real no nos va a entregar |
| B. Se descarga | Fuentes públicas y reales. Son más gruesas | Mapa, clima, población y totales para calibrar |

Lo generado es la operación de la empresa ficticia. Lo descargado es real y **no trae** al cliente ni al transformador de esa empresa.

El simulador todavía no se escribe. Espera a que las muestras reales digan cuántas ciudades y qué periodo entran. Simular cada cliente, cada hora, en los 125 municipios es pesado. El mapa sigue siendo Antioquia.

XM queda fuera de este corte. Si más adelante entra, solo como contexto del sistema eléctrico por fecha, sin municipio y sin transformador.

## Cajón A — el simulador, más adelante

Seis orígenes. Todavía no se generan.

| Origen | Rol | Qué se le exige |
|---|---|---|
| Clientes | Saber quién es el usuario y en qué ciudad está | Una fila por cliente. Llaves: `cliente_id` y `cod_divipola`. No es el censo ni la lista de suscriptores de SUI |
| Consumo | Ver cómo evoluciona la energía medida | Una fila por cliente y hora. Medida: `consumo_kwh`. No es el kWh de la factura ni el total de SUI |
| Facturación | Ver el valor cobrado en el mes, la tarifa y si pagó | Una fila por cliente y mes. Trae valor, tarifa, estado de pago y un `consumo_kwh` del mes. No se une a la hora solo por `cliente_id` |
| Infraestructura | Saber qué activo hay y en qué ciudad cae | Una fila por activo (transformador, poste, línea, subestación, protección), con latitud y longitud. El `cod_divipola` sale del polígono, no del nombre del municipio. No cuenta las fallas |
| Incidentes | Ver cada falla: cuántos clientes afectó y cuántos minutos duró | Una fila por evento, colgada del activo con `infraestructura_id`. No es el archivo de interrupciones de EPM ni el SAIDI publicado |
| Mantenimiento | Ver cada trabajo sobre un activo y su costo | Una fila por intervención (preventivo, correctivo o predictivo). No es la lista de fallas |

## Cajón B — se descarga primero

Cinco orígenes. Una muestra, no el histórico completo. Cada descarga desbloquea la siguiente.

| Orden | Origen | Rol | Qué anotar en la muestra |
|---|---|---|---|
| 1 | DANE / DIVIPOLA | Padrón de municipios de Antioquia y su población | Código de 5 dígitos, departamento `05`, nombre solo como etiqueta, cuántos municipios quedaron |
| 2 | OpenStreetMap | El dibujo de cada municipio | Si el polígono trae el código o solo el nombre. Si comuna o barrio viene en todos los municipios o solo en algunos |
| 3 | Open-Meteo | Saber si en esa ciudad, a esa hora, llovió o hizo calor | 2 o 3 cabeceras, no las 125. Si la hora llega como número o como fecha-hora. Nombre real del campo de lluvia |
| 4 | SUI | Total publicado del sector, para calibrar | Si el periodo es mes o año. Si trae suscriptores y consumo, y con qué nombre de columna. Si el municipio trae código o solo nombre |
| 5 | EPM Datos Abiertos | Tarifas e interrupciones publicadas, como contexto | Si la fila es municipio o circuito. Si el circuito se puede llevar a un `cod_divipola` |

El nombre exacto del archivo de SUI y el de EPM no está cerrado. Se anota al abrir la muestra.

### Mismas preguntas para cada archivo

1. ¿Cuántas filas trae la muestra?
2. ¿Qué columnas vinieron?
3. ¿Cuál columna es el código de municipio?
4. ¿El periodo es mes o año?
5. ¿De qué años hay datos?
6. ¿El nombre del municipio viene aparte del código?

## Qué mirar en cada muestra para poder cruzar después

**DIVIPOLA.** ¿Cada municipio de Antioquia tiene un código de 5 dígitos? ¿Hay códigos repetidos? ¿El nombre viene solo como etiqueta? ¿Cuántos municipios quedaron?

**OpenStreetMap.** ¿El polígono trae `cod_divipola` o solo el nombre? ¿Falta algún municipio del padrón? ¿Comuna o barrio viene en todos, o solo en algunos? Si no viene parejo, la zona fina (comuna o barrio) se deja quieta. Para este alcance la llave es el municipio.

**Open-Meteo.** ¿La hora viene como número (`18`) o como fecha-hora (`2026-01-15 18:00`)? ¿Cómo se llama la columna de lluvia en el JSON? ¿El punto consultado es la cabecera de ese municipio? El clima de la cabecera es el clima del municipio, no el de cada vereda ni el de un transformador.

**DANE (población).** ¿La población está a municipio y año? ¿El código coincide con el padrón de DIVIPOLA? ¿Qué municipios faltan? Esos habitantes no son la lista de clientes.

**SUI.** ¿El periodo del archivo es mes o año? El documento de integración dice mes; el documento inicial decía año. Manda el archivo. ¿Trae suscriptores, consumo, o las dos cosas, y con qué nombre de columna? ¿El municipio viene con código o solo con nombre? ¿El mismo municipio y periodo está repetido? SAIDI y SAIFI, si vienen, son una referencia municipal publicada. No se calculan sobre los datos simulados.

**EPM Datos Abiertos.** ¿La fila es municipio o circuito? ¿De qué periodo? Si es circuito, ¿hay alguna columna que lo lleve a un `cod_divipola`? Si no la hay, el circuito no se cruza todavía. Esas interrupciones publicadas no reemplazan la tabla de incidentes simulados.

## Orden de la semana

1. Congelar este inventario: seis orígenes se generan, cinco se descargan.
2. Bajar una muestra del cajón B, en el orden de la tabla, y llenar las seis preguntas.
3. Con las columnas reales, cerrar la ficha de cruce del otro documento.
4. Recién ahí se diseña el simulador, calibrado contra suscriptores, consumo municipal e interrupciones que la muestra haya mostrado. Cada fila generada lleva `cod_divipola` para filtrar por ciudad.

Dos decisiones pueden esperar a ver los archivos. No frenan la descarga:

- Si la falla se ubica con su propio punto o hereda el municipio del activo. El dibujo de cruce, por ahora, la pone donde vive el activo.
- Cuántos municipios entran al consumo horario. El mapa sigue siendo Antioquia. El volumen fino se decide con la muestra.

El arquitecto del modelo se llama cuando la ficha de cruce esté llena con las columnas reales de esos archivos.
