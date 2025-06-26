Análisis del Sector Pesquero Vasco


# 1. Introducción

El sector pesquero ha sido históricamente una actividad económica y cultural fundamental en el País Vasco. A lo largo de los siglos, la pesca ha moldeado la identidad de numerosas comunidades costeras, funcionando no solo como fuente de empleo, sino como pilar en la construcción de tradiciones, gastronomía y formas de vida. En un territorio íntimamente ligado al mar Cantábrico, la evolución de la producción pesquera —tanto en volumen como en valor económico— refleja transformaciones profundas en la economía regional, las políticas europeas y los patrones de consumo de productos marinos.

Este trabajo analiza la importancia actual del sector pesquero en el País Vasco, evaluando su peso económico y su evolución reciente. El estudio abarca las principales especies capturadas, los volúmenes de producción, la evolución de los precios y los factores determinantes de estos cambios, como la sostenibilidad, la innovación tecnológica y la normativa comunitaria. El análisis busca comprender el papel actual de la pesca en el desarrollo económico vasco y sus perspectivas ante los desafíos medioambientales y sociales.### Objetivos específicos:Además del análisis económico, se evaluarán correlaciones entre producción y valor, eficiencia de la flota y rentabilidad por especie### Relevancia:impacto social y cultural de los hallazgos

## Objetivos específicos:

Además del análisis económico,  se evaluarán correlaciones entre producción y valor, eficiencia de la flota y rentabilidad por especie

## Herramientas Utilizadas:

PostgreSQL para gestión y consultas SQL
Python (pandas, seaborn, matplotlib) para análisis estadístico y visualización
Tableau, no me llego a desarollarlo aqui

# 2. Metodología

## 2.1 Heramientas usadas

## 2.2 Fuentes de Datos y EstructuraSe

