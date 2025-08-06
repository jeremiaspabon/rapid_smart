# Anexo B: Código Fuente en Python

Este anexo presenta el código fuente en Python utilizado para el procesamiento y análisis de los datos de la encuesta Rapid SMART COL 2023. El código fue generado a partir de un Jupyter Notebook y adaptado para su inclusión como anexo en esta tesis de grado. Se incluyen las librerías utilizadas, la configuración inicial para la descarga de datos (con credenciales omitidas por seguridad), la importación de tablas nutricionales de la OMS, y el procesamiento de los diferentes dataframes de la encuesta.

```python



# Importación de Librerías
import pandas as pd
import seaborn as sns
from matplotlib import pyplot as plt
import numpy as np
import collections
from collections import Counter
import plotly.express as px
import os
from plotly.subplots import make_subplots
import plotly.graph_objects as go
import matplotlib.pyplot as plt
import seaborn as sns
import requests
import base64
from io import BytesIO

# Configuración [Datos no públicos]
usuario = ""
repo = ""
ruta_archivo = ""
token = ""

# Descargar usando la API
url = f"https://api.github.com/repos/{usuario}/{repo}/contents/{ruta_archivo}"
headers = {"Authorization": f"token {token}"}
response = requests.get(url, headers=headers)
content = response.json()["content"]

# Decodificar contenido (está en base64)
import base64
decoded = base64.b64decode(content)

# Guardar como archivo Excel
with open("SMART_COL_2023.xlsx", "wb") as f:
    f.write(decoded)

# Cargar las hojas
all_sheets = pd.read_excel("SMART_COL_2023.xlsx", sheet_name=None)

# Importación de tablas nutricionales WHZ de la OMS
# Fuente: https://www.who.int/tools/child-growth-standards/standards/weight-for-length-height

archivos = {
    "data/wfl_boys_0-to-2-years_zscores.xlsx": "lms_boys_0_2",
    "data/wfl_girls_0-to-2-years_zscores.xlsx": "lms_girls_0_2",
    "data/wfh_boys_2-to-5-years_zscores.xlsx": "lms_boys_2_5",
    "data/wfh_girls_2-to-5-years_zscores.xlsx": "lms_girls_2_5"
}
# Diccionario para almacenar los DataFrames de cada archivo
dfs = {}

for ruta_archivo, nombre_df in archivos.items():
    try:
        # Descargar el archivo desde GitHub
        url = f"https://api.github.com/repos/{usuario}/{repo}/contents/{ruta_archivo}"
        headers = {"Authorization": f"token {token}"}
        response = requests.get(url, headers=headers)
        response.raise_for_status()

        # Decodificar y leer el Excel
        content = base64.b64decode(response.json()["content"])
        df = pd.read_excel(BytesIO(content))

        # Asignar el DataFrame a una variable global con el nombre deseado
        globals()[nombre_df] = df
        dfs[nombre_df] = df  # Opcional: guardar en diccionario

        print(f"✅ Archivo \'{ruta_archivo}\' cargado como \'{nombre_df}\'. Shape: {df.shape}")
    except Exception as e:
        print(f"❌ Error al cargar \'{ruta_archivo}\': {str(e)}")

df_hogar = all_sheets["SMART HOGAR"]
df_fallecimientos = all_sheets["FALLECIMIENTOS"]
df_plw = all_sheets["GEST-LACT"]
df_miembros = all_sheets["MIEMBROS"]
df_ninos = all_sheets["NIÑOS"]
df_no_miembros = all_sheets["NO MIEMBROS"]

# Lista de DataFrames y sus nombres de hoja
dataframes = {
    "df_hogar": "SMART HOGAR",
    "df_fallecimientos": "FALLECIMIENTOS",
    "df_plw": "GEST-LACT",
    "df_miembros": "MIEMBROS",
    "df_ninos": "NIÑOS",
    "df_no_miembros": "NO MIEMBROS"
}

# Crear la lista de resumen
summary_data = []
for df_name, sheet_name in dataframes.items():
    df = all_sheets[sheet_name]
    summary_data.append([df_name, sheet_name, df.shape[1], df.shape[0]])

# Crear el DataFrame resumen
summary_df = pd.DataFrame(summary_data, columns=["DF", "Descripción", "Número de Columnas", "Número de registros"])

# Mostrar la tabla resumen
display(summary_df)

display(df_hogar.head(5))

# Crear un diccionario que mapea los nombres de las variables de hogar
dicc_hogar = {col: f"Hog_{i+1}" for i, col in enumerate(df_hogar.columns)}

# Renombrar las columnas en el DataFrame
df_hogar.rename(columns=dicc_hogar, inplace=True)

# Convertir el diccionario en un DataFrame para facilitar su consulta o guardado
df_dicc_hogar = pd.DataFrame(list(dicc_hogar.items()), columns=["Columna_Original", "Columna_Nueva"])

# Elimimos variables que no aportan al análsis de los datos
# Lista de columnas a eliminar
columnas_a_eliminar = ["Hog_1", "Hog_2", "Hog_3", "Hog_4", "Hog_8", "Hog_9", "Hog_10",
                        "Hog_11", "Hog_12", "Hog_13", "Hog_14", "Hog_15", "Hog_17", "Hog_19",
                        "Hog_21", "Hog_22", "Hog_24", "Hog_25", "Hog_26", "Hog_27",
                        "Hog_28", "Hog_29", "Hog_31", "Hog_32", "Hog_33", "Hog_34",
                        "Hog_35", "Hog_36", "Hog_37", "Hog_38", "Hog_39", "Hog_40",
                        "Hog_41", "Hog_42", "Hog_43", "Hog_44", "Hog_45", "Hog_47",
                        "Hog_48", "Hog_49", "Hog_50", "Hog_51", "Hog_52", "Hog_53",
                        "Hog_54", "Hog_55", "Hog_56", "Hog_57", "Hog_58", "Hog_60",
                        "Hog_61", "Hog_62", "Hog_82", "Hog_83", "Hog_85", "Hog_86",
                        "Hog_87", "Hog_88", "Hog_89", "Hog_90", "Hog_91", "Hog_93",
                        "Hog_95", "Hog_96", "Hog_97", "Hog_98", "Hog_99", "Hog_100",
                        "Hog_102", "Hog_103", "Hog_104", "Hog_105", "Hog_106",
                        "Hog_107", "Hog_108", "Hog_109", "Hog_111", "Hog_112", "Hog_113"]

# Filtrar solo las columnas que existen en df_hogar
columnas_existentes = [col for col in columnas_a_eliminar if col in df_hogar.columns]

# Eliminar solo las columnas existentes
df_hogar.drop(columns=columnas_existentes, inplace=True)


# Filtramos los rigristros que fueron marcado en como encuesta completada 100% [variable 101]
df_hogar = df_hogar[df_hogar["Hog_101"] == "Encuestado 100%"]

#Eliminamos registros duplicados si existen
df_hogar = df_hogar.drop_duplicates()

# Diccionario con el mapeo de nombres específicos
mapeo_nombres = {
    "Hog_5": "sector", "Hog_6": "municipio", "Hog_7": "zona",
    "Hog_16": "numero_familia", "Hog_18": "numero_ninos", "Hog_20": "numero_mujeres",
    "Hog_23": "numero_no_miembros", "Hog_30": "mumero_fallecidos", "Hog_46": "indicador_hdds",
    "Hog_59": "indicador_rcsi", "Hog_63": "lcsi_vende_bienes", "Hog_64": "lcsi_gasta_ahorros",
    "Hog_65": "lcsi_vende_animales", "Hog_66": "lcsi_come_fuera", "Hog_67": "lcsi_alimentos_prestados",
    "Hog_68": "lcsi_dinero_prestado", "Hog_69": "lcsi_cambio_escuela", "Hog_70": "lcsi_vende_activos",
    "Hog_71": "lcsi_retiro_escuela", "Hog_72": "lcsi_redujo_gastos", "Hog_73": "lcsi_cultivos_inmaduros",
    "Hog_74": "lcsi_consume_reservas", "Hog_75": "lcsi_gastos_fertilizantes", "Hog_76": "lcsi_patrimonio",
    "Hog_77": "lcsi_mendigar", "Hog_78": "lcsi_otras_actividades", "Hog_79": "lcsi_ganado",
    "Hog_80": "lcsi_emigro_hogar", "Hog_81": "wash_acceso_agua", "Hog_84": "wash_no_tratamiento",
    "Hog_92": "wash_acceso_higiene", "Hog_94": "wash_no_practicas_higiene", "Hog_101": "estado_encuesta",
    "Hog_110": "id_hogar", "Hog_114": "index_hogar"
}

# Renombrar las columnas en df_hogar
df_hogar.rename(columns=mapeo_nombres, inplace=True)


import pandas as pd

# Definir la clasificación de cada estrategia según su nivel de gravedad
clasificacion_lcsi = {
    "Estrategias de Estrés": [
        "lcsi_gasta_ahorros",
        "lcsi_alimentos_prestados",
        "lcsi_dinero_prestado",
        "lcsi_redujo_gastos",
        "lcsi_consume_reservas",
        "lcsi_gastos_fertilizantes",
    ],
    "Estrategias de Crisis": [
        "lcsi_vende_bienes",
        "lcsi_vende_animales",
        "lcsi_cultivos_inmaduros",
        "lcsi_patrimonio",
        "lcsi_ganado",
    ],
    "Estrategias de Emergencia": [
        "lcsi_retiro_escuela",
        "lcsi_cambio_escuela",
        "lcsi_mendigar",
        "lcsi_vende_activos",
        "lcsi_emigro_hogar",
        "lcsi_otras_actividades",
    ],
}

# Definir el orden de gravedad (de menor a mayor)
niveles_gravedad = {
    "Ninguna": 0,
    "Estrategias de Estrés": 1,
    "Estrategias de Crisis": 2,
    "Estrategias de Emergencia": 3,
}

# Crear una función para determinar el nivel máximo de gravedad por fila
def clasificar_lcsi(fila):
    nivel_maximo = "Ninguna"
    max_puntaje = 0
    for categoria, variables in clasificacion_lcsi.items():
        # Verificar si alguna variable de la categoría tiene "Sí"
        if any(fila[var] == "Sí" for var in variables if var in fila):
            puntaje = niveles_gravedad[categoria]
            if puntaje > max_puntaje:
                max_puntaje = puntaje
                nivel_maximo = categoria
    return nivel_maximo

# Aplicar la función al DataFrame para crear la columna "LCSI"
df_hogar["LCSI"] = df_hogar.apply(clasificar_lcsi, axis=1)

# Lista de columnas a eliminar
columnas_a_eliminar = [
    "lcsi_gasta_ahorros",
    "lcsi_vende_animales",
    "lcsi_come_fuera",
    "lcsi_alimentos_prestados",
    "lcsi_dinero_prestado",
    "lcsi_cambio_escuela",
    "lcsi_vende_activos",
    "lcsi_retiro_escuela",
    "lcsi_redujo_gastos",
    "lcsi_cultivos_inmaduros",
    "lcsi_consume_reservas",
    "lcsi_gastos_fertilizantes",
    "lcsi_patrimonio",
    "lcsi_mendigar",
    "lcsi_otras_actividades",
    "lcsi_ganado",
    "lcsi_emigro_hogar",
    "lcsi_vende_bienes",
]

# Eliminar las columnas del DataFrame
df_hogar = df_hogar.drop(columns=columnas_a_eliminar)

# Definir el diccionario de mapeo
mapeo_agua = {
    "Acueducto (servicio público)": "Acueducto",
    "Fuente de agua subterránea (pozo con extracción manual, pozo con bomba, puntillo)": "Fuente de agua subterránea",
    "Fuente de agua superficial (río, quebrada, nacimiento)": "Fuente de agua superficial",
}

# Aplicar el mapeo a la columna
df_hogar["wash_acceso_agua"] = df_hogar["wash_acceso_agua"].replace(mapeo_agua)

# Renombrar las columnas
df_hogar = df_hogar.rename(columns={
    "wash_no_practicas_higiene": "wash_practicas_higiene",
    "wash_no_tratamiento": "wash_tratamiento"
})

# Cambiar los valores: 0 → "Sí", 1 → "No"
df_hogar["wash_practicas_higiene"] = df_hogar["wash_practicas_higiene"].replace({0: "Sí", 1: "No"})
df_hogar["wash_tratamiento"] = df_hogar["wash_tratamiento"].replace({0: "Sí", 1: "No"})

# Verificar las columnas restantes
print(df_hogar.columns)

# Obtener valores originales de df_hogar desde summary_df
original_cols = summary_df.loc[summary_df["DF"] == "df_hogar", "Número de Columnas"].values[0]
original_rows = summary_df.loc[summary_df["DF"] == "df_hogar", "Número de registros"].values[0]

# Crear DataFrame de comparación
comparison_df = pd.DataFrame({
    "Métrica": ["Número de columnas", "Número de registros"],
    "Antes": [original_cols, original_rows],
    "Después": [df_hogar.shape[1], df_hogar.shape[0]]
})

# Mostrar la tabla
display(comparison_df)

# Graficar la comparación
fig, ax = plt.subplots(figsize=(6, 4))
comparison_df.set_index("Métrica").plot(kind="bar", ax=ax, color=["skyblue", "salmon"])
plt.title("Comparación antes y después de la limpieza de df_hogar")
plt.ylabel("Cantidad")
plt.xticks(rotation=0)
plt.legend(title="Estado")
plt.grid(axis="y", linestyle="--", alpha=0.7)

# Mostrar el gráfico
plt.show()

display(df_plw.head(5))

# Crear un diccionario que mapea los nombres de las variables de mujeres gestantes y/o lactantes
dicc_plw = {col: f"Plw_{i+1}" for i, col in enumerate(df_plw.columns)}

# Renombrar las columnas en el DataFrame
df_plw.rename(columns=dicc_plw, inplace=True)

# Convertir el diccionario en un DataFrame para facilitar su consulta o guardado
df_dicc_plw = pd.DataFrame(list(dicc_plw.items()), columns=["Columna_Original", "Columna_Nueva"])

# Elimimos variables que no aportan al análsis de los datos
# Lista de columnas a eliminar
columnas_a_eliminar = ["Plw_1", "Plw_2", "Plw_3", "Plw_6", "Plw_10", "Plw_11",
                       "Plw_12", "Plw_13", "Plw_14", "Plw_15", "Plw_16", "Plw_17",
                       "Plw_18", "Plw_19", "Plw_20", "Plw_21", "Plw_22", "Plw_23",
                       "Plw_24", "Plw_25", "Plw_27", "Plw_30", "Plw_33", "Plw_34", "Plw_35","Plw_36"]

# Filtrar solo las columnas que existen en df_hogar
columnas_existentes = [col for col in columnas_a_eliminar if col in df_plw.columns]

# Eliminar solo las columnas existentes
df_plw.drop(columns=columnas_existentes, inplace=True)

display(df_plw.head(5))

# Diccionario con el mapeo de nombres específicos
mapeo_nombres = {"Plw_4" : "edad_anio", "Plw_5" : "sexo", "Plw_7" : "estado",
                 "Plw_8" : "controles", "Plw_9" : "suplemento",
                 "Plw_26" : "indicador_mddw", "Plw_28" : "pb_plw",
                 "Plw_29" : "index_plw", "Plw_31" : "index_hogar", "Plw_32" : "id_hogar"
}

# Renombrar las columnas en df_hogar
df_plw.rename(columns=mapeo_nombres, inplace=True)


display(df_plw.head(5))

#Se filtan los registros de mujeres gestantes/lactante de acuerdo con los hogares finales
df_plw = df_plw.merge(df_hogar[["index_hogar"]], on="index_hogar", how="inner")

# Filtrar por sexo igual a "Mujer"
df_plw = df_plw[df_plw["sexo"] == "Mujer"]

# Eliminar filas donde la columna "estado" tenga valores vacíos
df_plw = df_plw[df_plw["estado"].notna()]


# Obtener valores originales de df_plw desde summary_df
original_cols = summary_df.loc[summary_df["DF"] == "df_plw", "Número de Columnas"].values[0]
original_rows = summary_df.loc[summary_df["DF"] == "df_plw", "Número de registros"].values[0]

# Crear DataFrame de comparación
comparison_df = pd.DataFrame({
    "Métrica": ["Número de columnas", "Número de registros"],
    "Antes": [original_cols, original_rows],
    "Después": [df_plw.shape[1], df_plw.shape[0]]
})

# Mostrar la tabla
display(comparison_df)

# Graficar la comparación
fig, ax = plt.subplots(figsize=(6, 4))
comparison_df.set_index("Métrica").plot(kind="bar", ax=ax, color=["skyblue", "salmon"])
plt.title("Comparación antes y después de la limpieza de df_hogar")
plt.ylabel("Cantidad")
plt.xticks(rotation=0)
plt.legend(title="Estado")
plt.grid(axis="y", linestyle="--", alpha=0.7)

# Mostrar el gráfico
plt.show()


# Calcular el promedio de cada columna por id_hogar
df_plw_avg = df_plw.groupby("id_hogar")[["indicador_mddw", "pb_plw"]].mean().reset_index()

# Unir los promedios calculados con df_hogar en base a id_hogar
df_hogar = pd.merge(df_hogar, df_plw_avg, on="id_hogar", how="left")

display(df_hogar.head(5))

# Crear un diccionario que mapea los nombres de las variables de niños y niñas < 5 años
dicc_ninos = {col: f"Nn_{i+1}" for i, col in enumerate(df_ninos.columns)}

# Renombrar las columnas en el DataFrame
df_ninos.rename(columns=dicc_ninos, inplace=True)

# Convertir el diccionario en un DataFrame para facilitar su consulta o guardado
df_dicc_ninos = pd.DataFrame(list(dicc_ninos.items()), columns=["Columna_Original", "Columna_Nueva"])

# Elimimos variables que no aportan al análsis de los datos
# Lista de columnas a eliminar
columnas_a_eliminar = ["Nn_1", "Nn_2", "Nn_3", "Nn_6", "Nn_7", "Nn_10", "Nn_12",
                       "Nn_13", "Nn_14", "Nn_15", "Nn_17", "Nn_19", "Nn_21",
                       "Nn_22", "Nn_23", "Nn_26", "Nn_27", "Nn_42", "Nn_43",
                       "Nn_44", "Nn_48", "Nn_49", "Nn_50", "Nn_51", "Nn_52", "Nn_53",
                       "Nn_54", "Nn_55", "Nn_56", "Nn_57", "Nn_58", "Nn_59",
                       "Nn_60", "Nn_61", "Nn_62", "Nn_63", "Nn_64", "Nn_65",
                       "Nn_66", "Nn_67", "Nn_68", "Nn_69", "Nn_70", "Nn_71",
                       "Nn_72", "Nn_73", "Nn_74", "Nn_75", "Nn_76","Nn_77", "Nn_78",
                       "Nn_79", "Nn_80", "Nn_81", "Nn_82", "Nn_83", "Nn_84",
                       "Nn_85", "Nn_86", "Nn_87", "Nn_88", "Nn_89", "Nn_90",
                       "Nn_91", "Nn_92", "Nn_93", "Nn_94", "Nn_95", "Nn_97",
                       "Nn_99", "Nn_100", "Nn_101", "Nn_102", "Nn_103", "Nn_109",
                       "Nn_110", "Nn_111", "Nn_118", "Nn_155", "Nn_157", "Nn_160",
                       "Nn_161", "Nn_162",  "Nn_28", "Nn_29", "Nn_30", "Nn_31",
                       "Nn_32", "Nn_33", "Nn_34", "Nn_35", "Nn_36", "Nn_37",
                       "Nn_38", "Nn_39", "Nn_41", "Nn_45", "Nn_46", "Nn_47", "Nn_48",
                       "Nn_105", "Nn_132", "Nn_139", "Nn_145", "Nn_150", "Nn_151",
                       "Nn_152", "Nn_153", "Nn_154", "Nn_122", "Nn_144", "Nn_104"]

# Filtrar solo las columnas que existen en df_ninos
columnas_existentes = [col for col in columnas_a_eliminar if col in df_ninos.columns]

# Eliminar solo las columnas existentes
df_ninos.drop(columns=columnas_existentes, inplace=True)

# Diccionario con el mapeo de nombres específicos
mapeo_nombres = {"Nn_4" : "edad_anio", "Nn_5" : "sexo", "Nn_8" : "fecha_nacimiento",
                 "Nn_9" : "fecha_nacimiento2", "Nn_11" : "edad_meses",
                 "Nn_16" : "nino_presente", "Nn_18" : "peso",
                 "Nn_20" : "altura", "Nn_24" : "perimetro_braquial",
                 "Nn_25" : "enema", "Nn_" # truncated for brevity


