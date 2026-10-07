# Enlaces de las fuentes

Dónde bajar cada archivo del cajón B, y qué no se descarga porque se genera.

El paso 0 del [plan](00-plan-de-trabajo.md) es leer los documentos. No pide archivos. La descarga es el **paso 1**. Este archivo es la lista para ese paso.

Guarda lo que bajes en `muestras/`, con el nombre de la fuente y la fecha. No transformes la primera copia. Los enlaces se revisaron el 6 de octubre de 2026. Si un portal cambia el botón, manda la ficha de la página, no un enlace viejo copiado de memoria.

## Qué se descarga y qué se genera

| Origen | ¿Hay enlace? | Qué hacer ahora |
|---|---|---|
| DIVIPOLA | Sí | Bajar el CSV de Antioquia |
| Polígono del municipio | Sí | Bajar el shapefile del DANE, nivel municipio |
| OpenStreetMap | Sí | Dejarlo para después. El archivo de Colombia pesa unos 314 MB y no trae el código DIVIPOLA |
| Clima Open-Meteo | Sí | Pedir 7 días de 2 o 3 cabeceras, en JSON |
| Población DANE | Sí | Bajar la serie municipal 2018-2042 |
| SUI | Sí | Para la muestra de hoy, dos CSV chicos de datos.gov.co. El reporte de EPM por ciudad queda para después |
| EPM tarifas | Sí. Ya hay una copia en `muestras/` | Precio publicado, diciembre de 2016 a diciembre de 2021. No es la falla ni el cliente |
| EPM interrupciones de energía | No hay archivo histórico encontrado | No uses los archivos de interrupciones de acueducto. La falla simulada se genera en el paso 6 |
| Clientes, consumo, facturas, activos, fallas, mantenimientos | No | Se inventan con un script, después de ver estas muestras |

## 1. DIVIPOLA — códigos de municipio

Rol: padrón de ciudades. La llave `cod_mpio` es el `cod_divipola` de 5 dígitos.

| Qué | Enlace |
|---|---|
| Ficha en datos.gov.co | https://www.datos.gov.co/Mapas-Nacionales/DIVIPOLA-C-digos-municipios/gdxc-w37w |
| CSV solo Antioquia | https://www.datos.gov.co/resource/gdxc-w37w.csv?cod_dpto=05 |
| Excel nacional, listado vigente | https://geoportal.dane.gov.co/servicios/descarga-y-metadatos/datos-geoestadisticos/?cod=113 |

En la página del Excel, baja **Listado completos de Codificación Divipola (Actual-vigente)**. Para la muestra basta el CSV de Antioquia.

Columnas que ya se vieron en el archivo: `cod_dpto`, `dpto`, `cod_mpio`, `nom_mpio`, `tipo_municipio`, `longitud`, `latitud`.

Medellín llega así: `cod_mpio` `05001`, latitud `6,246631`, longitud `-75,581775`. El archivo usa coma decimal. En una URL de clima esa coma se cambia por punto.

El nombre es etiqueta. El cruce usa `cod_mpio`.

## 2. Polígono del municipio

Rol: dibujo de cada municipio, ya con el código, para saber en qué ciudad cae un punto.

Para el primer archivo no uses OpenStreetMap. El Marco Geoestadístico del DANE trae el polígono y el código de 5 dígitos en el campo `MPIO_CDPMP`.

| Qué | Enlace |
|---|---|
| Página de descarga MGN 2025 | https://geoportal.dane.gov.co/servicios/descarga-y-metadatos/datos-geoestadisticos/?cod=111 |
| Diccionario de campos de la capa Municipio | https://geoportal.dane.gov.co/mparcgis/rest/services/MGN2025/Serv_CapasMGN_2025/FeatureServer/layers |

En esa página baja **Versión MGN2025-Nivel Municipio**, shapefile, unos 40 MB. No bajes el ZIP de todos los niveles: pesa 1,5 GB.

Después de abrirlo, filtra `DPTO_CCDGO = 05`. El código del municipio es `MPIO_CDPMP`. El nombre es `MPIO_CNMBRE` y no se usa para cruzar.

