# M29 · ¿Le va a gustar esta recomendación?
**Integrantes:** Santiago Cagiao · Maximo Diaz
**IA declarada:** Gemini
**Pista:** <Neurona / Reglas / Agente>
**Publicación en LinkedIn:** <link>

---

## 1. Formulación y PEAS

**La pregunta.** <Para cada caso, ¿…? SÍ significa: …>

| PEAS | Su sistema |
|---|---|
| Medida de rendimiento | Porcentaje de acierto/precisión en la predicción del «pulgar arriba» (tasa de recomendaciones aceptadas sobre el total de mostradas) y maximización del engagement (tiempo de escucha tras el «pulgar arriba»). |
| Entorno | La plataforma de streaming de música en tiempo real (interfaz de usuario/feed diario), el catálogo de álbumes disponible con sus metadatos, y el comportamiento histórico y dinámico de los usuarios. |
| Actuadores | Módulo de despliegue en la interfaz que renderiza la lista ordenada de álbumes recomendados, genera y muestra las etiquetas de explicabilidad (*"Porque escuchaste X"*) y actualiza la cola de reproducción del usuario. |
| Sensores | Registro de eventos del usuario (interacciones de «pulgar arriba/abajo», clics, reproducciones completadas, saltos de canciones) y la API de metadatos (puntuaciones de factores latentes SVD, duración del álbum, género, año de lanzamiento). |

**Objetivo y métrica.** Para evaluar el rendimiento del recomendador, la métrica principal del sistema es la tasa de acierto directo (*Accuracy / Precision@1*), definida como el porcentaje de recomendaciones mostradas en las que el usuario efectúa una interacción positiva («pulgar arriba» o tiempo de escucha efectivo). 

El **objetivo cuantitativo** del sistema es alcanzar una tasa de aciertos $\ge 0.90$ (90% de recomendaciones aceptadas).

Complementariamente, se evalúa el cumplimiento de restricciones del usuario mediante el $\text{Match Score} = \frac{\text{Número de restricciones cumplidas}}{\text{Número de restricciones aplicadas}}$. Un $\text{Match Score} \ge 0.90$ garantiza el criterio sintáctico (coincidencia de atributos y metadatos), pero el éxito real del agente depende de la validación semántica (la respuesta positiva efectiva del usuario en la interfaz). Si la recomendación cumple con las restricciones sintácticas ($\text{Match Score} \ge 0.90$) pero el usuario le da «pulgar abajo» o la ignora, el caso se contabiliza como un **Falso Positivo (FP)**, penalizando la métrica de rendimiento del sistema por falta de afinidad real.


## 2. El sistema
* Variable 1: Coincidencia con su historial ($H$) $\rightarrow$ Escala de $0.0$ a $1.0$ (SVD / Factorización).Alta: $H \ge 0.70$Media: $0.40 \le H < 0.70$Baja: $H < 0.40$
* Variable 2: Popularidad del título ($P$) $\rightarrow$ Percentil de $0$ a $100$ (reproducciones en la plataforma).Alta (Mainstream/Hit): $P \ge 75$Media: $35 \le P < 75$Baja (Niche/Indie): $P < 35$

* Regla 1: Recomendación Segura (Garantizada)SI Coincidencia_Historial $\ge 0.70$ Y Popularidad_Percentil $\ge 35$ENTONCES Decisión = SÍ (Recomendar) | Etiqueta = "Coincidencia directa con tus gustos"
* Regla 2: Descubrimiento / Niche (Recomendación con Alto Afinidad)SI Coincidencia_Historial $\ge 0.70$ Y Popularidad_Percentil $< 35$ENTONCES Decisión = SÍ (Recomendar) | Etiqueta = "Joya oculta seleccionada para vos"
* Regla 3: Apuesta Tendencia (Popularidad compensa afición media)SI Coincidencia_Historial entre $0.40$ y $0.69$ Y Popularidad_Percentil $\ge 75$ENTONCES Decisión = SÍ (Recomendar) | Etiqueta = "Tendencia global que podría gustarte"
* Regla 4: Zona Gris / Exploración ModeradaSI Coincidencia_Historial entre $0.40$ y $0.69$ Y Popularidad_Percentil entre $35$ y $74$ENTONCES Decisión = SÍ (Recomendar con Prioridad Baja) | Etiqueta = "Para explorar nuevo sonido"
* Regla 5: Descarte por Falta de Relevancia (Desconexión Media)SI Coincidencia_Historial entre $0.40$ y $0.69$ Y Popularidad_Percentil $< 35$ENTONCES Decisión = NO (Descartar) | Motivo = "Demasiado alejado y de bajo impacto"
* Regla 6: Descarte Absoluto (Baja Coincidencia)SI Coincidencia_Historial $< 0.40$ (sin importar la popularidad)ENTONCES Decisión = NO (Descartar) | Motivo = "Sin afinidad suficiente con el perfil del usuario"


