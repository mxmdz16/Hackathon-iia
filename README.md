=== PARTE 1 DE 5 ===



# M29 · ¿Le va a gustar esta recomendación?

**Integrantes:** Santiago Cagiao · Maximo Diaz  

**IA declarada:** Gemini  

**Pista:** Reglas  

**Publicación en LinkedIn:** <link>  



---



## 1. Formulación y PEAS



**La pregunta.** Para cada caso, ¿el usuario le dará «pulgar arriba» a este álbum recomendado? SÍ significa: El usuario interactúa positivamente (pulgar arriba / escucha efectiva tras la recomendación).



| PEAS | Su sistema |

|---|---|

| Medida de rendimiento | Porcentaje de acierto/precisión en la predicción del «pulgar arriba» (tasa de recomendaciones aceptadas sobre el total de mostradas) y maximización del engagement (tiempo de escucha tras el «pulgar arriba»). |

| Entorno | La plataforma de streaming de música en tiempo real (interfaz de usuario/feed diario), el catálogo de álbumes disponible con sus metadatos, y el comportamiento histórico y dinámico de los usuarios. |

| Actuadores | Módulo de despliegue en la interfaz que renderiza la lista ordenada de álbumes recomendados, genera y muestra las etiquetas de explicabilidad ("Porque escuchaste X") y actualiza la cola de reproducción del usuario. |

| Sensores | Registro de eventos del usuario (interacciones de «pulgar arriba/abajo», clics, reproducciones completadas, saltos de canciones) y la API de metadatos (puntuaciones de factores latentes SVD, duración del álbum, género, año de lanzamiento). |



**Objetivo y métrica.** Para evaluar el rendimiento del recomendador, la métrica principal del sistema es la tasa de acierto directo (Accuracy / Precision@1), definida como el porcentaje de recomendaciones mostradas en las que el usuario efectúa una interacción positiva («pulgar arriba» o tiempo de escucha efectivo). 



El objetivo cuantitativo del sistema es alcanzar una tasa de aciertos >= 0.90 (90% de recomendaciones aceptadas).



Complementariamente, se evalúa el cumplimiento de restricciones del usuario mediante el Match Score = (Número de restricciones cumplidas) / (Número de restricciones aplicadas). Un Match Score >= 0.90 garantiza el criterio sintáctico (coincidencia de atributos y metadatos), pero el éxito real del agente depende de la validación semántica (la respuesta positiva efectiva del usuario en la interfaz). Si la recomendación cumple con las restricciones sintácticas (Match Score >= 0.90) pero el usuario le da «pulgar abajo» o la ignora, el caso se contabiliza como un Falso Positivo (FP), penalizando la métrica de rendimiento del sistema por falta de afinidad real.



=== PARTE 2 DE 5 ===



## 2. El sistema

- Variable 1: Coincidencia con su historial (H) -> Escala de 0.0 a 1.0 (SVD / Factorización). Alta: H >= 0.70 | Media: 0.40 <= H < 0.70 | Baja: H < 0.40

- Variable 2: Popularidad del título (P) -> Percentil de 0 a 100 (reproducciones globales en la plataforma). Alta (Mainstream/Hit): P >= 75 | Media: 35 <= P < 75 | Baja (Niche/Indie): P < 35



**Origen y disponibilidad de los datos (H y P):**  

- H (Coincidencia con el historial) se calcula mediante la factorización matricial (SVD) sobre la matriz de interacciones previas usuario-álbum.  

- P (Popularidad) se calcula a partir del volumen acumulado de reproducciones globales del catálogo en la plataforma, convertido a percentiles.  

*¿Existen al momento de decidir?* Sí. Ambos valores están precalculados en la base de datos o se obtienen vía API de metadatos en el milisegundo previo a la inferencia, por lo que el sistema dispone de H y P de forma sincrónica antes de evaluar las reglas y decidir el despliegue.



- Regla 1: Recomendación Segura (Garantizada)

SI Coincidencia_Historial >= 0.70 Y Popularidad_Percentil >= 35 ENTONCES Decisión = SÍ (Recomendar) | Etiqueta = "Coincidencia directa con tus gustos"

- Regla 2: Descubrimiento / Niche (Recomendación con Alto Afinidad)

SI Coincidencia_Historial >= 0.70 Y Popularidad_Percentil < 35 ENTONCES Decisión = SÍ (Recomendar) | Etiqueta = "Joya oculta seleccionada para vos"

- Regla 3: Apuesta Tendencia (Popularidad compensa afición media)

SI Coincidencia_Historial entre 0.40 y 0.69 Y Popularidad_Percentil >= 75 ENTONCES Decisión = SÍ (Recomendar) | Etiqueta = "Tendencia global que podría gustarte"