OpenStreetMap queda como mapa complementario, no como padrón:

| Qué | Enlace |
|---|---|
| Página del extracto de Colombia | https://download.geofabrik.de/south-america/colombia.html |
| Archivo PBF, unos 314 MB | https://download.geofabrik.de/south-america/colombia-latest.osm.pbf |

Licencia ODbL: hay que citar OpenStreetMap. Ese archivo no trae `cod_divipola`. Si más adelante se usa, el nombre se traduce una vez al código y no se vuelve a usar como llave.

## 3. Clima — Open-Meteo

Open-Meteo es el clima real de un punto del mapa, hora por hora. Responde si en esa ciudad, a esa hora, llovió o hizo calor.

No trae el código del municipio. Trae la coordenada de la cabecera. El `cod_mpio` se le escribe al guardar, usando el DIVIPOLA. Un activo no tiene clima propio: hereda el de su municipio.

| Qué | Enlace |
|---|---|
| Documentación | https://open-meteo.com/en/docs/historical-weather-api |
| Condiciones de uso | https://open-meteo.com/en/terms |
| Servicio | `https://archive-api.open-meteo.com/v1/archive` |

Sin clave para uso no comercial. Hay que citar la fuente. Uso comercial o de alto volumen pide plan de pago.

La primera muestra son 7 días de Medellín, del 1 al 7 de enero de 2024. No son los 125 municipios ni un año. Ábrela en el navegador y guarda el JSON:

https://archive-api.open-meteo.com/v1/archive?latitude=6.246631&longitude=-75.581775&start_date=2024-01-01&end_date=2024-01-07&hourly=temperature_2m,precipitation,relative_humidity_2m,wind_speed_10m,surface_pressure&timezone=America/Bogota

Cómo se lee esa URL:

- `latitude=6.246631` y `longitude=-75.581775` son la cabecera de Medellín, copiadas del DIVIPOLA. En el CSV venían con coma (`6,246631`). En la URL la coma se cambia por punto.
- `start_date` y `end_date` marcan esos 7 días.
- `hourly=temperature_2m,precipitation,...` pide temperatura, lluvia, humedad, viento y presión, una fila por hora.
- `timezone=America/Bogota` deja la hora en hora de Colombia.

Dentro del JSON, `time` es la hora (`2024-01-01T18:00`) y `precipitation` es la lluvia en milímetros. Eso se anota en la bitácora.

Para una segunda ciudad se copia su latitud y longitud del CSV, se cambia la coma por punto y se arma la misma URL. Ejemplo de cruce, con números inventados: Medellín `05001`, 15 de enero de 2026, hora 18, lluvia 8,5 mm, y 3 fallas en esa misma ciudad y hora. La lluvia queda en su fila. Las fallas se cuentan en otra. Esas dos filas se igualan por código, fecha y hora. La lluvia no se suma encima de cada falla ni se une al medidor.

## 4. Población — DANE

Sirve para saber cuántas personas estima el DANE en cada municipio, en cada año. No dice quiénes son. No es la lista de clientes de EPM.

| Qué | Enlace |
|---|---|
| Página de proyecciones | https://www.dane.gov.co/index.php/estadisticas-por-tema/demografia-y-poblacion/proyecciones-de-poblacion |

En esa página hay varias tablas. Las que dicen **nacional** (periodos 1950-2017 y 2018-2070) son Colombia entera. No traen municipio ni código. Esas no se bajan.

La que sí se baja está más abajo:

**Serie municipal de población por área, para el periodo 2018-2042**

En la página figura actualizada el 8 de agosto de 2025. El botón Descargar está en esa fila. “Por área” separa cabecera y resto rural. Si se suman, queda el total del municipio. La serie por sexo y edad es más detalle del que hace falta en esta muestra.

Qué trae el archivo:

- 2018 es la base, el censo.
- Cada año siguiente, hasta 2042, es una proyección: la población que el DANE espera en ese municipio.
- De 2018 al año en curso el número es una estimación de un año que ya pasó. De ahí a 2042 es lo que se espera más adelante.