He usado las fuentes seguientes:
[Banco de datos - Pesca](https://www.euskadi.eus/gobierno-vasco/-/estadistica/banco-de-datos-pesca/)
(https://www.eustat.eus/estadisticas/tema_496/)

He descargado varios datos en formato excel, he elegidos los datos que podrian responder a la preguntas, y he limpiado los csv ordenadoshe creado una base de datos en postgres, y he aportado los datos con dos formas, con Create table y aportacion csv, tambien otra manera con pythonlas seguientes son las tablas creadas:

Buques: Evolución de la flota por tipo (2010-2022)
Cantidad_pesca: Capturas anuales por especie (en toneladas métricas)
Valor_pesca: Valor económico por especie (en miles de €)
Flota_pesquera: Capacidad productiva (arqueo, potencia, personal)Add commentMore actions
Rentabilidad: Indicadores financieros del subsector

``` SQL
CREATE DATABASE agribask;
```
Ejemplo de Insercion de una Tabla con Python

``` Python
import pandas as pd
import psycopg2
from sqlalchemy import create_engine

# Configuración de la base de datos
DB_CONFIG = {
    'host': 'localhost',
    'database': 'agribask',
    'user': 'postgres',
    'password': 'Floridablanca134',
    'port': '5432'
}

CSV_PATH = r"C:\Users\database\Desktop\pescavasca\Buques_total.csv"

def leer_csv_correctamente(ruta_archivo):
    """Lee el CSV saltando las filas de metadatos"""
    # Primero detectamos cuántas líneas hay que saltar
    with open(ruta_archivo, 'r', encoding='latin1') as f:
        lineas = f.readlines()
    
    # Buscamos la línea que contiene los encabezados reales
    for i, linea in enumerate(lineas):
        if linea.startswith('"periodo"'):
            skiprows = i
            break
    else:
        skiprows = 0  # Si no encuentra el patrón, no saltar líneas
    
    # Leer el CSV saltando las filas de metadatos
    return pd.read_csv(ruta_archivo, encoding='latin1', skiprows=skiprows)

def limpiar_datos(df):
    """Realiza la limpieza y conversión de datos"""
    # Eliminar comillas de los nombres de columnas si las hay
    df.columns = df.columns.str.strip('"')
    
    # Verificar que tenemos las columnas esperadas
    columnas_esperadas = ['periodo', 'Total', 'Bajura', 'Altura al fresco', 
                         'Arrastreros congeladores', 'Atuneros congeladores', 'Bacaladeros']
    
    for col in columnas_esperadas:
        if col not in df.columns:
            raise ValueError(f"Columna esperada no encontrada: {col}")
    
    # Renombrar columnas
    mapeo_nombres = {
        'periodo': 'anio',
        'Total': 'total_buques',
        'Bajura': 'bajura',
        'Altura al fresco': 'altura_al_fresco',
        'Arrastreros congeladores': 'arrastreros_congeladores',
        'Atuneros congeladores': 'atuneros_congeladores',
        'Bacaladeros': 'bacaladeros'
    }
    df = df.rename(columns=mapeo_nombres)
    
    # Convertir datos
    df['anio'] = df['anio'].astype(int)
    
    # Reemplazar 0 con NULL donde corresponda
    df['arrastreros_congeladores'] = df['arrastreros_congeladores'].replace(0, None)
    
    return df

def crear_tabla_postgres(conn):
    """Crea la tabla en PostgreSQL si no existe"""
    query = """
    CREATE TABLE IF NOT EXISTS flota_pesquera (
        anio INT PRIMARY KEY,
        total_buques INT,
        bajura INT,
        altura_al_fresco INT,
        arrastreros_congeladores INT,
        atuneros_congeladores INT,
        bacaladeros INT
    )
    """
    with conn.cursor() as cursor:
        cursor.execute(query)
    conn.commit()

def cargar_datos_postgres(conn, df):
    """Carga los datos en la tabla PostgreSQL"""
    try:
        engine = create_engine(
            f"postgresql://{DB_CONFIG['user']}:{DB_CONFIG['password']}@"
            f"{DB_CONFIG['host']}:{DB_CONFIG['port']}/{DB_CONFIG['database']}"
        )
        df.to_sql('flota_pesquera', engine, if_exists='replace', index=False)
        print("¡Datos cargados exitosamente en PostgreSQL!")
    except Exception as e:
        print(f"Error al cargar datos: {e}")

def main():
    try:
        print("\n=== Procesando archivo CSV ===")
        
        # 1. Leer el archivo correctamente
        df = leer_csv_correctamente(CSV_PATH)
        print("\nColumnas detectadas en el CSV:")
        print(df.columns.tolist())
        
        # 2. Limpiar y transformar los datos
        df = limpiar_datos(df)
        print("\nDatos procesados (primeras filas):")
        print(df.head())
        
        # 3. Conectar a PostgreSQL y cargar datos
        conn = psycopg2.connect(**DB_CONFIG)
        print("\n¡Conexión a PostgreSQL establecida!")
        
        crear_tabla_postgres(conn)
        cargar_datos_postgres(conn, df)
        
    except Exception as e:
        print(f"\nERROR: {str(e)}")
    finally:
        if 'conn' in locals():  # Verifica si 'conn' existe
            conn.close()        # Cierra la conexión si existe
            print("Conexión cerrada.")
```
Ejemplo de Insercion de una Tabla con SQL
``` SQL
CREATE TABLE indicadores_rentabilidad (
    año INTEGER,
    subsector TEXT,
    importe_neto_cifra_negocio NUMERIC,
    variacion_existencias_producto NUMERIC,
    trabajos_realizados_para_activo NUMERIC,
    aprovisionamientos NUMERIC,
    otros_ingresos_explotacion NUMERIC,
    ingresos_accesorios NUMERIC,
    subvencion_explotacion NUMERIC,
    gastos_personal NUMERIC,
    otros_gastos_explotacion NUMERIC,
    servicios_exteriores NUMERIC,
    impuestos_actividad NUMERIC,
    otros_gastos_gestion_corrientes NUMERIC,
    amortizacion NUMERIC,
    otros_ingresos_gastos_explotacion NUMERIC,
    resultado_explotacion NUMERIC,
    ingresos_financieros NUMERIC,
    gastos_financieros NUMERIC,
    resultado_financiero NUMERIC,
    resultado_antes_impuestos NUMERIC,
    impuesto_sobre_beneficios NUMERIC,
    resultado_ejercicio_operaciones_continuadas NUMERIC
);
```

Ejemplo de cambiar el nombre de las columnas para facilitar la lectura 

```SQL
ALTER TABLE rentabilidad RENAME COLUMN "importe_neto_cifra_negocio" TO ventas_netas;
ALTER TABLE rentabilidad RENAME COLUMN "variacion_existencias_producto" TO var_existencias;
```

## 2.3 Enfoque Analítico

## a) Análisis descriptivo:

Evolución temporal de capturas y valores
Comparativa entre especies
Relación flota-capacidad productiva

## b) Análisis de correlación:

Relación cantidad-valor por especie
Eficiencia económica (valor por tonelada)
Indicadores de rentabilidad

## c) Visualización:

Gráficos temporales
Diagramas de dispersión para correlaciones
Mapas de calor para relaciones multivariable

# 3. Análisis y Resultados

  3.1 Valor por Tonelada por Especie
Presentar los resultados de tu vista valor_por_tonelada:
``` SQl
CREATE OR REPLACE VIEW valor_por_tonelada AS
SELECT 
  anio,
  ROUND((v.anchoa_mil::numeric / NULLIF(cantidad.anchoa_tm::numeric, 0)), 2) AS anchoa_valor_tm,
  ROUND((v.atun_rojo_mil::numeric / NULLIF(cantidad.atun_rojo_tm::numeric, 0)), 2) AS atun_rojo_valor_tm,
  ROUND((v.bonito_norte_mil::numeric / NULLIF(cantidad.bonito_norte_tm::numeric, 0)), 2) AS bonito_valor_tm,
  ROUND((v.chicharro_mil::numeric / NULLIF(cantidad.chicharro_tm::numeric, 0)), 2) AS chicharro_valor_tm,
  ROUND((v.merluza_mil::numeric / NULLIF(cantidad.merluza_tm::numeric, 0)), 2) AS merluza_valor_tm,
  ROUND((v.besugo_mil::numeric / NULLIF(cantidad.besugo_tm::numeric, 0)), 2) AS besugo_valor_tm,
  ROUND((v.verdel_estornino_tm::numeric / NULLIF(cantidad.verdel_estornino_tm::numeric, 0)), 2) AS verdel_valor_tm,
  ROUND((v.otras_especies_mil::numeric / NULLIF(cantidad.otras_especies_tm::numeric, 0)), 2) AS otras_valor_tm
FROM valor_pesca v
JOIN cantidad_pesca cantidad USING(anio);
```
``` SQL
-- Especies más rentables en el último año disponible
SELECT anio, anchoa_valor_tm, merluza_valor_tm, atun_rojo_valor_tm
FROM valor_por_tonelada
ORDER BY anio DESC LIMIT 5;
```
------ claramente 

## 3.2 Correlaciones entre variables
Existe correlación entre cantidad y valor por especie

``` Python
# Análisis de correlación mejorado
especies = ['anchoa', 'atun_rojo', 'merluza', 'bonito_norte']
corr_results = []

for esp in especies:
    corr = df[[f'cantidad_{esp}', f'valor_{esp}']].corr().iloc[0,1]
    corr_results.append({'Especie': esp, 'Correlación': corr})

pd.DataFrame(corr_results).sort_values('Correlación', ascending=False)
```

### 3.4 Vista especie_valor_cantidad_long

```SQL
CREATE OR REPLACE VIEW especie_valor_cantidad_long AS
SELECT anio, 'anchoa' AS especie, anchoa_cantidad AS cantidad_tm, anchoa_valor AS valor_eur 
FROM especie_valor_cantidad
UNION ALL
-- Resto de especies...;
```

### 3.5 Vista personal_por_buque

```SQL
sql
CREATE OR REPLACE VIEW personal_por_buque AS
SELECT 
    f.anio,
    ROUND((f.personal::numeric / NULLIF(f.num_buques, 0)), 2) AS personas_por_buque,
    f.tipo_pesca
FROM flota_pesquera f;
```


### 3.6Rentabilidad por subsector

```SQL
SELECT anio, subsector,
       ROUND(resultado_explotacion::numeric / NULLIF(ventas_netas, 0), 3) AS margen_explotacion
FROM rentabilidad
ORDER BY anio DESC, margen_explotacion DESC;
```

# 4. Análisis de Correlación con Python

``` python
import pandas as pd
import seaborn as sns
import matplotlib.pyplot as plt
import psycopg2
from scipy import stats

# Conexión a la base de datos
conn = psycopg2.connect(
    dbname="agribask",
    user="postgres",
    password="TU_CONTRASEÑA",
    host="localhost",
    port="5432"
)

# Consulta para obtener datos de valor y cantidad
query = """
SELECT * FROM especie_valor_cantidad_long
WHERE cantidad_tm > 0 AND valor_eur > 0
"""
df = pd.read_sql(query, conn)

# 1. Correlación general por especie
corr_results = df.groupby('especie')[['cantidad_tm', 'valor_eur']].corr().unstack().iloc[:,1]
corr_results = corr_results.reset_index()
corr_results.columns = ['Especie', 'Correlación']
print("Correlación entre cantidad y valor por especie:")
print(corr_results.sort_values('Correlación', ascending=False))

# 2. Visualización de correlaciones
plt.figure(figsize=(12, 8))
sns.heatmap(df.pivot_table(index='anio', columns='especie', values='valor_eur').corr(),
            annot=True, cmap='coolwarm', center=0)
plt.title('Matriz de correlación del valor entre especies')
plt.show()

# 3. Análisis detallado por especie (ejemplo con anchoa)
anchoa = df[df['especie'] == 'anchoa']

# Gráfico de dispersión con línea de tendencia
plt.figure(figsize=(10, 6))
sns.regplot(x='cantidad_tm', y='valor_eur', data=anchoa)
plt.title('Relación cantidad-valor: Anchoa')
plt.xlabel('Toneladas métricas')
plt.ylabel('Valor (miles €)')

# Cálculo estadístico
slope, intercept, r_value, p_value, std_err = stats.linregress(
    anchoa['cantidad_tm'], anchoa['valor_eur'])
print(f"\nEstadísticos para anchoa:")
print(f"Coeficiente correlación (r): {r_value:.3f}")
print(f"Valor p: {p_value:.4f}")
print(f"Ecuación: y = {slope:.2f}x + {intercept:.2f}")

# 4. Evolución temporal del valor por tonelada
valor_tm = pd.read_sql("SELECT * FROM valor_por_tonelada", conn)
valor_tm.plot(x='anio', y=['anchoa_valor_tm', 'atun_rojo_valor_tm', 'merluza_valor_tm'],
              figsize=(12, 6), title='Evolución del valor por tonelada métrica')
plt.ylabel('€ por tm')
plt.show()
```
## Resultados Clave de Correlación
Tabla de Correlaciones por Especie