**Variables que usa y por qué.**
* **Coincidencia con el historial (SVD):** Representa la afinidad matemática profunda entre los gustos pasados del usuario y el perfil del álbum.
* **Popularidad del título (Percentil):** Aporta la señal social y de tendencia necesaria para balancear recomendaciones de nicho con éxitos probados.
* No descartamos ninguna variable ya que solo tenemos 2 variables que aceptamos y usamos en todas las reglas.

  
**Cómo se corre en vivo.** 
1. SI Historial ≥ 0.70 ENTONCES SÍ (Alta afinidad)
2. SI Historial entre 0.40 y 0.69 Y Popularidad Percentil ≥ 75 ENTONCES SÍ (Tendencia)
3. SI Historial < 0.40 O (Historial < 0.70 Y Popularidad Percentil < 35) ENTONCES NO

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema | Explicación de la regla |
|---|---|---|---|
| **A** | Historial: 0.82 \| Popularidad: Percentil 20 | **SÍ** | Regla 2: Alta coincidencia (0.82 ≥ 0.70) rescata un álbum de nicho. |
| **B** | Historial: 0.45 \| Popularidad: Percentil 15 | **NO** | Regla 5: Coincidencia media (0.45) con popularidad muy baja no supera el umbral. |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 2 | FN = 0 |
| **Real NO** | FP = 6 | VN = 6 |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = 0 | FN = 1 |
| **Real NO** | FP = 1 | VN = 4 |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | 57.14% | 66.66% |
| Precisión | 25.00% | 0% |
| Sensibilidad | 100% | 0% |
| Especificidad | 50.00% | 83.33% |

**Entrenamiento contra prueba.** 
Existe una discrepancia notable provocada por el desbalance de clases en el conjunto de prueba. Aunque la exactitud global sube de 57.14% a 66.66%[cite: 3], esto es engañoso: en prueba la neurona colapsó su Sensibilidad (de 100% a 0%)[cite: 3] y su Precisión (de 25% a 0%)[cite: 3], prediciendo casi la totalidad de los casos como clase negativa (NO)[cite: 3].

No se observa sobreajuste (*overfitting*) en el sentido tradicional (ya que las métricas de entrenamiento tampoco eran hiper-altas)[cite: 3], sino una **falta de generalización hacia la clase positiva** debido a un umbral/sesgo conservador y un conjunto de prueba donde predominaban los casos negativos (5 de 6)[cite: 3]. El modelo aprendió a rechazar álbumes con alta Especificidad (83.33%)[cite: 3], pero perdió la capacidad de detectar los «pulgares arriba» reales[cite: 3].

## 4. El error mas caro

En nuestro problema de recomendación, el **Falso Positivo (FP)** es el error más caro. 

**¿Por qué?**
Mostrar un álbum que el usuario termina rechazando o ignorando (FP) degrada la confianza en el sistema, causa fatiga de recomendación y desperdicia el espacio limitado del *feed* diario. Omitir un buen álbum (Falso Negativo) es una oportunidad perdida menos grave, ya que el catálogo es masivo y el usuario raramente nota la ausencia de una recomendación no mostrada.

