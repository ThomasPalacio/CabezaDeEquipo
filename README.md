# Cabeza de Equipo — Asistente con RAG sobre psicología del deporte

Trabajo Práctico de NLP. Sistema de **Retrieval-Augmented Generation** en español sobre un corpus
de artículos científicos de psicología del deporte aplicada al fútbol formativo.

**Integrantes:** Thomas Palacio y Ezequiel Suñer
**Fecha de entrega:** 09/10/2026

---

## Qué hace

Un asistente orientado al **futbolista juvenil**. La premisa: el acompañamiento psicológico
profesional está fuera del alcance económico de la mayoría, mientras que la evidencia científica
sobre esas mismas cuestiones es de acceso abierto pero está escrita en un registro inaccesible
para un adolescente. El sistema recupera los fragmentos pertinentes y responde en lenguaje llano,
sin inventar.

> **Alcance.** Herramienta de divulgación y orientación. No reemplaza el acompañamiento de un
> profesional de la salud mental.

## Historia del proyecto

El trabajo atravesó dos iteraciones, y la segunda nació de un hallazgo de la primera. Lo contamos
porque el camino explica decisiones que de otro modo parecerían arbitrarias.

### Primera iteración: cinco artículos científicos

El corpus inicial fueron **cinco artículos científicos** de acceso abierto —176.872 caracteres—
elegidos por tres criterios: que el modelo generador no los conociera, para que la comparación CON
RAG vs SIN RAG mostrara un contraste medible; que su estructura rígida permitiera contrastar las
dos estrategias de chunking; y que sus datos fueran verificables, para anotar el conjunto de
evaluación sin ambigüedad.

Esa configuración funcionó bien en lo que medía. El retrieval alcanzó valores altos y el RAG
redujo las alucinaciones de forma clara frente al modelo sin contexto.

### El hallazgo que cambió el rumbo

El problema apareció al agregar un segundo conjunto de preguntas, escritas en el registro de un
jugador de 16 años en lugar del registro académico del primero: *"me pongo re nervioso antes de
los partidos, cómo hago para calmarme"*.

Las respuestas se leían bien. Pero al medir la cobertura del corpus encontramos que términos como
**respiración**, **relajación**, **dormir** o **autoestima** no aparecían **ni una sola vez** en
los 176.872 caracteres. El asistente respondía con listas de técnicas de respiración y meditación
que no provenían del corpus sino de la memoria paramétrica del modelo. La verificación automática
lo confirmó de forma independiente: para ese tipo de consulta, las respuestas con contexto y sin
contexto eran prácticamente idénticas.

El diagnóstico fue que **el corpus respondía bien "¿qué me está pasando y por qué?" y mal "¿cómo
lo soluciono?"**. Los artículos empíricos reportan qué se midió, no cómo se hace: la proporción
entre construcciones instructivas y descriptivas era de 0,3 a 1.

Dicho de otro modo: el sistema funcionaba, pero no para la persona que decíamos que lo iba a usar.

### Segunda iteración: incorporar el registro aplicado

Se sumaron **cuatro fuentes de registro instructivo** —un libro de entrenamiento mental, una tesis
de maestría y dos materiales formativos— elegidas por tener el perfil inverso al de los artículos.

| Término | Corpus inicial | Corpus actual |
|---|---|---|
| respiración | 0 | 111 |
| relajación | 0 | 86 |
| visualización | 1 | 216 |
| autodiálogo | 1 | 6 |
| dormir | 0 | 12 |

La proporción entre construcciones instructivas y descriptivas se invirtió: de **0,3 a 1** a
**5,8 a 1**.

La incorporación obligó a resolver un problema de escala. Una de las fuentes nuevas tiene 289
páginas: como documento único habría vuelto el `hit@k` ininterpretable, y como fuente entera
habría dominado el corpus. La solución fue separar **fuente** de **documento** y dividir las
fuentes largas, de modo que todos los documentos tengan un tamaño comparable. El detalle del
procedimiento —y del intento fallido de dividir el libro por capítulos— está más abajo.

