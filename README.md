---
jupyter:
  colab:
  kernelspec:
    display_name: Python 3
    name: python3
  language_info:
    name: python
  nbformat: 4
  nbformat_minor: 0
---

::: {.cell .markdown id="689b99c6"}
``` markdown
# Sistema de Recomendación de Películas Top 500

Este notebook contiene un sistema interactivo para explorar y recibir recomendaciones de películas basadas en el "Top 500" de un dataset específico. Incluye visualizaciones de datos, funciones de búsqueda por país y género, y un quiz personalizado para generar recomendaciones.

## 1. Configuración Inicial

### Instalación de Plotly
Se instala la librería Plotly, esencial para la visualización de mapas coropléticos y otros gráficos interactivos.

```python
!pip install plotly
```

### Importación de Librerías

Se importan las librerías necesarias para el análisis y visualización de datos:

- `pandas` para manipulación de datos.
- `plotly.express` para gráficos interactivos.
- `matplotlib.pyplot` y `seaborn` para gráficos estáticos.
- `IPython.display` para mostrar HTML en el notebook.

``` python
import pandas as pd
import plotly.express as px
import matplotlib.pyplot as plt
import seaborn as sns
from IPython.display import HTML, display
```

### Carga de Datos

Se carga el dataset `top500.csv` en un DataFrame de pandas llamado `df_top500`.

``` python
df_top500 = pd.read_csv("/content/top500.csv")
```

## 2. Exploración y Visualización de Datos

### Cantidad de Películas por País (Mapa Coroplético)

Esta sección calcula el número de películas por país en el dataset y las visualiza en un mapa coroplético interactivo. Esto permite identificar rápidamente las regiones con mayor producción de películas en el Top 500.

``` python
# Define el número de películas por país y los pone en columnas
country_count = df_top500["country"].value_counts().reset_index()
country_count.columns = ["country", "Cantidad de películas"]

# Ilustra el mapa
fig = px.choropleth(
    country_count,
    locations="country",
    locationmode="country names",
    color="Cantidad de películas",
    hover_name="country",
    hover_data=["Cantidad de películas"],
    color_continuous_scale="Greens",
    title="Cantidad de películas del Top 500 por país"
)
fig.update_layout(
    width=1000,
    height=600
)
fig.show()
```

### Top 10 Directores con Más Películas

Se identifican y grafican los 10 directores con la mayor cantidad de películas en el Top 500. Esto proporciona una visión sobre los cineastas más influyentes o prolíficos dentro de esta selección.

``` python
# Lista los directores a partir de la base de datos
director_cols = [col for col in df_top500.columns if 'director/' in col]
all_directors = df_top500[director_cols].stack().dropna().tolist()

# Se define la tabla del top 10 directores con más películas en el top
director_counts = pd.Series(all_directors).value_counts().reset_index()
director_counts.columns = ['Director', 'Cantidad de Películas']
display(director_counts.head(10))

# Crea gráfico de barras según cuál director tiene más películas en el top
fig = plt.figure(figsize=(12, 6))
sns.barplot(x='Cantidad de Películas', y='Director', data=director_counts.head(10), palette='viridis')
plt.title('Top 10 Directores con Más Películas en el Top 500')
plt.xlabel('Número de Películas')
plt.ylabel('Director')
plt.tight_layout()
plt.show()
```

### Top 10 Géneros Más Comunes

Similar a los directores, esta sección analiza la distribución de géneros y muestra un gráfico de barras con los 10 géneros más frecuentes en el Top 500.

``` python
# Lista los géneros a partir de la base de datos
if 'genres_combined' not in df_top500.columns:
    genre_cols = [col for col in df_top500.columns if 'genres/' in col]
    df_top500['genres_combined'] = df_top500[genre_cols].apply(lambda row: ', '.join(row.dropna().astype(str)), axis=1)

# Dataframe de todos los directores definidso en una variable
all_genres = df_top500['genres_combined'].str.split(', ').explode().str.strip().tolist()
all_genres = [genre for genre in all_genres if genre]