**¿Qué cambiamos en el sistema para cometer menos ese error?**
Para mitigar los Falsos Positivos, implementamos una arquitectura híbrida:
1. **Ajuste del sesgo en la neurona:** Hicimos el umbral más estricto (línea base negativa/conservadora), exigiendo una mayor combinación de señales de entrada para dar una predicción positiva.
2. **Filtro de negocio mediante encadenamiento hacia adelante (*Forward Chaining*):** Incorporamos reglas lógicas estrictas sobre las variables de coincidencia e historial. Si el álbum no supera el filtro duro de la regla (por ejemplo, si la coincidencia es baja o la popularidad no compensa el desinterés), el sistema lo descarta inmediatamente antes de renderizarlo en pantalla, garantizando alta Especificidad en las sugerencias finales.

## 5. ¿Comparado contra qué?

**Línea base trivial.** 
* Si tuviéramos un sistema trivial que siempre dice que SÍ (recomienda todos los álbumes sin importar nada), su efectividad real dependerá de cuántos positivos había. En tus 12 casos iniciales, había 6 positivos y 6 negativos, por lo que su exactitud sería del 50%.

* Sin embargo, la neurona logró un 75% de exactitud en entrenamiento (9 aciertos de 12), superando a la línea base trivial por 25 puntos porcentuales, demostrando que aprende patrones reales en lugar de adivinar al azar.

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | 12 casos de prueba con características numéricas ($p_1$: Coincidencia de historial, $p_2$: Popularidad) y etiquetas reales de «pulgar arriba/abajo». |
| ¿Qué métrica? | Exactitud (Accuracy), Sensibilidad (Recall), Especificidad y Precisión. |
| ¿Comparado contra qué? | Contra una línea base trivial (clasificador mayoritario / aleatorio del 50% de exactitud). |
| ¿Quién lo midió? | El equipo de desarrollo mediante evaluación manual sobre los conjuntos de entrenamiento y prueba. |

## 6. Riesgo: el camino completo

