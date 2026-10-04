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

Cinco artículos científicos en español, de acceso abierto. **176.872 caracteres** tras la limpieza
—2,9 veces el mínimo exigido— distribuidos en cinco documentos.

La carpeta `Corpus/` contiene **nueve PDFs**: cinco integran el corpus y cuatro fueron evaluados
y no incorporados.

### Los cinco documentos del corpus

| Archivo | Referencia | Eje |
|---|---|---|
| `iberoam_intervencion.pdf` | Navarrón et al. (2017), Rev. Iberoamericana de Psicología del Ejercicio y el Deporte, 12(1) | Habilidades psicológicas |
| `cpd_burnout.pdf` | De Francisco et al. (2009), Cuadernos de Psicología del Deporte, 9(2) | Burnout |
| `anales_bayesiano.pdf` | Garcia-Mas et al. (2015), Anales de Psicología, 31(1) | Motivación y clima motivacional |
| `sportis_motiv_ansied.pdf` | Arroyo del Bosque et al. (2025), Sportis, 11(3) | Ansiedad precompetitiva |
| `ariza_fisio_psico_futbol.pdf` | León Ariza et al. (2011), Cuerpo, Cultura y Movimiento, 1(2) | Demandas de la competencia |

### Los cuatro documentos evaluados y no incorporados

Las cifras corresponden al texto extraído antes de la limpieza.

| Archivo | Qué es | Caracteres | Instructivas / descriptivas |
|---|---|---|---|
| `Libro-Ganar-Con-La-Cabeza-...pdf` | *Ganar con la Cabeza: una guía completa del entrenamiento mental para el fútbol*. Libro, 289 páginas | 478.750 | 839 / 13 |
| `Documento_completo__.pdf` | Cornejo Zambrano (2012), *Intervención psicológica en futbolistas juveniles*. Tesis de Maestría en Deportes, UNLP, 132 páginas | 187.254 | 71 / 43 |
| `Consejos_psicologico-deportivos_para_el_futbolista.pdf` | Material divulgativo, 17 páginas | 54.779 | 88 / 5 |
| `Tema-4-Gestion-de-la-frustracion-...-Diego-Benito.pdf` | Material formativo sobre gestión de la frustración, 29 páginas | 42.412 | 16 / 2 |

Se conservan en el repositorio porque documentan una de las conclusiones del trabajo.

**Por qué se los consideró.** El análisis mostró que el corpus de artículos científicos responde
bien las preguntas *sobre los estudios* y mal las preguntas *sobre qué hacer*: en el corpus la
proporción de construcciones descriptivas frente a instructivas es de 3,6 a 1. Estos cuatro
documentos presentan el perfil inverso y cubrirían esa carencia.

**Por qué no se incorporaron.** Por dos razones de distinto peso. La primera afecta a los cuatro:
sumarlos obliga a repetir la evaluación completa, porque las métricas reportadas dejarían de ser
comparables, y la anotación correspondiente excede el alcance de esta entrega. La segunda afecta
al libro y a la tesis: su tamaño desbalancea el corpus —sumados los cuatro, el material nuevo
sería el 81% del total y el libro por sí solo el 51%—, de modo que los cinco artículos quedarían
diluidos. Incorporarlos con rigor exige además dividir el libro por capítulos, ya que un documento
de 289 páginas como unidad única vuelve ininterpretable el `hit@k`.

Ambas cuestiones quedan planteadas como trabajo futuro en el reporte.

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

## Resultados principales

**El retrieval funciona; el generador es el cuello de botella.** Con `hit@1` del 90%, `MRR 0,95` y
`precision@4` de 0,75, el modelo recibe el pasaje correcto en primera posición y con poco ruido, y
aun así falla en el 60% de las preguntas. El límite lo impone la comprensión lectora de un modelo
de 1.5B parámetros, no la recuperación.

**El RAG reduce la invención.** Las respuestas correctas pasan del 10% al 40% y las alucinaciones
del 60% al 20%.

**El sistema opera en dos regímenes según el tipo de pregunta.** Sobre preguntas de dato específico
el RAG mejora sustancialmente los resultados; sobre preguntas abiertas de consejo —las que un
jugador real formularía— no aporta nada: las respuestas con y sin contexto resultan equivalentes,
porque el corpus no cubre esos temas.

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

El punto de partida es que este sistema tiene la recuperación cerca del techo (`hit@1` 90%,
`MRR 0,95`) y los problemas localizados en la generación y en la evaluación.

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
- *Por qué se descartó:* sobre el conjunto anotado el margen de mejora es de cinco puntos —`hit@1`
  ya está en 90%—, de modo que no puede producir un efecto medible. Sobre el conjunto de preguntas
  reales, donde sí tendría sentido, no existe métrica de recuperación, precisamente porque esas
  preguntas no tienen un documento correcto único. Una técnica cuyo efecto no puede medirse con el
  instrumental disponible no se incorpora.

**HyDE (Hypothetical Document Embeddings)**

- *Por qué se quiso usar:* genera una respuesta hipotética y busca con su vector, salvando la
  asimetría entre una pregunta breve y documentos extensos y técnicos. Es la configuración de este
  trabajo.
- *Por qué se descartó:* además del mismo problema de techo, HyDE depende de la calidad de la
  respuesta hipotética, y este generador alucina en el 20% de los casos **aun disponiendo del
  contexto correcto**. Una respuesta hipotética inventada produce un vector de consulta que apunta
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
- **Cobertura del corpus.** Los artículos científicos no responden las preguntas aplicadas que
  formularía el usuario real del asistente, como se detalla más arriba.
