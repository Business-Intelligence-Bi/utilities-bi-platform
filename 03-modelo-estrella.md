# Modelo estrella provisional

Aún no existe en PostgreSQL ni en Power BI. Esta es la forma que ya se puede ver, para entenderla antes de bajar los archivos.

Las muestras del cajón B confirman nombres de columnas y algunos granos. No inventan la estrella. El dibujo está en el lienzo `makaia-modelo-estrella`. El orden de trabajo sigue en [00-plan-de-trabajo.md](00-plan-de-trabajo.md).

## La forma

En el centro están los hechos: lo que se mide. Alrededor, cuatro dimensiones: quién, cuándo, dónde y sobre qué activo.

```text
                         TIEMPO
                    fecha, hora, mes
                           |
                           |
     CLIENTE ——— HECHOS ——— GEOGRAFÍA
   cliente_id               cod_divipola
                           |
                           |
                        ACTIVO
                 infraestructura_id
```

El centro contiene cinco hechos:

| Hecho | Una fila es | Se cuelga de | Qué guarda |
|---|---|---|---|
| Consumo | Un cliente en una hora | Cliente y Tiempo | kWh medido |
| Facturación | Un cliente en un mes | Cliente y Tiempo | kWh del mes, valor, tarifa, si pagó |
| Incidentes | Una falla | Activo y Tiempo | Clientes afectados y minutos |
| Mantenimiento | Un trabajo | Activo y Tiempo | Tipo, costo y duración |
| Clima | Un municipio en una hora | Geografía y Tiempo | Lluvia, temperatura, viento |

Dos caminos no van directo al centro:

- El consumo llega a la ciudad pasando por el cliente. El cliente es quien trae `cod_divipola`.
- La falla y el mantenimiento llegan a la ciudad pasando por el activo. El activo recibió su código porque el punto cayó dentro del polígono.

## De dónde sale cada caja

| Caja | Cómo nace |
|---|---|
| Geografía | Se arma con el código DIVIPOLA y el polígono de OpenStreetMap |
| Tiempo | Se construye para el periodo elegido. No se descarga |
| Cliente | Lo genera el simulador, ya con `cod_divipola` |
| Activo | Lo genera el simulador, con un punto dentro del polígono |
| Consumo, facturación, incidentes, mantenimiento | Los genera el simulador, en el orden del documento de cruce |
| Clima | Llega de Open-Meteo por coordenada. Al guardarlo se le escribe el código del municipio |

## Contexto, fuera del centro

SUI, la población del DANE y EPM abierto no son hechos de la empresa ficticia. Se sientan al lado y solo tocan Geografía y Tiempo.

| Tabla | Grano provisional | Llave |
|---|---|---|
| SUI | Municipio y periodo | `cod_divipola` + `periodo` |
| DANE población | Municipio y año | `cod_divipola` + año |
| EPM abierto | Municipio y periodo, si el archivo trae el código | `cod_divipola` + `periodo` |

## Qué puede cambiar al abrir un archivo

| Si la muestra muestra esto | Qué se ajusta |
|---|---|
| SUI es anual | El grano de SUI pasa a municipio × año |
| La hora del clima es una fecha-hora, no un número | Tiempo se arma de esa misma forma |
| EPM trae circuito y no trae código | Esa tabla se queda fuera de este corte |

Esas tres cosas cambian una etiqueta. La estrella sigue igual: hechos en el centro, cliente, tiempo, geografía y activo alrededor.