### Qué conservamos y qué descartamos

Las mediciones de la primera iteración quedaron obsoletas al cambiar el corpus, y el notebook no
las conserva: presentar cifras de una configuración como si fueran de otra sería engañoso. Lo que
sí se conserva es **todo lo que la primera iteración enseñó**, porque no depende del corpus:

- Los **cinco modos de fallo** identificados al leer las respuestas, que son los que los ejemplos
  del prompt few-shot atacan de forma directa.
- La decisión de evaluar con **dos conjuntos de preguntas** con funciones distintas, que es lo que
  permitió descubrir la carencia del corpus inicial.
- Las **métricas complementarias** —`precision@k` y `MRR`—, incorporadas al constatar que `hit@k`
  saturaba y dejaba de discriminar.
- Las **dos verificaciones automáticas** de anclaje y aporte del RAG, nacidas de comprobar que la
  anotación humana no distingue una respuesta genérica bien redactada de una fundada.
- El criterio para **descartar dos de las cuatro técnicas** de RAG avanzado consideradas.

Si tuviéramos que empezar de nuevo, armaríamos el conjunto de evaluación **antes** de fijar el
corpus. Habríamos detectado la carencia desde el inicio y nos habríamos ahorrado una iteración
completa. Esa es, probablemente, la lección más transferible del trabajo.

## Estructura del repositorio

```
.
├── README.md                        este archivo
├── TP_RAG_CabezaDeEquipo.ipynb      notebook único: todo el trabajo, de la Fase 0 al reporte
└── Corpus/                          documentos fuente en PDF
```

## Cómo ejecutar

1. Abrir el notebook en Google Colab.
2. `Entorno de ejecución → Cambiar tipo de entorno → T4 GPU`.
3. `Entorno de ejecución → Reiniciar y ejecutar todo`.

**Los PDFs se cargan solos.** La celda 1.2 prueba cuatro orígenes y usa el primero disponible:

| Orden | Origen | Cuándo aplica |
|---|---|---|
| 1 | Carpeta `Corpus/` del proyecto | Carpeta completa subida a Colab, o ejecución local |
| 2 | Clonado de este repositorio | Colab sin archivos: basta abrir el notebook y ejecutar |
| 3 | Google Drive | Si se copió la carpeta al Drive propio |
| 4 | Subida manual | Último recurso |

La búsqueda local no se limita al directorio actual: en Colab el directorio de trabajo es
`/content`, de modo que una carpeta subida entera queda en `/content/<nombre>/Corpus`. La celda
recorre hasta tres niveles de profundidad.

Como el repositorio es público y el corpus está versionado en él, el origen 2 hace que el trabajo
sea **reproducible por un tercero sin disponer de los archivos**: alcanza con abrir el notebook en
Colab y ejecutarlo.

Los modelos empleados son de acceso abierto y se descargan sin autenticación:
`sentence-transformers/paraphrase-multilingual-mpnet-base-v2` para embeddings,
`Qwen/Qwen2.5-1.5B-Instruct` para generación y `Qwen/Qwen2.5-3B-Instruct` como juez automático.

Las celdas de anotación humana vienen con los valores ya cargados, de modo que las tablas de
resultados se regeneran solas. La generación es determinística (`do_sample=False`), por lo que las
respuestas deberían reproducirse idénticas entre corridas.

## El corpus

Nueve fuentes en español de acceso abierto, expandidas en **26 documentos**: **895.185
caracteres** tras la limpieza, casi quince veces el mínimo exigido.

La composición es deliberadamente mixta: cinco artículos científicos aportan el registro empírico
y cuatro fuentes aportan el aplicado. Como se explica en la historia del proyecto, esa mezcla no
fue el punto de partida sino el resultado de una iteración.

### Las nueve fuentes