- Regla 4: Zona Gris / Exploración Moderada

SI Coincidencia_Historial entre 0.40 y 0.69 Y Popularidad_Percentil entre 35 y 74 ENTONCES Decisión = SÍ (Recomendar con Prioridad Baja) | Etiqueta = "Para explorar nuevo sonido"

- Regla 5: Descarte por Falta de Relevancia (Desconexión Media)

SI Coincidencia_Historial entre 0.40 y 0.69 Y Popularidad_Percentil < 35 ENTONCES Decisión = NO (Descartar) | Motivo = "Demasiado alejado y de bajo impacto"

- Regla 6: Descarte Absoluto (Baja Coincidencia)

SI Coincidencia_Historial < 0.40 (sin importar la popularidad) ENTONCES Decisión = NO (Descartar) | Motivo = "Sin afinidad suficiente con el perfil del usuario"



**Variables que usa y por qué.**

- Coincidencia con el historial (SVD): Representa la afinidad matemática profunda entre los gustos pasados del usuario y el perfil del álbum.

- Popularidad del título (Percentil): Aporta la señal social y de tendencia necesaria para balancear recomendaciones de nicho con éxitos probados.

- No descartamos ninguna variable ya que solo tenemos 2 variables que aceptamos y usamos en todas las reglas.



**Cómo se corre en vivo.**

1. SI Historial >= 0.70 ENTONCES SÍ (Alta afinidad)

2. SI Historial entre 0.40 y 0.69 Y Popularidad Percentil >= 75 ENTONCES SÍ (Tendencia)

3. SI Historial < 0.40 O (Historial < 0.70 Y Popularidad Percentil < 35) ENTONCES NO



**Dos casos de ejemplo.**

- Caso A | Historial: 0.82 | Popularidad: Percentil 20 | Salida: SÍ | Regla 2: Alta coincidencia (0.82 >= 0.70) rescata un álbum de nicho.

- Caso B | Historial: 0.45 | Popularidad: Percentil 15 | Salida: NO | Regla 5: Coincidencia media (0.45) con popularidad muy baja no supera el umbral.



=== PARTE 3 DE 5 ===



## 3. Medición



**Entrenamiento (casos 1–14)**

- Real SÍ: VP = 2 | FN = 0

- Real NO: FP = 6 | VN = 6



**Prueba (casos 15–20)**

- Real SÍ: VP = 0 | FN = 1

- Real NO: FP = 1 | VN = 4



| Métrica | Entrenamiento | Prueba |

|---|---|---|

| Exactitud | 57.14% | 66.66% |

| Precisión | 25.00% | 0.00% |

| Sensibilidad | 100.00% | 0.00% |

| Especificidad | 50.00% | 80.00% |



**Entrenamiento contra prueba.**

Existe una discrepancia notable provocada por el desbalance de clases en el conjunto de prueba. Aunque la exactitud global sube de 57.14% a 66.66%, esto es engañoso: en prueba la neurona colapsó su Sensibilidad (de 100% a 0%) y su Precisión (de 25% a 0%), prediciendo casi la totalidad de los casos como clase negativa (NO).



No se observa sobreajuste (overfitting) en el sentido tradicional (ya que las métricas de entrenamiento tampoco eran hiper-altas), sino una falta de generalización hacia la clase positiva debido a un umbral/sesgo conservador y un conjunto de prueba donde predominaban los casos negativos (5 de 6 real NO, con VN=4 y FP=1, lo que otorga un 80.00% de Especificidad). El modelo aprendió a rechazar álbumes con alta Especificidad, pero perdió la capacidad de detectar los «pulgares arriba» reales.



## 4. El error mas caro



En nuestro problema de recomendación, el Falso Positivo (FP) es el error más caro.



**¿Por qué?**

Mostrar un álbum que el usuario termina rechazando o ignorando (FP) degrada la confianza en el sistema, causa fatiga de recomendación y desperdicia el espacio limitado del feed diario. Omitir un buen álbum (Falso Negativo) es una oportunidad perdida menos grave, ya que el catálogo es masivo y el usuario raramente nota la ausencia de una recomendación no mostrada.



**¿Qué cambiamos en el sistema para cometer menos ese error?**

Para mitigar los Falsos Positivos, implementamos una arquitectura híbrida:

1. Ajuste del sesgo en la neurona: Hicimos el umbral más estricto (línea base negativa/conservadora), exigiendo una mayor combinación de señales de entrada para dar una predicción positiva.

2. Filtro de negocio mediante encadenamiento hacia adelante (Forward Chaining): Incorporamos reglas lógicas estrictas sobre las variables de coincidencia e historial. Si el álbum no supera el filtro duro de la regla (por ejemplo, si la coincidencia es baja o la popularidad no compensa el desinterés), el sistema lo descarta inmediatamente antes de renderizarlo en pantalla, garantizando alta Especificidad en las sugerencias finales.



