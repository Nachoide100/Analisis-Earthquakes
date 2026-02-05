# 🌍 Monitorización y Análisis de Actividad Sísmica

![Status](https://img.shields.io/badge/Status-Finalizado-success)
![Tools](https://img.shields.io/badge/Tools-SQL%20%7C%20Python%20%7C%20Power%20BI-blue)
![Date](https://img.shields.io/badge/Fecha-05%20Febrero%202026-lightgrey)

## 📋 Descripción del Proyecto
Este proyecto transforma datos sísmicos brutos en un sistema de **Business Intelligence (BI)** avanzado. El objetivo es proporcionar una herramienta interactiva para la evaluación de riesgos geológicos, identificando patrones de recurrencia y calculando el impacto energético real de los movimientos telúricos.

---

## 🔬 1. Análisis Estadístico y Diagnóstico
Antes del modelado, se realizó un diagnóstico matemático de la variable magnitud (`mag`) para entender la naturaleza del riesgo:

* **Asimetría (Skewness) de 0.90 (Positiva):** Indica una alta frecuencia de sismos leves con una "cola" hacia eventos de gran magnitud. El promedio es un indicador engañoso; el riesgo reside en los extremos.
* **Curtosis (Kurtosis) de 4.38 (Leptocúrtica):** Las colas son más gruesas que en una distribución normal, confirmando que los "outliers" (terremotos extremos) ocurren con más frecuencia de la prevista estadísticamente.

![histograma](https://github.com/Nachoide100/Analisis-Earthquakes/blob/bf56fd8bdf4fceb09549857143bf88ab57bf6eb6/visualizations/histograma_magnitud.png)

---

## 🛠️ 2. Ingeniería de Datos y Métricas Personalizadas
Se desarrollaron nuevas variables para enriquecer el análisis mediante Python y SQL:

### A. Limpieza Geográfica
* **Proceso:** Extracción de la entidad principal (Ej: "CA", "Japan") desde la columna de texto sucio `place`.
* **Resultado:** Agrupación regional precisa para el informe.

```sql
    SELECT place,
        -- Dividimos por la coma y cogemos la parte final (-1)
        -- Usamos TRIM para quitar espacios sobrantes
        TRIM(SPLIT_PART(place, ',', -1)) as Location
    FROM earthquakes
```


### B. Métrica de Energía (Fórmula de Gutenberg-Richter)
La magnitud es logarítmica, por lo que creamos `Energy_Julios` para visualizar el impacto real:
$$E = 10^{(1.5 \times mag + 4.8)}$$
```python
df_final['Energy_Julios'] = 10 ** (1.5 * df_final['mag'] + 4.8)
```
![mag-energy](https://github.com/Nachoide100/Analisis-Earthquakes/blob/bf56fd8bdf4fceb09549857143bf88ab57bf6eb6/visualizations/mag-energy.png)

### C. Métrica de Riesgo (Risk_Score)
Implementamos una función para establecer una relacion real entre magnitud y profundidad:
$$Risk\_Score = \frac{2^{mag}}{depth + 1}$$
```python
df_final['risk_score'] = (2 ** df_final['mag']) / (df_final['depth'] + 1)
```
![mag-depth](https://github.com/Nachoide100/Analisis-Earthquakes/blob/bf56fd8bdf4fceb09549857143bf88ab57bf6eb6/visualizations/map-depth.png)

### D. Análisis de Recurrencia (Enjambres)
Mediante funciones de ventana en SQL (`LAG` con `ORDER BY`), calculamos el tiempo transcurrido entre eventos para detectar enjambres sísmicos.
```sql
SELECT *,
      date_diff('minute', LAG(time) OVER (ORDER BY time), time) as minutos_desde_ultimo_sismo
      FROM df_final
```
![tiempo](https://github.com/Nachoide100/Analisis-Earthquakes/blob/bf56fd8bdf4fceb09549857143bf88ab57bf6eb6/visualizations/tiempo.png)

---

## 📊 3. Estrategia de Visualización (Power BI)
El dashboard se diseñó bajo una filosofía de **Dark Mode (#1A1A1A)** para maximizar el contraste de las alertas y una interfaz tipo "App" con esquinas redondeadas.

### Página 1: Impacto y Riesgo (Resumen Ejecutivo)
* **Mapa de Calor:** Burbujas por magnitud y color según el `Risk_Score`.
* **KPIs Críticos:** Energía total liberada (en trillones), conteo de sismos críticos y magnitud máxima.
* **Top 5 Zonas Críticas:** Gráfico de barras basado en la mediana de riesgo.

![informe](https://github.com/Nachoide100/Analisis-Earthquakes/blob/ebc4564a8682668a012bca38e0f5bfa41ae8ef4c/visualizations/informe.png)

### Página 2: Análisis Físico y Geológico
* **Scatter Plot (Profundidad vs. Magnitud):** Permite descartar sismos fuertes que, por su profundidad, no representan riesgo inmediato.
* **Scatter Plot (Energía vs. Magnitud):** Permite entender la relación real entre el aumento de magnitud y la producción de energía. 
* **Histograma de Tiempos de Espera:** Identificación visual de lo recurrentes que son los sismos. 

Para terner una visión mejorada -> ![Informe PDF](https://github.com/Nachoide100/Analisis-Earthquakes/blob/5dbbfb44b41f2377d7d7580ab04641e92b9ff34e/analisis_terremotos.pdf)
Para acceder al informe dinámico -> ![Enlace](https://drive.google.com/file/d/124XLvt6LqnuFo2LlTl7vs1bFyK8pWcmG/view?usp=drive_link)
---

## ⚙️ 4. Flujo de Trabajo (Workflow)
El sistema utiliza un ciclo de actualización robusto para asegurar la integridad de los datos:

1.  **Procesamiento:** SQL/Python ejecutan los cálculos complejos y la limpieza.
2.  **Exportación:** Generación de un archivo `.csv` optimizado (sobrescritura automática).
3.  **Visualización:** Power BI consume el archivo con un modelo de datos que soporta la actualización de columnas sin errores de formato (manejo de separadores decimales).

---

## 🚀 Conclusión
Este sistema permite pasar de datos aislados a respuestas estratégicas inmediatas, permitiendo identificar no solo dónde ocurrió un sismo, sino **qué tan peligroso fue realmente** y si existe una frecuencia determinada en su aparición. 

---
**Desarrollado por:** José Ignacio Rubio

**Contacto:** [https://www.linkedin.com/in/jos%C3%A9-ignacio-rubio-194471308/]