| Archivo | Referencia | Registro | Docs |
|---|---|---|---|
| `iberoam_intervencion.pdf` | Navarrón et al. (2017), Rev. Iberoamericana de Psicología del Ejercicio y el Deporte, 12(1) | Empírico | 1 |
| `cpd_burnout.pdf` | De Francisco et al. (2009), Cuadernos de Psicología del Deporte, 9(2) | Empírico | 1 |
| `anales_bayesiano.pdf` | Garcia-Mas et al. (2015), Anales de Psicología, 31(1) | Empírico | 1 |
| `sportis_motiv_ansied.pdf` | Arroyo del Bosque et al. (2025), Sportis, 11(3) | Empírico | 1 |
| `ariza_fisio_psico_futbol.pdf` | León Ariza et al. (2011), Cuerpo, Cultura y Movimiento, 1(2) | Empírico | 1 |
| `Consejos_psicologico-deportivos_para_el_futbolista.pdf` | Material divulgativo, 17 páginas | Aplicado | 1 |
| `Tema-4-Gestion-de-la-frustracion-...-Diego-Benito.pdf` | Benito, D. *Gestión de la frustración desde la psicología deportiva*, 29 páginas | Aplicado | 1 |
| `Documento_completo__.pdf` | Cornejo Zambrano (2012), *Intervención psicológica en futbolistas juveniles*. Tesis de Maestría, UNLP, 132 páginas | Mixto | 5 |
| `Libro-Ganar-Con-La-Cabeza-...pdf` | *Ganar con la Cabeza: guía completa del entrenamiento mental para el fútbol*, 289 páginas | Aplicado | 14 |

### División de las fuentes largas

La distinción entre **fuente** y **documento** importa: el documento es la unidad de trazabilidad
con la que se mide el `hit@k`, y una fuente de 289 páginas como documento único vuelve la métrica
ininterpretable.

La **tesis se divide por capítulos**, detectados en el texto descartando las páginas de índice
—una página con tres o más encabezados es índice, no cuerpo—. Resultan cinco documentos de 24.000
a 39.000 caracteres.

El **libro se divide en partes de tamaño parejo**, rotuladas por rango de páginas. Se intentó
primero por capítulos, pero el libro sólo imprime los encabezados en su índice y el mapeo a las
páginas del PDF resultó inconsistente, produciendo capítulos de 1.000 a 50.000 caracteres. La
división por tamaño es menos informativa pero reproducible y homogénea, que es lo que la métrica
necesita.

### Composición y limitación

| Registro | Documentos | Caracteres | Participación |
|---|---|---|---|
| Aplicado | 16 | 561.015 | 62,7% |
| Empírico | 5 | 176.872 | 19,8% |
| Mixto | 5 | 157.298 | 17,6% |

El libro aporta por sí solo el 52,4% de los caracteres y la tesis otro 17,6%. Dividirlos resuelve
la trazabilidad pero no la participación: una consulta cualquiera tiene mayor probabilidad a
priori de recuperar material del libro simplemente porque hay más. El baseline aleatorio del
notebook cuantifica ese efecto, y las métricas de recuperación deben leerse contra él.

## Organización del notebook

| Fase | Contenido |
|---|---|
| 0 | Instalación y configuración |
| 1 | Corpus: carga, extracción, limpieza, estadísticas y dos estrategias de chunking |
| 2 | Embeddings e índice vectorial FAISS |
| 3 | Evaluación del retrieval: `hit@k`, `precision@k` y `MRR` |
| 4 | Generación: comparación CON RAG vs SIN RAG, con dos conjuntos de preguntas |
| 5 | Iteración: correcciones y técnicas de RAG avanzado |
| 6 | Evaluación a escala: conjunto ampliado, inferencia estadística y barrido de hiperparámetros |
| 7 | Reporte final |

## Resultados

Las mediciones sobre el corpus actual se obtienen ejecutando el notebook; las tablas quedan en las
Fases 3, 4, 5 y 6, y el análisis en la Fase 7. La sección "Historia del proyecto" explica por qué
no se conservan las cifras de la configuración anterior.

Las conclusiones **metodológicas**, en cambio, no dependen del corpus y se sostienen:

- **`hit@k` satura y deja de discriminar** en corpus con pocos documentos, lo que motivó
  incorporar `precision@k` y `MRR`.
