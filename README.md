# EP2: Clasificación de razas de gatos con una CNN

## Objetivo

Construir y evaluar una red neuronal convolucional (CNN) para clasificar imágenes en cinco razas de gatos. El trabajo compara el desempeño en dos grupos de dificultad visual —PARECIDAS y DISTINTAS— y contrasta la CNN seleccionada con el perceptrón multicapa (MLP) final de la EP1. La aplicación se plantea como apoyo al registro veterinario: la predicción debe ser confirmada por una persona y no se usa como decisión autónoma.

## Datos

Se utiliza Oxford-IIIT Pet Dataset y sus anotaciones oficiales. El notebook selecciona cinco razas para cada subconjunto:

- PARECIDAS: Birman, Ragdoll, Siamese, Persian y Maine_Coon.
- DISTINTAS: Bombay, Sphynx, Russian_Blue, Bengal y Abyssinian.

Se toman 184 imágenes por raza con semilla 2604. La división estratificada es 70% entrenamiento, 15% validación y 15% prueba: 644 imágenes de entrenamiento, 138 de validación y 138 de prueba por subconjunto. La lista exacta de nombres, razas, etiquetas y particiones se guarda en data/particiones.csv. Las imágenes originales no se incluyen; el notebook las descarga del sitio oficial.

## Contenido del repositorio

- README.md: descripción del proyecto, ejecución y resultados.
- notebooks/EP2_CNN_Razas_Gatos_interpretado_para_entrega.ipynb: notebook con código, salidas e interpretaciones.
- data/particiones.csv: asignación reproducible de imágenes a subconjuntos y particiones.
- images/: figuras EDA, curvas de entrenamiento, matrices de confusión, comparaciones, mapas de características y Grad-CAM.
- models/: modelos guardados de CNN y MLP para la semilla 101.
- resultados/: resultados por semilla y resúmenes CSV de selección, evaluación, comparación, KPI complementario y ablaciones.

## Cómo ejecutar

El notebook está preparado para Google Colab: usa comandos de descarga de Colab y rutas bajo /content.

1. Descarga o clona este repositorio y abre el archivo de notebooks en Google Colab.
2. En Colab, selecciona Entorno de ejecución > Cambiar tipo de entorno de ejecución. Puedes activar una GPU para reducir el tiempo; el notebook también puede ejecutarse sin GPU.
3. Ejecuta todas las celdas en orden desde la primera. El notebook descarga las imágenes y anotaciones oficiales, prepara los datos, entrena los modelos, genera las figuras y tablas, y realiza las interpretaciones.
4. La última celda genera la carpeta entrega_EP2, crea README.md y prepara EP2_proyecto.zip. Si Colab no encuentra el archivo notebook en el entorno, la celda solicitará seleccionarlo para copiarlo dentro del paquete.

Para ejecutar fuera de Colab hay que adaptar las celdas de descarga, los comandos de shell y las rutas /content. Se requieren Python, TensorFlow/Keras, NumPy, pandas, scikit-learn, Matplotlib, Seaborn y Pillow.

## Desarrollo experimental

Las imágenes se redimensionan a 64x64 RGB y se normalizan dentro del modelo. La semilla 2604 fija la selección y partición de imágenes; las configuraciones se comparan con las semillas 11, 22 y 33 usando únicamente validación de PARECIDAS. Se elige la configuración de mayor F1 macro medio; entre las que quedan a una desviación estándar, se prefiere la de menos parámetros y después la de mayor F1.

K0 es la CNN base con filtros 32-64-128 y dropout 0,3. K1 cambia el dropout a 0,5; K2 reduce el learning rate; K3 usa menos filtros; K4 agrega regularización L2. K1 fue seleccionada y se evaluó en ambos subconjuntos con cinco semillas (101 a 105). El MLP final de la EP1 se reentrenó con la misma partición y semillas. Se utilizaron EarlyStopping y aumento de datos solo durante entrenamiento.

Además se revisan calidad y similitud de datos, curvas, métricas por raza, calibración, errores, mapas de características y Grad-CAM. Las ablaciones del anexo comparan alternativas de arquitectura solo en validación.

## Resultados

La selección en validación dio a K1 un F1 macro de 0,381 ± 0,028. En prueba, las medias y desviaciones estándar de cinco semillas fueron:

| Subconjunto | Modelo | F1 macro | Accuracy | Recall mínimo |
|---|---|---:|---:|---:|
| DISTINTAS | CNN K1 | 0,518 ± 0,084 | 0,551 ± 0,079 | 0,139 ± 0,083 |
| DISTINTAS | MLP EP1 | 0,487 ± 0,037 | 0,490 ± 0,036 | 0,360 ± 0,059 |
| PARECIDAS | CNN K1 | 0,330 ± 0,064 | 0,349 ± 0,063 | 0,122 ± 0,102 |
| PARECIDAS | MLP EP1 | 0,363 ± 0,041 | 0,365 ± 0,041 | 0,254 ± 0,026 |

La CNN cumple el KPI de F1 macro ≥ 0,40 en DISTINTAS, pero no en PARECIDAS. El KPI de recall mínimo ≥ 0,20 no se cumple para la CNN en ninguno de los dos subconjuntos. La CNN tiene 101.836 parámetros frente a 1.573.644 del MLP, aproximadamente 15,5 veces menos, pero el tiempo observado fue mayor en esta ejecución: unos 25 s frente a 13 s en DISTINTAS y 20 s frente a 11 s en PARECIDAS.

## Interpretación y límites

La CNN mejora los promedios de F1 y accuracy respecto del MLP en DISTINTAS, mientras que el MLP obtiene mejores promedios en PARECIDAS y mayor recall mínimo en ambos grupos. Con cinco semillas y una sola partición no se concluye superioridad estadística general. Las ablaciones de Flatten, filtros 5x5 y BatchNorm superaron al control en validación, pero fueron exploratorias, se realizaron después de revisar el test y no demuestran mejora en prueba. Grad-CAM sugiere que la CNN suele activar más sobre el animal que lo esperado por el área que ocupa, con excepciones por raza y método.

Las cifras dependen del entorno de software y hardware. La semilla mejora la reproducibilidad, pero no garantiza resultados idénticos entre dispositivos o versiones de TensorFlow. La clasificación requiere supervisión humana.
