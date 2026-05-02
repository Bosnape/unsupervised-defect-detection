# Unsupervised Defect Detection
**Integrantes:** Pablo Cabrejos · Isabella Camacho · Paula Llanos

## Problema a resolver

La inspección visual de productos industriales es una tarea crítica en manufactura, tradicionalmente realizada de forma manual o mediante sistemas supervisados que requieren grandes volúmenes de imágenes defectuosas etiquetadas. En la práctica, los defectos son eventos raros y su recolección y etiquetado es costoso, lo que hace inviable el uso directo de clasificadores supervisados estándar.

Este proyecto aborda la detección de anomalías visuales en productos industriales como un problema no supervisado: el modelo se entrena exclusivamente con imágenes normales y detecta defectos en inferencia a partir del error de reconstrucción. Una imagen que el modelo no logra reconstruir fielmente se considera anómala. Este enfoque elimina la dependencia de datos etiquetados de defectos y es directamente aplicable a escenarios industriales reales donde dichos datos no están disponibles.

El proyecto implementa y compara tres niveles de complejidad arquitectónica — un autoencoder denso como baseline, un VAE convolucional entrenado desde cero, y un autoencoder con encoder ResNet-18 preentrenado con fine-tuning — evaluados sobre el dataset MVTec AD, benchmark estándar para detección de anomalías industriales. El desempeño de cada arquitectura se mide con AUROC (área bajo la curva ROC), métrica estándar para este tipo de problema que cuantifica qué tan bien el error de reconstrucción separa imágenes normales de defectuosas independientemente de un umbral fijo.

## Datos

Se toma como base The MVTec Anomaly Detection Dataset, un conjunto de datos para evaluar métodos de detección de anomalías con enfoque en la inspección industrial. Contiene más de 5000 imágenes de alta resolución, divididas en quince categorías diferentes de objetos y texturas. Cada categoría incluye un conjunto de imágenes de entrenamiento sin defectos y un conjunto de pruebas con diversos tipos de defectos, así como imágenes sin defectos.

**Referencias**
- Paul Bergmann, Michael Fauser, David Sattlegger, and Carsten Steger, *"A Comprehensive Real-World Dataset for Unsupervised Anomaly Detection"*, IEEE Conference on Computer Vision and Pattern Recognition, 2019
- https://www.mvtec.com/company/research/datasets/mvtec-ad
- https://www.kaggle.com/datasets/ipythonx/mvtec-ad/data

## Arquitectura base

Se elige un Autoencoder denso (fully connected) como arquitectura base por la simplicidad de su implementación (encoder FC, bottleneck, decoder FC), lo que permite depurar el pipeline completo antes de introducir convoluciones. Esta arquitectura define una base de rendimiento comprensible, suficiente para demostrar que la detección por reconstrucción funciona; sus debilidades estructurales (como la pérdida de estructura espacial) son las que las arquitecturas siguientes están diseñadas para corregir. De esta manera cualquier mejora en los próximos pasos es medible y atribuible a decisiones arquitectónicas concretas, no a diferencias de pipeline.

## Arquitectura propuesta según la naturaleza de los datos

Dado que los datos son imágenes de alta resolución de productos industriales sin etiquetas de defecto, se propone un Autoencoder Variacional Convolucional (Conv VAE). El encoder convolucional comprime la imagen en una distribución gaussiana en el espacio latente, produciendo una media y una varianza. A partir de esa distribución se muestrea un vector latente de forma diferenciable mediante el reparameterization trick. El decoder convolucional reconstruye la imagen original a partir de ese vector. El modelo se entrena únicamente con imágenes normales, por lo que el error de reconstrucción actúa como indicador de anomalía.

## Arquitectura con transfer learning

Se propone utilizar ResNet-18 preentrenado en ImageNet como encoder, conectado a un decoder convolucional entrenado desde cero. El entrenamiento se realiza en dos fases: primero el encoder permanece congelado para que el decoder aprenda a reconstruir imágenes normales del dataset industrial, y luego se realiza un fine-tuning completo desbloqueando los pesos de ResNet-18 para ajustar las representaciones al dominio industrial. El resultado es un autoencoder asimétrico donde el encoder aporta representaciones visuales ricas aprendidas de millones de imágenes, refinadas posteriormente para el contexto específico de inspección industrial.