- **La anotación humana tiene puntos ciegos.** No distingue los aciertos accidentales —respuestas
  correctas que el modelo habría dado igual sin contexto—, de ahí las dos verificaciones
  automáticas de anclaje y aporte del RAG.
- **Un corpus puede estar bien construido y aun así no servir** para las preguntas que formularía
  el usuario real. Es el hallazgo que motivó la segunda iteración.
- **Diez preguntas no alcanzan** para distinguir una diferencia real del ruido de medición, lo que
  motivó la ampliación del conjunto de evaluación en la Fase 6.

## Metodología de evaluación

El trabajo trata el tamaño del conjunto de evaluación como un problema de medición, no de
presentación. Con las diez preguntas iniciales, los intervalos de confianza del 95% sobre cada
tasa miden entre 39 y 52 puntos de ancho, y el test exacto de McNemar sobre la comparación entre
estrategias de chunking arroja **p = 0,50**: la diferencia observada entre 90% y 70% de `hit@1`
no es distinguible del azar con esa muestra.

La Fase 6 aborda esa limitación con tres instrumentos:

**Conjunto ampliado por generación asistida.** Se muestrean chunks de forma estratificada por
documento, priorizando los que contienen cifras, y se solicita al modelo un par pregunta-respuesta
por fragmento. Cada candidata pasa luego por verificación humana. El procedimiento tiene una
ventaja metodológica sobre la redacción manual: la pregunta queda anotada con su documento de
origen **por construcción**, lo que elimina la fuente de error más frecuente en la anotación.

**Inferencia estadística.** Cada tasa se reporta con su intervalo de Wilson —apropiado para
proporciones con muestras pequeñas— y las comparaciones entre sistemas se evalúan con el test
exacto de McNemar, que es el adecuado cuando ambos se miden sobre los mismos ítems.

**Baseline aleatorio.** Se calcula qué `hit@k` obtendría un recuperador que eligiera chunks al
azar: cerca del 20% en `hit@1` y del 66% en `hit@5` sobre este corpus. Sin esa referencia, un
`hit@5` del 100% sugiere un logro mayor del real; con ella queda claro que el aporte del sistema
se concentra en la precisión del ranking, no en la cobertura.

**Barrido de tamaño de chunk.** Las dos estrategias se comparan a 400, 800 y 1200 caracteres
sobre el mismo conjunto de preguntas. La comparación de la Fase 3 se hizo a un único tamaño, lo
que dejaba abierta la posibilidad de que la ventaja de paragraph-aware fuera un artefacto de esa
elección y no una propiedad de la estrategia. El barrido resuelve esa ambigüedad: si la ventaja se
sostiene en los tres tamaños es atribuible a la estrategia; si se invierte en alguno, no lo es.

**Escalado condicionado de la anotación.** El juez automático sólo se emplea para anotar el
conjunto ampliado si su kappa frente al patrón humano supera 0,40. Por debajo de ese umbral las
métricas de generación se mantienen sobre las preguntas anotadas a mano y se declara la
limitación; las métricas de recuperación no se ven afectadas, porque no dependen del juez. Ese es
el motivo por el cual el juez se valida antes de usarlo y no después.

## Técnicas de RAG avanzado: criterio de selección

Se evaluaron cuatro técnicas. Se incorporaron dos y se descartaron dos. **El criterio no fue el
prestigio de cada técnica sino su correspondencia con el diagnóstico del análisis de errores**, y
la decisión de descartar se documenta con el mismo detalle que la de incorporar.

El punto de partida es el diagnóstico de la primera iteración: la recuperación operaba cerca de
su techo y los problemas estaban localizados en la generación y en la evaluación. Dos de las
cuatro técnicas actúan sobre la recuperación, que es la etapa que mejor funcionaba.

### Incorporadas

**Few-shot prompting** *(sección 5.6)*

- *Por qué se quiso usar:* es la única de las cuatro que opera sobre la generación, que es donde
  el análisis de errores ubica el cuello de botella.