En el tablero se usa el año que coincida con el resto de los datos, por ejemplo 2024, pegado al `cod_mpio`. No se reparte entre los clientes simulados. El DANE dice habitantes. El SUI, si entra, dice suscriptores del servicio. Son dos cifras distintas.

En la bitácora anota si el municipio viene con código de 5 dígitos o solo con nombre, y si el año es una columna o está en columnas separadas.

## 5. SUI — totales del sector

El SUI es el informe oficial que las empresas eléctricas le entregan al Estado. No es EPM por dentro y no es la lista de clientes.

Una fila puede decir: “EPM, Medellín, 2024: tantos suscriptores y tantos kWh facturados”. El simulador crea clientes falsos en Medellín. Si el SUI dice un total y la suma simulada queda muy lejos, la ciudad quedó mal calibrada. Se parece en orden de magnitud. No es una copia exacta. El SUI no dice quién es cada cliente ni qué marcó el medidor a las 6 de la tarde.

Se cruza solo con la geografía, por el código del municipio y el periodo. Si el archivo trae el nombre y no el código, el nombre se traduce una vez al `cod_mpio` del DIVIPOLA.

### Muestra de hoy, sin entrar al portal

Abre estos dos enlaces y guarda cada CSV en `muestras/`:

| Qué es | Enlace de la muestra |
|---|---|
| Facturación, 200 filas. Año, mes, kWh y valor | https://www.datos.gov.co/resource/gw2d-7n7y.csv?$limit=200 |
| Caracterización de usuarios, 200 filas. Nombre del municipio, estrato y circuito. Corte de 2021 | https://www.datos.gov.co/resource/endg-udcv.csv?$limit=200 |

Fichas completas, por si hace falta ver la página:

- Facturación: https://www.datos.gov.co/Minas-y-Energ-a/Superservicios-Facturaci-n-a-Usuarios-Energia/gw2d-7n7y
- Caracterización: https://www.datos.gov.co/Minas-y-Energ-a/Superservicios-Caracterizaci-n-de-Usuarios-Energia/endg-udcv

Límites de esas dos muestras:

- La facturación revisada empieza en el Quindío, no en Antioquia, y no trae `cod_mpio`. No se une a un cliente.
- La caracterización trae `dane_nom_dpto` y `dane_nom_mpio`, el nombre, no el código. Si se filtra Antioquia, el nombre se traduce una sola vez al `cod_mpio`.

Con estos dos CSV ya se ve la forma del SUI. Pueden quedar en pausa y seguir con el clima, la población, los códigos y las tarifas.

### El archivo de EPM por ciudad, después

En el portal no aparece la palabra EPM en la primera lista. Esa lista son tipos de reporte, no empresas.

| Qué | Enlace |
|---|---|
| Reportes de energía | https://sui.superservicios.gov.co/Reportes-del-Sector/Energia |
| Índice de reportes del sector | https://sui.superservicios.gov.co/Reportes-del-Sector |

Cuando se retome:

1. Abre Reportes de energía.
2. Entidad: Energía. Categoría: Comerciales. Pulsa Aplicar.
3. Entra a **Consolidado energía por empresa departamento y municipio**. Ese baja a la ciudad. El consolidado solo por departamento se queda en el departamento.
4. Dentro, la empresa no dice “EPM”. Busca **Empresas Públicas de Medellín**. Elige un solo año y, si deja, Antioquia.
5. Exporta y anota el nombre del reporte, el año y si el municipio trae código o solo nombre.

Los reportes que dicen **ZNI** no sirven. Esa es la zona no interconectada, y el servicio de EPM en Antioquia no va por ahí.

SAIDI y SAIFI son la duración y la frecuencia de los cortes que publicó el sector. El PDF se lee. No se calculan con las fallas simuladas ni se cruzan fila a fila:

https://www.superservicios.gov.co/sites/default/files/inline-files/Informe-de-Calidad-del-Servicio-de-Energia-2023.pdf