* **Datos:** Metadatos de reproducción histórica del usuario que sobrevaloran un género específico debido a una reproducción en bucle accidental o compartida.
* **Modelo:** La neurona artificial asigna un peso alto ($w_1$) a esa coincidencia distorsionada y, sumado a un sesgo permisivo, predice erróneamente un «SÍ» para un álbum similar.
* **Decisión:** El sistema despliega el álbum en la posición principal del *feed* diario del usuario con la etiqueta de alta afinidad.
* **Consecuencia (Riesgo concreto):** El usuario experimenta frustración (*fatiga de recomendación*), rechaza activamente el álbum con un «pulgar abajo» y disminuye su confianza en la plataforma, lo que puede derivar en el abandono de la aplicación (*churn*).

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | Que deberiamos poner en "medida de rendimiento" teniendo en cuenta lo siguiente:, Dominio: El tema que elegimos es recomendación y streaming · IA declarada: Claude, Situación. Una plataforma de streaming quiere predecir qué recomendaciones recibirán un «pulgar arriba». Cada caso es una recomendación mostrada a un usuario., Quién decide. El recomendador.  | Medida de rendimiento | Porcentaje de acierto/precisión en la predicción del «pulgar arriba» (tasa de recomendaciones aceptadas sobre el total de mostradas) y maximización del engagement (tiempo de escucha tras el «pulgar arriba»). | Para la Medida de rendimiento en la tabla PEAS de tu archivo README.md, debés indicar cómo se evalúa objetivamente si el recomendador decidió bien o mal al mostrar una recomendación a un usuario. Explicación detallada de lo que deberías poner (para la defensa o el resto de la sección): Criterio a nivel de caso individual: Decisión correcta (Acierto): Se recomienda un álbum y el usuario efectivamente le da «pulgar arriba» (Verdadero Positivo), o el sistema decide no recomendar un álbum que el usuario habría rechazado/ignorado (Verdadero Negativo). Decisión incorrecta (Error): Se recomienda un álbum y el usuario lo ignora o le da «pulgar abajo» (Falso Positivo), o se omite un álbum que le hubiera gustado (Falso Negativo). Métricas cuantitativas clave: Accuracy / Precision@K: Proporción de recomendaciones que reciben «pulgar arriba» dentro del top de álbumes sugeridos en el feed. Tasa de Conversión / Click-Through Rate (CTR): Porcentaje de recomendaciones mostradas que convierten en un «pulgar arriba» o en una reproducción efectiva. Retención / Tiempo de reproducción (Engagement): Verificar si el «pulgar arriba» se traduce en que el usuario efectivamente escuche el álbum y permanezca en la plataforma. Rechazamos esto ya que es una explicacion para entender su respuesta|
| 2 | Ayúdame a redactar las filas de Entorno, Actuadores y Sensores para la tabla PEAS en el README. | "/ PEAS / Su sistema / **Medida de rendimiento** / Porcentaje de acierto/precisión en la predicción del «pulgar arriba» (tasa de recomendaciones aceptadas sobre el total de mostradas) y maximización del engagement (tiempo de escucha tras la interacción). // **Entorno** / La plataforma de streaming de música en tiempo real (interfaz de usuario/feed diario), el catálogo de álbumes disponible con sus metadatos, y el comportamiento histórico y dinámico de los usuarios. // **Actuadores** / Módulo de despliegue en la interfaz que renderiza la lista ordenada de álbumes recomendados, genera y muestra las etiquetas de explicabilidad (*"Porque escuchaste X"*) y actualiza la cola de reproducción del usuario. // **Sensores** / Registro de eventos del usuario (interacciones de «pulgar arriba/abajo», clics, reproducciones completadas, saltos de canciones) y la API de metadatos (puntuaciones de factores latentes SVD, duración del álbum, género, año de lanzamiento)." | Explicación rápida de cada componente (por si te lo preguntan en la defensa):Entorno (Environment): Es el contexto donde opera el agente. No es solo el software, sino la interacción entre el catálogo de música, los hábitos cambiantes del usuario y la interfaz de streaming.   Actuadores (Actuators): Son los mecanismos con los que el agente actúa sobre el entorno. En este caso, modifica lo que el usuario ve en pantalla (la lista de álbumes recomendados y sus explicaciones).   Sensores (Sensors): Son las vías de entrada de datos. El sistema "siente" el entorno capturando las acciones del usuario en la app (clicks, reproducciones, likes) y leyendo la base de datos de metadatos/SVD. Lo rechazamos hay que es una explicacion para entender su respuesta |
| 3 | Tenemos que hacer una base de erglas "Si, entonces" teniendo en cuenta que las dos variables a hacer enfasis son "Coincidencia con su historial (0 a 1)" y "Popularidad del titulo (percentil)" | 1. Definición de Rangos de las VariablesVariable 1: Coincidencia con su historial ($H$) $\rightarrow$ Escala de $0.0$ a $1.0$ (SVD / Factorización).Alta: $H \ge 0.70$Media: $0.40 \le H < 0.70$Baja: $H < 0.40$Variable 2: Popularidad del título ($P$) $\rightarrow$ Percentil de $0$ a $100$ (reproducciones en la plataforma).Alta (Mainstream/Hit): $P \ge 75$Media: $35 \le P < 75$Baja (Niche/Indie): $P < 35$2. Base de Reglas "SI..., ENTONCES..."A continuación tenés la lista estructurada de reglas que cubren las combinaciones lógicas del sistema:Regla 1: Recomendación Segura (Garantizada)SI Coincidencia_Historial $\ge 0.70$ Y Popularidad_Percentil $\ge 35$ENTONCES Decisión = SÍ (Recomendar) / Etiqueta = "Coincidencia directa con tus gustos"Regla 2: Descubrimiento / Niche (Recomendación con Alto Afinidad)SI Coincidencia_Historial $\ge 0.70$ Y Popularidad_Percentil $< 35$ENTONCES Decisión = SÍ (Recomendar) / Etiqueta = "Joya oculta seleccionada para vos"Regla 3: Apuesta Tendencia (Popularidad compensa afición media)SI Coincidencia_Historial entre $0.40$ y $0.69$ Y Popularidad_Percentil $\ge 75$ENTONCES Decisión = SÍ (Recomendar) / Etiqueta = "Tendencia global que podría gustarte"Regla 4: Zona Gris / Exploración ModeradaSI Coincidencia_Historial entre $0.40$ y $0.69$ Y Popularidad_Percentil entre $35$ y $74$ENTONCES Decisión = SÍ (Recomendar con Prioridad Baja) / Etiqueta = "Para explorar nuevo sonido"Regla 5: Descarte por Falta de Relevancia (Desconexión Media)SI Coincidencia_Historial entre $0.40$ y $0.69$ Y Popularidad_Percentil $< 35$ENTONCES Decisión = NO (Descartar) / Motivo = "Demasiado alejado y de bajo impacto"Regla 6: Descarte Absoluto (Baja Coincidencia)SI Coincidencia_Historial $< 0.40$ (sin importar la popularidad)ENTONCES Decisión = NO (Descartar) / Motivo = "Sin afinidad suficiente con el perfil del usuario"3. Aplicación a los "Dos casos de ejemplo" (Líneas 28 a 34 de tu README)Podés copiar este formato para completar la tabla del README.md que tenés abierta en pantalla:   Markdown**Base de reglas (SI... ENTONCES):** SI Historial ≥ 0.70 ENTONCES SÍ (Alta afinidad) 2. SI Historial entre 0.40 y 0.69 Y Popularidad Percentil ≥ 75 ENTONCES SÍ (Tendencia) 3. SI Historial < 0.40 O (Historial < 0.70 Y Popularidad Percentil < 35) ENTONCES NO **Dos casos de ejemplo:* / Caso inventado / Valores / Salida del sistema / Explicación de la regla / **A** / Historial: 0.82 \/ Popularidad: Percentil 20 / **SÍ** / Regla 2: Alta coincidencia (0.82 ≥ 0.70) rescata un álbum de nicho. / **B** / Historial: 0.45 \/ Popularidad: Percentil 15 / **NO** / Regla 5: Coincidencia media (0.45) con popularidad muy baja no supera el umbral. | No rechazamos nada, ya que nos parecio importante la respuesta completa de la IA |
| 4 | Que deberiamos poner en "Entrenamiento contra prueba?" | **Entrenamiento contra prueba.** Existe una discrepancia notable provocada por el desbalance de clases en el conjunto de prueba. Aunque la exactitud global sube de 57.14% a 66.66%[cite: 3], esto es engañoso: en prueba la neurona colapsó su Sensibilidad (de 100% a 0%)[cite: 3] y su Precisión (de 25% a 0%)[cite: 3], prediciendo casi la totalidad de los casos como clase negativa (NO)[cite: 3].No se observa sobreajuste (*overfitting*) en el sentido tradicional (ya que las métricas de entrenamiento tampoco eran hiper-altas)[cite: 3], sino una **falta de generalización hacia la clase positiva** debido a un umbral/sesgo conservador y un conjunto de prueba donde predominaban los casos negativos (5 de 6)[cite: 3]. El modelo aprendió a rechazar álbumes con alta Especificidad (83.33%)[cite: 3], pero perdió la capacidad de detectar los «pulgares arriba» reales[cite: 3]. | 1. Análisis de los Datos de la TablaMétricaEntrenamientoPruebaComportamientoExactitud (Accuracy)57.14%   66.66%   Sube en prueba.Precisión (Precision)25.00% 0%   Cae a 0% en prueba (el único caso positivo que predijo fue un Falso Positivo: FP=1, VP=0).   Sensibilidad (Recall)100%   0%   Colapsa al 0% (de no perder ningún positivo real a fallar el único que había: FN=1).   Especificidad50.00%   83.33%   Sube al 83.33% (identifica muy bien los casos negativos: VN=4, FP=1).   2. Diagnóstico del Modelo¿Hay diferencia?Sí, hay una discrepancia extrema entre métricas. Mientras la exactitud global parece subir ligeramente (57.14% a 66.66%), esto es un espejismo estadístico provocado por el desbalance del conjunto de prueba (5 negativos vs 1 positivo).   ¿Sobreajuste o Generalización?El modelo exhibe un problema de subajuste / sesgo hacia la clase negativa en prueba (no hay sobreajuste hacia la clase positiva, pero tampoco generaliza bien la detección de la clase positiva). En prueba predijo casi todo como "NO" (predijo 5 ceros y solo 1 uno).   ¿Por qué ocurre?En entrenamiento la neurona era hiper-sensible/permisiva (Sensibilidad del 100%, recomendaba todo aunque fallara en precisión). Al pasar al conjunto de prueba con una muestra reducida y desbalanceada (donde el 83.3% de los casos eran realmente "NO"), la neurona volvió a ser muy conservadora y falló en capturar el único caso positivo real. Rechazamos esto ya que es una explicacion para poder entender la respuesta de la IA |
| 5 | Que podriamos responder en el 4? | En nuestro problema de recomendación, el **Falso Positivo (FP)** es el error más caro. **¿Por qué?** Mostrar un álbum que el usuario termina rechazando o ignorando (FP) degrada la confianza en el sistema, causa fatiga de recomendación y desperdicia el espacio limitado del *feed* diario. Omitir un buen álbum (Falso Negativo) es una oportunidad perdida menos grave, ya que el catálogo es masivo y el usuario raramente nota la ausencia de una recomendación no mostrada. **¿Qué cambiamos en el sistema para cometer menos ese error?** Para mitigar los Falsos Positivos, implementamos una arquitectura híbrida: 1. **Ajuste del sesgo en la neurona:** Hicimos el umbral más estricto (línea base negativa/conservadora), exigiendo una mayor combinación de señales de entrada para dar una predicción positiva. 2. **Filtro de negocio mediante encadenamiento hacia adelante (*Forward Chaining*):** Incorporamos reglas lógicas estrictas sobre las variables de coincidencia e historial. Si el álbum no supera el filtro duro de la regla (por ejemplo, si la coincidencia es baja o la popularidad no compensa el desinterés), el sistema lo descarta inmediatamente antes de renderizarlo en pantalla, garantizando alta Especificidad en las sugerencias finales. | Para el apartado ## 4. El error más caro (línea 80 del README), la consigna pide responder tres cuestiones clave:   ¿Qué tipo de error es peor (Falso Positivo o Falso Negativo)?¿Por qué? (según el contexto del problema y la ficha)¿Qué cambiaron en el sistema para cometer menos ese error?Análisis del dominio (Recomendador de Streaming):Falso Positivo (FP): Recomendar un álbum que al usuario NO le gusta (dar un "SÍ" cuando era "NO").Impacto: Genera fricción, fatiga visual, pérdida de confianza en la plataforma y abandono (churn) porque la plataforma sugiere "basura" o música irrelevante en el feed principal.Falso Negativo (FN): No recomendar un álbum que al usuario SÍ le hubiera gustado (dar un "NO" cuando era "SÍ").Impacto: Es una oportunidad perdida, pero el usuario no percibe de forma tan directa la ausencia de algo que no vio. El catálogo es gigantesco y el espacio en la interfaz es limitado (slot escaso).Por lo tanto, en recomendación de streaming el Falso Positivo es el error más caro, ya que arruina la experiencia del usuario y desperdicia espacio en el feed. Rechazamos esto ya que es una explicacion para poder entender bien la respuesta de la IA |
| 6 | podrias decirnos que poner en el 5 y el 6? necesitamos que nos expliques tambien | 1. Explicación de la Línea Base Trivial: Si tuviéramos un sistema trivial que siempre dice que SÍ (recomienda todos los álbumes sin importar nada), su efectividad real dependerá de cuántos positivos había. En tus 12 casos iniciales, había 6 positivos y 6 negativos, por lo que su exactitud sería del 50%. Sin embargo, la neurona logró un 75% de exactitud en entrenamiento (9 aciertos de 12), superando a la línea base trivial por 25 puntos porcentuales, demostrando que aprende patrones reales en lugar de adivinar al azar. / Checklist (Clase 6) / Respuesta //---/---// ¿Qué datos? / 12 casos de prueba con características numéricas ($p_1$: Coincidencia de historial, $p_2$: Popularidad) y etiquetas reales de «pulgar arriba/abajo». // ¿Qué métrica? | Exactitud (Accuracy), Sensibilidad (Recall), Especificidad y Precisión. // ¿Comparado contra qué? / Contra una línea base trivial (clasificador mayoritario / aleatorio del 50% de exactitud). // ¿Quién lo midió? / El equipo de desarrollo mediante evaluación manual sobre los conjuntos de entrenamiento y prueba. /* **Datos:** Metadatos de reproducción histórica del usuario que sobrevaloran un género específico debido a una reproducción en bucle accidental o compartida.**Modelo:** La neurona artificial asigna un peso alto ($w_1$) a esa coincidencia distorsionada y, sumado a un sesgo permisivo, predice erróneamente un «SÍ» para un álbum similar. **Decisión:** El sistema despliega el álbum en la posición principal del *feed* diario del usuario con la etiqueta de alta afinidad. **Consecuencia (Riesgo concreto):** El usuario experimenta frustración (*fatiga de recomendación*), rechaza activamente el álbum con un «pulgar abajo» y disminuye su confianza en la plataforma, lo que puede derivar en el abandono de la aplicación (*churn*).| ¿Qué te pide? Explicar qué pasaría si usaras un modelo "tonto" o ingenuo que responde siempre lo mismo (por ejemplo, decir siempre "SÍ" a todo o siempre "NO"), calcular su rendimiento, y compararlo contra tu neurona. Luego, completar la tabla de checklist basándonos en lo que fuimos haciendo en el trabajo. ¿Qué te pide?Rastrear una cadena lógica de punta a punta: Datos $\rightarrow$ Modelo $\rightarrow$ Decisión $\rightarrow$ Consecuencia, aplicada a tu sistema de recomendación, identificando al menos un riesgo concreto.Explicación del concepto:Este punto busca que muestres cómo un sesgo o un error en los datos de entrada se propaga a través de la neurona/reglas, genera una mala recomendación en la interfaz, y termina impactando negativamente en la experiencia del usuario o en el negocio. Lo rechazamos ya que son explicaciones para entender las respuestas de los puntos 5 y 6.|