- *Qué ataca:* los tres ejemplos del prompt están construidos sobre fallos efectivamente
  observados — contaminación entre documentos (P08), falso negativo del grounding (P09) e
  inversión de atributos (P01).
- *Cómo se mide:* se genera un segundo conjunto de respuestas sobre las mismas diez preguntas,
  dejando el prompt como única variable, y se comparan las tasas C/P/I/A.
- *Detalle de diseño:* los ejemplos usan datos ficticios sobre atletismo, ajenos al corpus. Con
  datos reales, el modelo podría reproducirlos como respuesta ante preguntas parecidas e inducir
  el error que se busca evitar.

**LLM-as-judge** *(sección 5.7)*

- *Por qué se quiso usar:* la anotación manual es costosa y tiene un punto ciego demostrado — no
  distingue los aciertos accidentales, respuestas correctas que el modelo habría dado igual sin
  contexto.
- *Qué permite:* este trabajo dispone de 36 respuestas anotadas a mano, que funcionan como patrón
  de referencia. La pregunta no es si el juez funciona, sino cuánto coincide con el criterio humano.
- *Cómo se mide:* porcentaje de coincidencia exacta y kappa de Cohen, que descuenta el acuerdo
  atribuible al azar, más la matriz de confusión para localizar dónde difieren.
- *Limitación declarada:* se emplea un modelo distinto y mayor que el generador para evitar sesgo
  de autoevaluación. Si la memoria de la GPU no alcanza, el notebook recurre al generador y lo
  reporta; en ese caso el acuerdo obtenido debe leerse como cota superior.

### Descartadas

**Query rewriting**

- *Por qué se quiso usar:* reformula la consulta para acercarla al vocabulario del corpus. El
  segundo conjunto de preguntas está en registro coloquial frente a un corpus académico, que es
  exactamente el escenario que la técnica contempla.
- *Por qué se descartó:* sobre el conjunto anotado el retrieval ya operaba cerca de su techo, de
  modo que la técnica no podía producir un efecto medible. Sobre el conjunto de preguntas
  reales, donde sí tendría sentido, no existe métrica de recuperación, precisamente porque esas
  preguntas no tienen un documento correcto único. Una técnica cuyo efecto no puede medirse con el
  instrumental disponible no se incorpora.

**HyDE (Hypothetical Document Embeddings)**

- *Por qué se quiso usar:* genera una respuesta hipotética y busca con su vector, salvando la
  asimetría entre una pregunta breve y documentos extensos y técnicos. Es la configuración de este
  trabajo.
- *Por qué se descartó:* además del mismo problema de techo, HyDE depende de la calidad de la
  respuesta hipotética, y este generador alucina **aun disponiendo del contexto correcto**. Una respuesta hipotética inventada produce un vector de consulta que apunta
  en la dirección equivocada y degrada una recuperación que hoy funciona. El valor esperado de
  aplicarla es negativo.

## Limitaciones

- **Tamaño del conjunto de evaluación.** Se amplió mediante generación asistida (Fase 6), pero
  una parte de las anotaciones del conjunto ampliado proviene del juez automático y no de
  anotación humana. El reporte declara esa composición: no todas las anotaciones tienen la misma
  procedencia ni la misma fiabilidad.
- **Un solo modelo de embeddings.** La Fase 2 emplea el modelo sugerido por el enunciado sin
  comparar alternativas.
- **Tamaño del modelo generador.** Qwen2.5-1.5B fue elegido por caber en la GPU gratuita de Colab.
  Los errores de inversión de atributos y de invención bajo densidad numérica son fallos de
  comprensión lectora que un modelo mayor probablemente evitaría.
- **Participación desigual de las fuentes.** El libro aporta más de la mitad de los caracteres del
  corpus. La división en documentos homogéneos resuelve la trazabilidad, pero aislar el efecto del
  registro del efecto del volumen requeriría acotar la cantidad de material por fuente.
- **División del libro por tamaño y no por capítulos**, por las razones expuestas más arriba. Una
  segmentación semántica daría documentos más interpretables.
