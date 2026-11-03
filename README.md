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

**Objetivo y métrica.** La coincidencia exacta de atributos (Attribute Match Score), Como metrica utilizamos el $\text{Match Score} = \frac{\text{Numero de restricciones cumplidas}}{\text{Numero de restricciones aplicadas}}$. La metrica del sistema es el promedio del match score en todas las recomendaciones generadas. El objetivo es que $\hat{x}(\text{Match Score}) >= 0.90$

## 2. El sistema

**Cómo decide.**A. Hechos Iniciales (Sensores / Filtros del Usuario)
• 𝐹1: Usuario_Genero_Deseado = "Synthwave" 
• 𝐹2: Usuario_Duracion_Max = 40 (minutos)
• 𝐹3: Usuario_Epoca_Deseada = "80s"
• 𝐹4: Album_Evaluado = "Endless Summer - The Midnight" 
• 𝐹5: Album_Genero = "Synthwave"
• 𝐹6: Album_Duracion = 38
• 𝐹7: Album_Año = 2016 (Época actual, no 80s) 
• 𝐹8: Album_Popularidad = "Alta" 

B. Reglas de Inferencia (9 Reglas en Total)

• Regla 1 (Evaluación de Género): IF Album_Genero == Usuario_Genero_Deseado THEN Coincide_Genero = True
• Regla 2 (Evaluación de Duración): IF Album_Duracion <= Usuario_Duracion_Max THEN Coincide_Duracion = True 
• Regla 3 (Evaluación de Época): IF Album_Año >= 1980 AND Album_Año <= 1989 AND Usuario_Epoca_Deseada == "80s" THEN Coincide_Epoca = True 
• Regla 4 (Cálculo de Match Score Base / Categorización Parcial): IF Coincide_Genero == True AND Coincide_Duracion == True AND NOT Coincide_Epoca THEN Match_Score_Base = 0.67 AND Categoria = "Recomendacion_Parcial" 
• Regla 5 (Ajuste/Bonificación por Popularidad): IF Match_Score_Base >= 0.60 AND Album_Popularidad == "Alta" THEN Match_Score_Final = 0.75
• Regla 6 (Filtro de Inclusión al Feed): IF Match_Score_Final >= 0.60 THEN Estado_Album = "Apto_Para_Mostrar" 
• Regla 7 (Priorización de Posición en Interfaz Web): IF Estado_Album == "Apto_Para_Mostrar" AND Match_Score_Final >= 0.70 THEN Posicion_Feed = "Top_Resultados" 
• Regla 8 (Asignación de Etiqueta/Badge de Destacado): IF Posicion_Feed == "Top_Resultados" AND Album_Popularidad == "Alta" THEN Etiqueta_UI = "Destacado_Popular" 
• Regla 9 (Generación de Explicabilidad - Actuador): IF Posicion_Feed == "Top_Resultados" AND Coincide_Genero == True AND Coincide_Duracion == True THEN Mensaje_Explicabilidad = "Te recomendamos este álbum porque dura menos de 40 min y pertenece al género Synthwave." 

**Variables que usa y por qué.** <Cuáles usa, cuáles descarta y por qué.>

**Cómo se corre en vivo.** <Pasos concretos para resolver un caso nuevo: link al Colab, a la planilla o la cuenta a mano.>

**Dos casos de ejemplo.**

| Caso inventado | Valores | Salida del sistema |
|---|---|---|
| A | <…> | <SÍ/NO> |
| B | <…> | <SÍ/NO> |

## 3. Medición

**Entrenamiento (casos 1–14)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

**Prueba (casos 15–20)**

| | Predice SÍ | Predice NO |
|---|---|---|
| **Real SÍ** | VP = <> | FN = <> |
| **Real NO** | FP = <> | VN = <> |

| Métrica | Entrenamiento | Prueba |
|---|---|---|
| Exactitud | <> | <> |
| Precisión | <> | <> |
| Sensibilidad | <> | <> |
| Especificidad | <> | <> |

**Entrenamiento contra prueba.** <¿Hay diferencia? ¿Por qué? ¿Sobreajuste o generalización?>

## 4. El error más caro

<¿Es peor el falso positivo o el falso negativo en su problema? ¿Por qué, según la ficha? ¿Qué cambiaron en el sistema para cometer menos ese error?>

## 5. ¿Comparado contra qué?

**Línea base trivial.** <Qué resultado saca un sistema que responde siempre lo mismo, y cómo se compara el suyo.>

| Checklist (Clase 6) | Respuesta |
|---|---|
| ¿Qué datos? | <> |
| ¿Qué métrica? | <> |
| ¿Comparado contra qué? | <> |
| ¿Quién lo midió? | <> |

## 6. Riesgo: el camino completo

<Datos → modelo → decisión → consecuencia aplicado a su caso. Al menos un riesgo concreto.>

## 7. Bitácora

**Uso de la IA**

| # | Qué le pedimos | Qué aceptamos | Qué rechazamos y por qué |
|---|---|---|---|
| 1 | <> | <> | <> |

**Corridas contra el evaluador**

| Corrida | Nota | Agujero que encontró | Qué cambiamos |
|---|---|---|---|
| 1 | <> | <> | <> |
