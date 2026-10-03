# Plan de trabajo — Makaia, Etapa 1

Cómo avanzar, en orden, hasta tener el tablero de Power BI.

Territorio: Antioquia. Ciudad = municipio con `cod_divipola`.
Los orígenes están en [01-origenes-y-eda.md](01-origenes-y-eda.md).
El cruce y el orden para generar datos están en [02-cruce-y-generacion.md](02-cruce-y-generacion.md).
La estrella provisional, para verla antes de bajar archivos, está en [03-modelo-estrella.md](03-modelo-estrella.md).

Se hace un paso. Se cierra con la prueba de “listo cuando”. Recién ahí se abre el siguiente. El simulador, la base y el tablero vienen después de mirar los archivos reales.
    
## Dónde vamos

| Paso | Qué es | Estado |
|---|---|---|
| 0 | Entender orígenes y cruce | Hecho. Están los dos documentos de arriba |
| 1 | Bajar una muestra de cada fuente del cajón B | Ahora |
| 2 | Llenar la ficha real de cada archivo | Después del paso 1 |
| 3 | Cerrar alcance: ciudades, periodo y las dudas del archivo | Después del paso 2 |
| 4 | Dejar por escrito la estrella con esas columnas reales | Después del paso 3 |
| 5 | Cargar geografía, clima y contexto en PostgreSQL | Después del paso 4 |
| 6 | Generar clientes, activos, consumo, facturas, fallas y mantenimientos | Después del paso 5 |
| 7 | Comprobar que los cruces no inflan números | Después del paso 6 |
| 8 | Conectar Power BI, medidas y relaciones | Después del paso 7 |
| 9 | Armar las páginas del tablero | Después del paso 8 |
| 10 | Revisar que el tablero cubre la Etapa 1 y dejar memoria | Al cerrar |

XM, anomalías, predicción y el asistente de preguntas quedan fuera de este plan.

## Reglas que no se saltan

1. No se cruza por el nombre del municipio. Se cruza por `cod_divipola`.
2. El kWh de la hora y el kWh de la factura no se suman entre sí. Primero se suman las horas del mes.
3. La lluvia no se suma encima de cada falla. Se cuenta las fallas en una fila y la lluvia se queda en la suya.
4. SUI, DANE y EPM no identifican a un cliente ni a un transformador.
5. No se reparte un total del municipio entre clientes como si fuera una medición.
6. No se genera el cajón A antes de ver las muestras del cajón B.

## Paso 1 — Bajar las muestras

Objetivo: ver cómo llega cada fuente pública, con un archivo chico, no con todo el histórico.

Orden, porque unas desbloquean a las otras:

1. DANE / DIVIPOLA. Lista de municipios de Antioquia, departamento `05`.
2. OpenStreetMap. Polígono de esos municipios.
3. Open-Meteo. Clima de la cabecera de 2 o 3 municipios, no de los 125.
4. SUI. Una muestra del reporte público.
5. EPM Datos Abiertos. Una muestra de tarifas y otra de interrupciones.

Cómo:

- Guarda cada muestra en una carpeta `muestras/`, con el nombre de la fuente y la fecha.
- Anota de dónde salió (portal o API) en la bitácora de abajo.
- No transformes el archivo todavía. La primera copia se deja como llegó.
- Si un portal pide elegir año o municipio, elige poco: un año y, si obliga, Medellín más otro municipio.

Listo cuando: hay cinco muestras guardadas y cada una abre sin error.

No hagas todavía: no escribas el simulador, no armes tablas SQL, no abras Power BI.

## Paso 2 — Ficha real de cada archivo

Objetivo: reemplazar lo que el documento supone por lo que el archivo trae.

Copia esta bitácora una vez por fuente y llénala con lo que ves. El nombre del archivo de SUI y el de EPM se escriben aquí, porque aún no están cerrados.

```text
Fuente:
Archivo o consulta:
Fecha de la muestra:
Filas de la muestra:
Años que aparecen:
El periodo es mes, año u hora:
Columnas, con el nombre tal como viene:
Columna del código de municipio:
El nombre del municipio viene en otra columna: sí / no
Qué grano es una fila (un municipio y un mes, un punto y una hora, etc.):
Sirve para:
No sirve para:
Duda que quedó:
```

Preguntas cortas, además de la bitácora:

| Fuente | Confirma esto |
|---|---|
| DIVIPOLA | Código de 5 dígitos, sin repetidos, solo municipios de Antioquia. El nombre es etiqueta |
| OpenStreetMap | El polígono trae el código, o solo el nombre. Comuna o barrio no viene en todos |
| Open-Meteo | La hora es un número o una fecha-hora. Cómo se llama la lluvia en el JSON. El punto es la cabecera |
| DANE población | Municipio y año. El código coincide con el padrón. Qué municipios faltan |
| SUI | Mes o año. Si trae suscriptores y consumo, y con qué nombre. Si el municipio trae código. Si la misma ciudad y periodo está repetida |
| EPM | Municipio o circuito. Si el circuito tiene `cod_divipola` |

Listo cuando: las cinco bitácoras están llenas y cada una dice la columna del código, el periodo y el grano.

No hagas todavía: no corrijas el cruce “a ojo” si el archivo contradice el documento. Manda el archivo.

## Paso 3 — Cerrar el alcance

Objetivo: decidir el primer corte con lo que las muestras mostraron. Se anota en un párrafo, no se deja hablado.

Decisiones ya tomadas:

- Territorio: Antioquia.
- Ciudad: municipio con `cod_divipola`.
- La falla, por ahora, vive donde vive el activo.

Decisiones que se cierran en este paso:

| Decisión | Cómo elegirla |
|---|---|
| Ventana de historia | El periodo en el que clima, SUI y EPM se solapan. Si uno solo tiene un año, el primer corte usa ese año |
| Ciudades del dato fino | El mapa de códigos puede ser Antioquia. Cliente por hora, activos y fallas entran solo en las ciudades que elijas ahora. Empieza por pocas, por ejemplo Medellín y una más, si el volumen asusta |
| SUI | Si el archivo es anual, la calibración es municipio × año. Si es mensual, municipio × mes |
| EPM | Si la fila es circuito y no trae código de municipio, queda fuera de este corte |
| Hora del clima | Se copia el tipo que vino en el JSON (número o fecha-hora) y el calendario se arma igual, para que el cruce no salga vacío |

Listo cuando: existe un párrafo con año, lista de municipios del dato fino, grano de SUI y si EPM entra o se aplaza.

## Paso 4 — Estrella con columnas reales

Objetivo: pasar la ficha de [02-cruce-y-generacion.md](02-cruce-y-generacion.md) a los nombres de columna que trajeron los archivos.

Cómo:

- Para cada cruce de esa ficha, escribe la columna real de la izquierda y la de la derecha.
- Si una columna no existe en el archivo, el cruce se aplaza. No se inventa.
- Se mantienen las dos sumas de antes: consumo a mes antes de la factura; fallas contadas aparte de la lluvia.
- Aquí entra el diseño del modelo (hechos, dimensiones y medidas). Todavía no se crean las tablas ni el archivo de Power BI.

Listo cuando: cada cruce que sigue en pie tiene columnas reales de los dos lados, y los cruces aplazados están nombrados.

## Paso 5 — Cargar lo público

Objetivo: tener en PostgreSQL con PostGIS el lugar y el contexto, antes de inventar la empresa.

Orden de carga, el mismo de la generación:

1. Municipios de Antioquia, con `cod_divipola`.
2. Polígono de cada municipio pegado a ese código. Si el mapa trae solo el nombre, se traduce una vez y después se usa el código.
3. Calendario del periodo elegido en el paso 3.
4. Clima de la cabecera, con el `cod_divipola` ya escrito.
5. Población, SUI y EPM (si entró), cada uno en su grano.

Cómo comprobarlo:

- Un municipio de prueba tiene código, nombre y polígono.
- El clima de ese municipio tiene filas por hora dentro del periodo.
- SUI o DANE de ese municipio se encuentran por el mismo código, no por el nombre.

Listo cuando: esa comprobación pasa en al menos dos municipios.

## Paso 6 — Generar la empresa ficticia

Objetivo: crear el cajón A calibrado con los totales del paso 5. Cada fila lleva `cod_divipola` o una llave que llega a él.

Orden:

1. Clientes de las ciudades del paso 3. La cantidad se orienta con los suscriptores de SUI, si el archivo los trajo. La población del DANE no se copia como lista de clientes.
2. Activos con un punto dentro del polígono de su municipio. Se confirma el código con el polígono.
3. Consumo por cliente y hora, con estación y franja del día.
4. Factura del mes: suma del consumo de ese cliente en ese mes, más valor, tarifa y si pagó.
5. Fallas sobre activos. Más fallas si el activo es viejo y si en esa ciudad y esa hora hubo lluvia o calor.
6. Mantenimientos sobre activos.