# Se define la tabla del top 10 directores con más películas en el top
genre_counts = pd.Series(all_genres).value_counts().reset_index()
genre_counts.columns = ['Género', 'Cantidad de Películas']
display(genre_counts.head(10))

# Crea gráfico de barras según cuál director tiene más películas en el top
fig = plt.figure(figsize=(12, 6))
sns.barplot(x='Cantidad de Películas', y='Género', data=genre_counts.head(10), palette='magma')
plt.title('Top 10 Géneros Más Comunes en el Top 500')
plt.xlabel('Número de Películas')
plt.ylabel('Género')
plt.tight_layout()
plt.show()
```

## 3. Funciones de Búsqueda de Películas

### Búsqueda de Películas por País

Permite al usuario ingresar un nombre de país y muestra todas las películas de ese país presentes en el Top 500, ordenadas por calificación promedio. Incluye la visualización de pósters si están disponibles.

``` python
# Importa archivos para mostrarlos
from IPython.display import HTML, display

# Equivale el resultado al dataframe que almacena los países de las películas
country = input("Escribe el nombre del país para ver sus películas: ")

# Equivale el resultado al dataframe que almacena los países de las películas
resultado = df_top500[df_top500["country"] == country]

# Visualiza todas las películas con imágenes
if resultado.empty:
    print(f"No se encontraron películas para {country} en el Top 500.")
else:
    print(f"Películas de {country} (Top 500):\n")
    for index, row in resultado.sort_values(by='averageRating', ascending=False).iterrows():
        title = row["title"]
        year = row["year"]
        director = row["director/0"] if "director/0" in row and pd.notna(row["director/0"]) else "N/A"
        avg_rating = row["averageRating"]
        poster_url = row["posterUrl"] if "posterUrl" in row and pd.notna(row["posterUrl"]) else ""

        # Coloca las imágenes
        movie_info_html = f""
        if poster_url:
            movie_info_html += f"<div style='display:flex; align-items:center; margin-bottom: 20px;'>"
            movie_info_html += f"<img src='{poster_url}' style='width:100px; height:auto; margin-right:15px;'>"
            movie_info_html += f"<div>"
        else:
            movie_info_html += f"<div>"

        movie_info_html += f"  <strong>Título:</strong> {title}<br>"
        movie_info_html += f"  <strong>Año:</strong> {year}<br>"
        movie_info_html += f"  <strong>Director:</strong> {director}<br>"
        movie_info_html += f"  <strong>Calificación Promedio (Ranking):</strong> {avg_rating}<br>"
        movie_info_html += f"</div>"

        if poster_url:
            movie_info_html += f"</div>"

        display(HTML(movie_info_html))
        print("-----------------------------------------")
```

### Búsqueda de Películas por Género

Esta función permite al usuario buscar películas por un género específico. Muestra las 10 películas mejor calificadas de ese género, incluyendo sus pósters y detalles.

``` python
# Importa archivos para mostrarlos
import pandas as pd
from IPython.display import HTML, display

# Equivale el resultado al dataframe que almacena los países de las géneros
genre_cols = [col for col in df_top500.columns if 'genres/' in col]
df_top500['genres_combined'] = df_top500[genre_cols].apply(lambda row: ', '.join(row.dropna().astype(str)), axis=1)

# Coloca el género
print("Géneros admitidos: Adventure, Drama, Fantasy, Music, Romance, Thriller, War, Western")
genre_input = input("Escribe el nombre del género para ver sus películas: ")

# Determina si el input está en la lista hecha de generos
resultado_genero = df_top500[df_top500['genres_combined'].str.contains(genre_input, case=False, na=False)]

# Visualiza todas las películas con imágenes
if resultado_genero.empty:
    print(f"No se encontraron películas para el género '{genre_input}' en el Top 500.")