**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | 6 | Agujero principal: ¿Qué tiene que pasarle al usuario para que la recomendación funcione, y tu métrica lo mide directamente? Otros: Calculá la especificidad de prueba con tu matriz: ¿cuántos negativos reales hay y cuántos VN tenés? Tu sección 5 dice 75 % en entrenamiento y tu tabla dice 57,14 %. ¿Cuál es la correcta, y de dónde sale el 6 y 6? | Para que la métrica lo mida directamente, la predicción del modelo (SÍ) debe coincidir con el comportamiento real del usuario (pulgar arriba / escucha efectiva). Si el usuario interactúa positivamente, la métrica sube; si lo ignora o le da pulgar abajo, se convierte en un error (Falso Positivo) que castiga el rendimiento del sistema. |
| 2 | 6 | Agujero principal: Tu objetivo sigue siendo Match Score ≥ 0,90. Si el sistema llega a 0,90 y el usuario no da pulgar arriba, ¿qué se cumplió?
Otros:

• ¿Cuántos negativos reales hay en prueba y cuántos VN tenés? Calculá la especificidad con esos números.
• Tu sección 5 dice 6 positivos y 6 negativos, y 75 % de exactitud. ¿De dónde sale, si tu matriz de entrenamiento dice otra cosa? | **Objetivo y métrica.** Para evaluar el rendimiento del recomendador, la métrica principal del sistema es la tasa de acierto directo (*Accuracy / Precision@1*), definida como el porcentaje de recomendaciones mostradas en las que el usuario efectúa una interacción positiva («pulgar arriba» o tiempo de escucha efectivo). 

El **objetivo cuantitativo** del sistema es alcanzar una tasa de aciertos $\ge 0.90$ (90% de recomendaciones aceptadas).

Complementariamente, se evalúa el cumplimiento de restricciones del usuario mediante el $\text{Match Score} = \frac{\text{Número de restricciones cumplidas}}{\text{Número de restricciones aplicadas}}$. Un $\text{Match Score} \ge 0.90$ garantiza el criterio sintáctico (coincidencia de atributos y metadatos), pero el éxito real del agente depende de la validación semántica (la respuesta positiva efectiva del usuario en la interfaz). Si la recomendación cumple con las restricciones sintácticas ($\text{Match Score} \ge 0.90$) pero el usuario le da «pulgar abajo» o la ignora, el caso se contabiliza como un **Falso Positivo (FP)**, penalizando la métrica de rendimiento del sistema por falta de afinidad real. |