Listo cuando: un cliente de prueba tiene horas de consumo, una factura del mes coherente con esa suma, y sus fallas caen en activos de su mismo municipio.

## Paso 7 — Comprobar que los números no se inflan

Objetivo: revisar los cruces con datos ya cargados, antes de pintarlos.

| Prueba | Resultado esperado |
|---|---|
| Sumar el consumo horario de un cliente en un mes | Igual al kWh de su factura de ese mes |
| Contar fallas de un municipio en una hora | Una fila. La lluvia de esa hora aparece una sola vez |
| Buscar un cliente en SUI o en el DANE | No aparece. Esos archivos no tienen `cliente_id` |
| Unir dos municipios por nombre | No se usa. El mismo código aparece en geografía, cliente, clima y SUI |
| Total simulado del municipio | Está en el mismo orden de magnitud que el total de SUI, si SUI trajo consumo o suscriptores. Es calibración, no una copia exacta |

Listo cuando: las cinco pruebas están anotadas, con un municipio y un mes de ejemplo.

## Paso 8 — Power BI: modelo

Objetivo: conectar el almacén y publicar relaciones y medidas mínimas.

Medidas de esta etapa, no más:

- kWh medidos.
- Valor facturado, kWh facturados, facturas pagadas, pendientes y en mora.
- Cantidad de incidentes, clientes afectados y minutos.
- Fallas por activo, estado del activo, costo de mantenimiento.
- Lluvia y temperatura junto al conteo de incidentes, al grano municipio y hora.

La demanda de XM no entra. SAIDI y SAIFI no se calculan sobre lo simulado. Si SUI los trae, se muestran como cifra publicada del municipio.

Listo cuando: un filtro de municipio cambia consumo, fallas y clima a la vez, y el kWh medido no se mezcla con el facturado.

Abres el archivo en Power BI Desktop y actualizas los datos antes de dibujar páginas.

## Paso 9 — Páginas del tablero

Una página por pregunta. El contexto va aparte.

| Página | Pregunta | Qué muestra |
|---|---|---|
| Consumo | ¿Cómo evoluciona el consumo? | kWh medidos por fecha, tipo de cliente, estrato y municipio |
| Facturación | ¿Cómo está la cuenta del mes? | Valor, kWh facturados y estado de pago |
| Incidentes | ¿Dónde se concentran las fallas? | Mapa, cantidad, clientes afectados y minutos |
| Infraestructura | ¿Qué activo falla más? | Fallas, estado, antigüedad y costo de mantenimiento |
| Clima | ¿La lluvia acompaña las fallas? | Lluvia y conteo de incidentes, municipio y hora |
| Contexto | ¿Cómo se ve el municipio en las fuentes públicas? | Población, SUI y EPM, cada uno en su grano, etiquetado como contexto |

“Zona de mayor riesgo” en esta etapa es un ranking: más incidentes, más clientes afectados, más minutos, activos en falla o más antiguos. No es una predicción.

Listo cuando: las seis páginas filtran por el mismo municipio y ninguna suma la lluvia sobre cada falla.

## Paso 10 — Cierre de la Etapa 1

Objetivo: comprobar el alcance y dejar escrito lo que quedó hecho.

Se revisa que el tablero cubra consumo, facturación, incidentes, infraestructura, clima y contexto. No se promete anomalías, predicción ni preguntas en lenguaje natural.

Se anota en la documentación: periodo, municipios del dato fino, grano real de SUI, si EPM entró, y las pruebas del paso 7.

## Qué se aplaza a propósito

| Tema | Cuándo se retoma |
|---|---|
| Comuna o barrio | Si OpenStreetMap trae ese polígono en todos los municipios del corte. Si no, la llave sigue siendo el municipio |
| Circuito de EPM | Cuando exista una tabla circuito → `cod_divipola` |
| XM | Después de la Etapa 1, solo como contexto por fecha |
| Anomalías, predicción y asistente | Etapa 2, sobre esta misma base |
| Los 125 municipios con consumo horario | Solo si el primer corte, con pocas ciudades, ya está estable |