=== PARTE 4 DE 5 ===



## 5. ¿Comparado contra qué?



**Línea base trivial.**

- En nuestro conjunto de entrenamiento de 14 casos (2 reales SÍ y 12 reales NO), un sistema trivial "negativo" que responda siempre NO alcanza una exactitud del 85.71% (12/14 aciertos por puro desbalance). En cambio, la neurona alcanza un 57.14% (8/14).

- **¿Cuál es más alto y qué conclusión sacamos?** La exactitud del sistema trivial que responde siempre NO (85.71%) es numéricamente mayor que la de la neurona (57.14%). Sin embargo, la conclusión sobre la neurona es que la exactitud es una métrica engañosa ante clases desbalanceadas: un sistema que responde siempre NO es completamente inútil para un recomendador porque posee Sensibilidad = 0% (nunca sugiere nada y jamás descubre una recomendación positiva). La neurona, en cambio, logra una Sensibilidad del 100% en entrenamiento al capturar todos los «pulgares arriba» reales (2/2), demostrando capacidad operativa real para sugerir contenido, aunque requiere afinar la precisión para reducir los falsos positivos.



| Checklist (Clase 6) | Respuesta |

|---|---|

| ¿Qué datos? | 14 casos de entrenamiento y 6 de prueba con características numéricas (p1: Coincidencia de historial, p2: Popularidad) y etiquetas reales de «pulgar arriba/abajo». |

| ¿Qué métrica? | Exactitud (Accuracy), Sensibilidad (Recall), Especificidad y Precisión. |

| ¿Comparado contra qué? | Contra una línea base trivial (clasificador mayoritario / aleatorio). |

| ¿Quién lo midió? | El equipo de desarrollo mediante evaluación manual sobre los conjuntos de entrenamiento y prueba. |



## 6. Riesgo: el camino completo



- Datos: Metadatos de reproducción histórica del usuario que sobrevaloran un género específico debido a una reproducción en bucle accidental o compartida.

- Modelo: La neurona artificial asigna un peso alto (w1) a esa coincidencia distorsionada y, sumado a un sesgo permisivo, predice erróneamente un «SÍ» para un álbum similar.

- Decisión: El sistema despliega el álbum en la posición principal del feed diario del usuario con la etiqueta de alta afinidad.

- Consecuencia (Riesgo concreto): El usuario experimenta frustración (fatiga de recomendación), rechaza activamente el álbum con un «pulgar abajo» y disminuye su confianza en la plataforma, lo que puede derivar en el abandono de la aplicación (churn).



=== PARTE 5 DE 5 ===



## 7. Bitácora



**Uso de la IA**

- Se utilizó IA para estructurar la tabla PEAS, definir las 6 reglas lógicas, analizar la matriz de confusión y justificar el impacto del Falso Positivo.



**Corridas contra el evaluador**



| Corrida | Nota | Agujero que encontró | Qué cambiamos |

|---|---|---|---|

| 1 | 6 | 1. ¿Qué tiene que pasarle al usuario para que la recomendación funcione, y tu métrica lo mide directamente? <br> 2. Calculá la especificidad de prueba con tu matriz. <br> 3. Discrepancia en la sección 5 entre 75% y 57,14%. | Se redefinió la métrica principal a la interacción real del usuario (Accuracy/Precision@1) y se ajustaron los datos de la sección 5 a la matriz de 14 casos. |

| 2 | 6 | 1. Tu objetivo sigue siendo Match Score >= 0.90. Si llega a 0.90 y no hay pulgar arriba, ¿qué se cumplió? <br> 2. Cantidad de negativos reales en prueba y VN para calcular especificidad. <br> 3. Discrepancia de los 6 positivos y 6 negativos en sección 5. | 1. Se aclaró la distinción entre criterio sintáctico (Match Score) y semántico (interacción del usuario). <br> 2. Se corrigió la Especificidad de prueba a 80.00% (4 VN de 5 negativos reales). <br> 3. Se unificaron los datos de la línea base trivial con el total de 14 casos. |

| 3 | 6 | 1. Mostrás 85,71% para un sistema que responde siempre NO y 57,14% para tu neurona. ¿Cuál es más alto y qué conclusión sacás? <br> 2. Campos sin llenar (pregunta y pista). <br> 3. Origen y existencia de H y P en el momento de decisión. | 1. Se explicó que el 85.71% es mayor por desbalance pero inútil (Recall 0%), rescatando la Sensibilidad (100%) de la neurona. <br> 2. Se completaron la pista ("Reglas") y la pregunta formulada en 1. <br> 3. Se justificó la disponibilidad previa de H (SVD) y P (percentil reproducciones). |
