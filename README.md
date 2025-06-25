# Análisis del Sector Pesquero Vasco: Metodología y Conclusiones

Estructura del Trabajo

## 1. Introducción
El sector pesquero ha sido históricamente una actividad económica y cultural fundamental en el País Vasco. A lo largo de los siglos, la pesca ha moldeado la identidad de numerosas comunidades costeras, funcionando no solo como fuente de empleo, sino como pilar en la construcción de tradiciones, gastronomía y formas de vida. En un territorio íntimamente ligado al mar Cantábrico, la evolución de la producción pesquera —tanto en volumen como en valor económico— refleja transformaciones profundas en la economía regional, las políticas europeas y los patrones de consumo de productos marinos.

Este trabajo analiza la importancia actual del sector pesquero en el País Vasco, evaluando su peso económico y su evolución reciente. El estudio abarca las principales especies capturadas, los volúmenes de producción, la evolución de los precios y los factores determinantes de estos cambios, como la sostenibilidad, la innovación tecnológica y la normativa comunitaria. El análisis busca comprender el papel actual de la pesca en el desarrollo económico vasco y sus perspectivas ante los desafíos medioambientales y sociales.

### Objetivos específicos:
Además del análisis económico, se evaluarán correlaciones entre producción y valor, eficiencia de la flota y rentabilidad por especie

### Relevancia:
impacto social y cultural de los hallazgos

# 2. Metodología


## 2.1 Fuentes de Datos y Estructura
Se ha usado las fuentes seguientes 
[Banco de datos - Pesca](https://www.euskadi.eus/gobierno-vasco/-/estadistica/banco-de-datos-pesca/)
(https://www.eustat.eus/estadisticas/tema_496/)

he descargado varios datos en formato excel, he elegidos los datos que podrian responder a la preguntas, y he limpiado los csv ordenados
he creado una base de datos en postgres, y he aportado los datos con dos formas, con Create table y aportacion csv, tambien otra manera con python
las seguientes son las tablas creadas:

Buques: Evolución de la flota por tipo (2010-2022)
Cantidad_pesca: Capturas anuales por especie (en toneladas métricas)
Valor_pesca: Valor económico por especie (en miles de €)
Flota_pesquera: Capacidad productiva (arqueo, potencia, personal)
Rentabilidad: Indicadores financieros del subsector

``` SQL
CREATE DATABASE agribask;
```
``` SQL

