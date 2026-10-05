# Inferencia Causal y Uplift Modeling en una Campaña de Email Marketing

Análisis del efecto causal de una campaña de email marketing sobre conversión y gasto, usando el
dataset experimental de Kevin Hillstrom (MineThatData, 2008). El proyecto además de medir "si
funciona en promedio" (ATE) estima a qué clientes conviene dirigir el email (CATE / uplift
modeling), validando estadísticamente en cada paso.

## Pregunta de negocio

¿Genera la campaña de email un efecto causal real sobre la conversión y el gasto? Y si es así,
¿a qué clientes conviene priorizar para maximizar el retorno y evitar el desgaste de contactar a
quien no lo necesita?

## Sobre el dataset

Es un experimento aleatorizado real (RCT), no observacional: ~64,000 clientes asignados
aleatoriamente a recibir o no una campaña de email permitiendo estimar efectos causales sin
necesidad de técnicas de corrección de sesgo por confusión verificando primeramente si
la aleatorización efectivamente produjo grupos balanceados, siendo este el primer paso del análisis.

## Metodología

1. **Carga y exploración de datos.**
2. **Chequeo de balance de covariables** (tests de Chi-cuadrada y t-test de Welch) se valida
   que el diseño experimental funcionó antes de interpretar cualquier diferencia como causal.
3. **Estimación del ATE** sobre `visit`, `conversion` y `spend`, con **corrección de Bonferroni**
   por comparaciones múltiples.
4. **Estimación de CATE con T-learner (XGBoost) y cross-fitting (5-fold):** las predicciones de
   uplift para cada cliente provienen de un modelo que nunca vio a ese cliente durante el
   entrenamiento (out-of-fold), lo que reduce el riesgo de sobreajuste optimista frente a un split
   único de validación.
5. **Evaluación del modelo de targeting** con curva Qini y uplift por decil, **con intervalos de
   confianza bootstrap** para distinguir qué diferencias entre deciles son señal real y cuáles
   son ruido muestral.
6. **Conclusiones y recomendación de negocio**, con limitaciones documentadas explícitamente.

## Hallazgos clave

- El email tiene un efecto causal positivo y estadísticamente significativo sobre conversión y
  gasto, incluso tras la corrección de Bonferroni. La conversión se incrementó en aproximadamente
  un 86.53% relativo frente al grupo control. El gasto promedio (Spend) aumentó en aproximadamente
  $0.60 dólares por usuario contactado.
- El modelo de targeting (CATE) discrimina de forma confiable en el decil superior (los IC no
  cruzan cero), pero el ranking en los deciles intermedios no es estadísticamente distinguible
  entre sí una diferencia que solo es visible al agregar intervalos de confianza bootstrap, no
  con el punto estimado solo.
- Recomendación: usar el modelo para **priorizar** a quién contactar (decil superior), no para
  **excluir** con base en los deciles bajos, donde la señal es demasiado ruidosa para sustentar esa
  decisión.

## Limitaciones y siguientes pasos

- Este análisis responde a "¿contactar o no?", no a "¿con qué campaña?", porque los brazos Mens/Womens
  se unificaron en un solo tratamiento. El siguiente paso natural es extender este T-learner a un esquema
  multi-brazo (3 modelos independientes: No E-Mail / Mens / Womens) para poder recomendar, además de a quién contactar, con qué tipo de publicidad.
- El T-learner no usa hiperparámetros optimizados vía validación cruzada (se usaron valores
  conservadores fijos, simétricos entre ambos modelos). Una extensión futura sería comparar contra
  un X-learner o un causal forest (`econml`/`causalml`), menos sensibles al desbalance de tamaño
  entre grupos de tratamiento y control.

## Estructura del repositorio

```
.
├── causal_analysis_hillstrom.html    # Notebook en formato html
├── causal_analysis_hillstrom.ipynb   # Notebook principal
├── requirements.txt                  # Dependencias
└── README.md
```

## Cómo correr este proyecto

```bash
python -m venv venv
source venv/bin/activate          # En Windows: venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook causal_analysis_hillstrom.ipynb
```

El dataset se descarga automáticamente dentro del notebook desde la fuente original
(MineThatData). No requiere configuración adicional ni credenciales.

## Stack técnico

`Python` · `pandas` / `numpy` · `scipy` / `statsmodels` (tests estadísticos) · `scikit-learn`
(cross-fitting) · `XGBoost` (modelos base del T-learner) · `scikit-uplift` (métricas de uplift:
curva Qini) · `matplotlib` / `seaborn`

## Fuente de datos

Kevin Hillstrom, *MineThatData E-Mail Analytics And Data Mining Challenge* (2008).
[minethatdata.com](http://www.minethatdata.com/Kevin_Hillstrom_MineThatData_E-MailAnalytics_DataMiningChallenge_2008.03.20.csv)