else:
    print(f"Películas del género '{genre_input}' (Top 500):\n")
    for index, row in resultado_genero.sort_values(by='averageRating', ascending=False).head(10).iterrows():
        title = row["title"]
        year = row["year"]
        director = row["director/0"] if "director/0" in row and pd.notna(row["director/0"]) else "N/A"
        avg_rating = row["averageRating"]
        poster_url = row["posterUrl"] if "posterUrl" in row and pd.notna(row["posterUrl"]) else ""

        # Se visualizan las imágenes
        movie_info_html = ""
        if poster_url:
            movie_info_html += f"<div style='display:flex; align-items:center; margin-bottom: 20px;'>"
            movie_info_html += f"<img src='{poster_url}' style='width:100px; height:auto; margin-right:15px;'>"
            movie_info_html += f"<div>"
        else:
            movie_info_html += f"<div>"

        movie_info_html += f"  <strong>Título:</strong> {title}<br>"
        movie_info_html += f"  <strong>Año:</strong> {year}<br>"
        movie_info_html += f"  <strong>Director:</strong> {director}<br>"
        movie_info_html += f"  <strong>Calificación Promedio (Ranking):</strong> {avg_rating}<br>"
        movie_info_html += f"</div>"

        if poster_url:
            movie_info_html += f"</div>"

        display(HTML(movie_info_html))
        print("-----------------------------------------")
```

## 4. Sistema de Recomendación Interactivo

Este es el corazón del notebook, un sistema de recomendación que guía al usuario a través de un quiz para entender sus preferencias y luego filtra y puntúa las películas para ofrecer las mejores sugerencias.

### `cargar_y_limpiar_datos(df_source)`

Esta función adapta el DataFrame original (`df_top500`) para el sistema de recomendación. Renombra columnas a un formato más amigable (ej., \'título\', \'año\', \'país o región\'), maneja datos faltantes y normaliza strings, especialmente para géneros y directores.

### `ejecutar_quiz()`

Despliega un cuestionario interactivo en la consola, solicitando al usuario sus preferencias sobre género, duración, época, calificación, región, idioma, experiencia emocional, géneros a evitar y directores favoritos. Recoge todas las respuestas y las devuelve en un diccionario.

### `filtrar_y_puntuar(df, respuestas)`

Esta función clave aplica los criterios del usuario para filtrar y asignar una puntuación a cada película. Utiliza:

- **Filtrado Duro**: Elimina películas de géneros que el usuario desea evitar.
- **Scoring Inteligente**: Asigna puntos a las películas basándose en la coincidencia con el género preferido, la experiencia emocional (inferida de la sinopsis), y la calificación de Letterboxd.
- **Filtros Adaptativos/Flexibles**: Ajusta la puntuación según la duración, época, rating mínimo, región e idioma preferidos. También da un bonus si el director favorito del usuario coincide.

Finalmente, ordena las películas por su puntuación de afinidad (y el rating de Letterboxd para desempates) y devuelve un DataFrame con las películas candidatas.

### `mostrar_recomendaciones(df_resultados, respuestas)`

Esta función toma el DataFrame de películas puntuadas y las respuestas del usuario para presentar las recomendaciones en un formato estructurado. Para cada película recomendada, muestra:

- Título, Año, Director, Calificación, Duración, País, Géneros.
- Una explicación dinámica (`Por qué te lo recomiendo`) que justifica la recomendación basándose en las preferencias del usuario.
- El póster de la película (si está disponible) para una mejor visualización.

### Ejecución Principal

La última celda coordina la ejecución de todo el sistema:

1.  Carga `df_top500`.
2.  Limpia y prepara los datos usando `cargar_y_limpiar_datos`.
3.  Ejecuta el quiz con `ejecutar_quiz`.
4.  Filtra y puntúa las películas con `filtrar_y_puntuar`.
5.  Muestra las recomendaciones finales con `mostrar_recomendaciones`.

Este notebook ofrece una experiencia completa desde la carga de datos hasta la generación de recomendaciones personalizadas, utilizando visualizaciones para una mejor comprensión de la data y una interfaz interactiva para el usuario.
:::