## 6. EPM — tarifas publicadas

Esta fuente muestra el precio real de la energía que EPM publicó. No es la empresa por dentro, no es el cliente y no es la falla.

Sirve para dos cosas:

- Ver cómo se movió la tarifa del mercado regulado.
- Orientar el campo `tarifa` de la factura simulada, para que el precio no salga al azar.

La factura de cada cliente se sigue generando en el simulador. Este archivo no trae `cliente_id`, ni el consumo de una hora, ni un transformador. Si no trae código de municipio, se guarda como la tarifa general de EPM en ese mes y no se cruza con una ciudad.

Es distinto del SUI. El SUI dice cuánto servicio reportó la empresa en el territorio. Este archivo dice cuánto costaba el kilovatio.

| Qué | Enlace |
|---|---|
| Cómo buscarlos, según EPM | https://www.epm.com.co/institucional/datos-abiertos-epm/ |
| Buscador | https://www.datos.gov.co/browse?q=EPM |
| Ficha que se abre | https://www.datos.gov.co/dataset/Tarifas-para-Servicios-de-Energ-a-EPM/se7p-ytdd |
| Otra ficha del mismo título, por si la primera sale vacía | https://www.datos.gov.co/d/sfcd-b3ey |

En la ficha se baja el botón **TEXT/CSV**. El JSON es la misma tabla. El diccionario solo explica los nombres de las columnas. Guarda una sola copia.

La página, vista el 7 de octubre de 2026, dice: tarifas y costo de energía eléctrica, mercado regulado, de **diciembre de 2016 a diciembre de 2021**. La última actualización de la ficha figura el 31 de octubre de 2025. La marca de tiempo `2021-12-21` es el cierre de la serie, no un error.

Si el resto de las muestras es de 2024, esta tarifa no coincide en el tiempo. Igual se guarda. Al cerrar el alcance se decide si entra en el primer corte o se queda como referencia de 2016 a 2021.

No descargues nada que diga acueducto. Estas dos son de agua, no de fallas eléctricas:

- https://www.datos.gov.co/Funci-n-p-blica/Interrupciones-de-Acueducto-Aguas-Nacionales-EPM/mvuk-tydp
- https://www.datos.gov.co/Planeaci-n/Interrupciones-programadas-del-servicio-de-acueduc/r9fv-awbc

El visor de interrupciones de energía es una consulta en línea, no un histórico para guardar:

https://aplicaciones.epm.com.co/interrupcionesenergia/

Las fallas del proyecto se generan después, con el simulador. EPM no las reemplaza.

## 7. Generación — cajón A

No hay enlace. Clientes, consumo, facturación, activos, incidentes y mantenimientos no se descargan. Los escribe un script de Python cuando ya existan el código del municipio, el polígono y los totales de las muestras.

El orden está en [02-cruce-y-generacion.md](02-cruce-y-generacion.md), pasos 6 a 11. Eso es el paso 6 del plan, no este.

Hasta entonces no se busca un dataset de “la empresa”. Una curva horaria de otro país, si más adelante se usa para ver la forma del día, no entra al modelo y no es Colombia. El documento inicial citaba el conjunto Household Electric Power Consumption. La ficha oficial está en:

https://archive.ics.uci.edu/dataset/235/individual+household+electric+power+consumption

No lo bajes en el paso 1.

## Orden de esta semana

1. CSV de DIVIPOLA, Antioquia.
2. Shapefile MGN 2025, nivel municipio.
3. JSON de 7 días de clima para Medellín y otras dos cabeceras, con coordenadas del CSV.
4. Serie de población municipal 2018-2042.
5. Los dos CSV chicos del SUI. El reporte de EPM por ciudad puede esperar.
6. El CSV de tarifas de EPM. Botón TEXT/CSV. Diciembre de 2016 a diciembre de 2021.

En cada archivo llena la bitácora del [plan](00-plan-de-trabajo.md), paso 2. La pregunta que no se puede dejar en blanco es: ¿el municipio trae código de 5 dígitos o solo el nombre?
