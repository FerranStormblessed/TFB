# <h1 style="font-size: 50px;"><i>Chatbot</i> de la <i>Coppermind</i>, una <i>wikipedia</i> del <i>Cosmere</i></h1>  

# Introducción  

# Objetivos  

# Desarrollo

## I. Creación de la base de datos 

<p style="text-align: justify;">
En primer lugar, he elegido la lista de entradas de la <i>wikipedia</i> que quiero utilizar como base de datos. Me he decantado por centrarme no en una saga en concreto, sino en los conceptos que son comunes a toda saga, a todo el universo del <i>Cosmere</i>.  

Adjunto la [_lista_](https://drive.google.com/file/d/1-2jTNQljoHThvsco3jfIMhAOamoCOCqM/view?usp=sharing) con las urls de las entradas elegidas. He hecho un script que descarga el html a partir de la lista de urls. La descarga de cada entrada, al ser de una web, ha sido en formato html.   
<span style="color: red;">(Script en Creación de base de datos-TFB.ipynb en local)</span>
</p>
  
### I.II. Tratamiento de los htmls  

<p style="text-align: justify;">
Para optimizar el RAG he convertido los htmls en markdown. Además, he hecho cierto procesamiento para eliminar varias cosas:  

* Eliminación de vínculos a otras entradas de la <i>wikipedia</i>.  
* Eliminación de citas.  
* Eliminación de imágenes.  
* Eliminación de la sección de notas en adelante (sección donde se ponen todas las referencias, no aporta valor).  

De momento no voy a hacer más tratamiento al texto. Más adelante, con la ayuda del Test Automático, estudiaré si sería conveniente.

<span style="color: red;">(Script en html_to_markdown.ipynb en local, luego subir markdowns al drive)</span>  
</p>

### I.III. Embeddings  

<p style="text-align: justify;">
Una vez procesado el contenido de cada entrada, he dividido el texto completo de la base de datos en <i>chunks</i> con un <i>overlap</i> del 20% (el tamaño del <i>chunk</i> está por ver, a estudiar con el Test Automático). He hecho los <i>embeddings</i> con el modelo <i>paraphrase-MiniLM-L6-v2</i>, un modelo de transformers que es relativamente compacto y eficiente cuya principal aplicaciión es la búsqueda semántica. He obtenido el modelo de <i>Hugging Face</i>.  

El código divide el texto en <i>chunks</i>, hace los <i>embeddings</i> de cada <i>chunk</i> y guarda en <i>Google Drive</i> cada <i>chunk</i> y <i>embedding</i> con varios metadatos asociados: el número de <i>chunk</i> (entre todos los <i>chunks</i> del documento), el número de documento o <i>DOCID</i> (número de entrada de las totales) y su nombre (la <i>url</i> asociada, o algo similar).  

<span style="color: red;">(Script en Creación de la base de datos.ipynb, Google Colab)</span>
</p>

## II. Funcionamiento del <i>RAG</i>

<p style="text-align: justify;">
Un modelo <i>RAG</i> (<i>Retrieval-Augmented Generation</i>) consiste en dos partes principales: búsqueda semántica y generación de la respuesta. La búsqueda semántica consiste en hallar un cierto número de <i>chunks</i> de la base de datos que tienen mayor similitud con la pregunta del usuario. Una vez hallados, se le pide a un <i>LLM</i> componer una respuesta en base a esa pregunta y los chunks encontrados.  
</p>

### II.I. Búsqueda semántica  

<p style="text-align: justify;">
Una vez hechos y guardados los <i>embeddings</i>, podemos acceder a ellos mediante <i>Google Drive</i>. Al hacer una pregunta, se hace el <i>embedding</i> de esa pregunta y se calcula la similitud entre esa pregunta y todos los <i>embeddings</i> de la base de datos. Otro de los hiperparámetros, junto con el tamaño de <i>chunk</i> y el <i>overlap</i>, es <i>top-n</i>, el número de chunks con más similitud que nos quedamos. El valor óptimo de este hiperparámetro también lo decidiremos con la ayuda del Test Automático.  

Al hacer una pregunta, obtenemos los <i>chunks</i> con mayor similitud, el número de <i>chunk</i>, el <i>DOCID</i>, su nombre y la similitud entre ese <i>chunk</i> y la pregunta. Quedaría esto:

> Question: ¿Se puede leer algún libro del Cosmere sin haber leído otras sagas?
> The 3 chunks most similar to your question are:
> 1. DOCID: 12 | Document Name: https://es.coppermind.net/wiki/Cosmere | Chunk number: 3 | Similarity: 0.6855  
> subyacentes, apareciendo algunos personajes en otros mundos ajenos al suyo. A pesar de las conexiones, Brandon ha dejado claro que uno no necesita ningún conocimiento del Cosmere en general para leer, entender, o disfrutar de los libros que tienen lugar en él. Las secuencia principal del Cosmere consistirá en la saga *Dragonsteel*"), la trilogía de *Elantris*, al menos cuatro eras de la saga *Nacidos de la bruma*") y *El archivo de las tormentas*. La historia del Cosmere no incluye ningún libro que haga referencia a la Tierra, puesto que la tierra no está en el Cosmere. Para una lista completa
> 
> 2. DOCID: 19 | Document Name: https://es.coppermind.net/wiki/Esquirla del Amanecer | Chunk number: 49 | Similarity: 0.6744  
no se refiera a Sigzil. En *Viento y verdad*, se confirmó que Hoid tuvo en su poder la Esquirla del Amanecer Existe durante los sucesos de los libros 1-5 de *El archivo de las tormentas*. ## Notes 1. ↑ a b c d e f g h i j k l m n o p q r s t u *Esquirla del Amanecer (novella)* capítulo 19")Summary: Esquirla del Amanecer (novella)/Chapter\_19/Chapter 19 (la página no existe)")#/Chapter 19") 2. ↑ a b General Reddit 2022 — Arcanum - 2022-12-02Cite: Arcanum-15961# 3. ↑ a b c Dawnshard Annotations Reddit Q&A — Arcanum -
>
> 3. DOCID: 24 | Document Name: https://es.coppermind.net/wiki/Hoid | Chunk number: 184 | Similarity: 0.6562  
qué libro fue eso respondió *Brazales de Duelo")*. * La misión de Hoid quizás sea: «hacer aquello que una vez fue». * Hoid no está impresionado por los Sangre Espectral. * Hoid detesta al Grupo, y los miembros de este último que le conocen también le detestan. * Aunque una vez le dijo a Kaladin (bastante acertado) que su vida comenzó como palabras en una página, este hecho no tenía la intención de romper la cuarta pared. * Hoid se ha travestido en el pasado, «muchas veces». * Antes de los eventos de *Palabras Radiantes*, a Hoid no le habían

Además, se puede obtener una gráfica interesante: el histograma de la similitud entre cada <i>chunk</i> de la base de datos y la pregunta.  

![](Images/distribucion-similitudes.png){ width=50%, align=center }

<span style="color: red;">(Script en Creación de la base de datos.ipynb, Google Colab)</span>
</p>

## II.II. Generación de la respuesta

<p style="text-align: justify;">
Esos <i>chunks</i> con mayor similitud con la pregunta se le pasan, junto con la pregunta del usuario, a un <i>LLM</i>, pidiéndole que responda a la pregunta del usuario en base a los <i>chunks</i> encontrados. 

El modelo seleecionado se decidirá más adelante a raíz de una serie de tests. Para esta y posteriores llamadas a <i>LLMs</i> se utilizarán varios proveedores que permiten hacer llamadas gratis vía API (ver [_LLMs_free_API_keys.ipynb_](https://drive.google.com/file/d/1F9DaZLL1n1gUDcGtaeGBuQ-fTJLYN5xE/view?usp=sharing) para más detalles).  

El <i>system prompt</i> es el siguiente:  

> Eres un asistente virtual experto en responder preguntas. A continuación vas a recibir la pregunta de un usuario y el conocimiento que debes utilizar para responderla en el siguiente formato:
> 
> Pregunta: Pregunta del usuario
> Conocimiento:
> Nombre del documento 1: contenido del documento 1
> Nombre del documento 2: contenido del documento 2
> ...
> Nombre del documento N: contenido del documento N
> 
> Responde como si fueras un chatbot de una wikipedia. Después de dar tu respuesta completa, di el nombre de los documentos en los que te has basado para elaborarla. Si te has basado dos veces en el mismo documento, no lo repitas al referenciarlo. Hazlo en este formato:
> 
> Pon aquí tu respuesta
> [[Pon aquí únicamente el nombre del primer documento en el que te has basado]]
> [[Pon aquí únicamente el nombre del segundo documento en el que te has basado, siempre y cuando no hayas puesto el mismo nombre antes]]
> ...
> 
> No utilices conocimiento propio de tu entrenamiento, utiliza solo el que se te proporciona. Si parte del conocimiento que se te proporciona no te sirve para responder a la pregunta, no lo utilices. Cita únicamente los documentos en los que te has basado para elaborar la respuesta. Si con el conocimiento que se te proporciona no puedes responder a la pregunta, responde únicamente "Lo siento, no puedo responderte a esa pregunta" y no cites ningún documento.

El <i>user prompt</i> es el siguiente:  

> Pregunta: {Pregunta del usuario}  
> 
> Conocimiento:  
> {Nombre del documento del chunk}: {contenido del chunk}

Como se puede observar, no solo se le pide al <i>LLM</i> que componga la respuesta, sino que cite sus fuentes. De esta forma el usuario puede, con solo pinchar en el nombre de la referencia, acceder a dicha entrada de la <i>wikipedia</i> mediante la <i>url</i>.
</p>

## III. Test Automático  

<p style="text-align: justify;">
Con el objetivo de poder valorar si los cambios tienen un impacto positivo en el modelo, he creado un test de regresión, al que llamaré Test Automático. Este test consiste en 137 preguntas de las que sé la respuesta correcta, a la que llamaré respuesta <i>best</i>, y la entrada (o entradas) de la wikipedia donde se responde a esa pregunta, que llamaré documento <i>best</i>. El test consistirá en 3 subtests:  

* OK/KO RAG: Porcentaje de preguntas en las que el documento/s <i>best</i> se encuentra entre los encontrados por la búsqueda semántica.  

* OK/KO <i>LLM</i>: Porcentaje de preguntas en las que el documento/s <i>best</i> se encuentra entre los elegidos por el <i>LLM</i> para redactar la respuesta.  

* OK/KO <i>LLM as a judge</i>: Porcentaje de preguntas en las que un segundo <i>LLM</i> valora que la respuesta del primer modelo se ajusta a la respuesta <i>best</i>. El LLM elegido para el test de <i>LLM as a judge</i> es el <i>Llama 3.3 70B versatile</i>, ya que es un <i>LLM</i> grande, la útima versión de los modelos <i>LLama</i> y apto para gran variedad de tareas.   

<span style="color: red;">(Script en Test Automático.ipynb, Google Colab)</span>

El <i>system prompt</i> es el siguiente:  

> Vas a recibir una pregunta de un usuario (Pregunta), la respuesta correcta a esta pregunta (Respuesta_Best) y una respuesta generada (Respuesta_Generada). Tus objetivos son los siguientes:
> 1. Determinar si la Respuesta_Generada responde a la Pregunta.
> 2. Determinar si la Respuesta_Generada concuerda con la Respuesta_Best y no la contradice.
> Tienes que hacer una valoración en detalle y razonando sobre tu valoración. Si la Respuesta_Generada no responde a la Pregunta, valorar como KO. Si la Respuesta_Generada concuerda con la Respuesta_Best y no la contradice, valorarla como OK. Si la Respuesta_Generada no concuerda con la Respuesta_Best y la contradice, valorarla como KO. No tener en cuenta posibles detalles adicionales que puedan estar incluidos en la Respuesta_Generada, siempre y cuando no contradigan la Respuesta_Best.
> 
> Una vez hayas hecho tu razonamiento, cúentalo, y al final pon tu valoración en este formato:
> [[Valoración: OK/KO]]

El <i>user prompt</i> es el siguiente:  

> Pregunta:  
> {Pregunta del usuario}  
> 
> Respuesta_Best:  
> {Respuesta marcada como correcta}  
> 
> Respuesta_Generada:  
> {Respuesta generada por el <i>LLM</i>}



<span style="color: red;">EXPLICAR, DECIR LOS % OK, LLM AS A JUDGE?, ENLACE AL GS Y A UN EJEMPLO DEL GS EJECUTADO, ENLACE A EXCEL DE RESULTADOS, ETC. </span> 

Las preguntas, documento/s <i>best</i> y respuestas <i>best</i> se pueden ver aquí: [_Input Test Automático.xlsx_](https://docs.google.com/spreadsheets/d/1gs--ymPUeTYjbyf6AwcPHpowz6-6ehQb/edit?usp=sharing&ouid=117815217117454739708&rtpof=true&sd=true).
</p>


### III.I. <i>chunk size</i>, <i>overlap</i> y <i>top n</i>

<p style="text-align: justify;">
El primer objetivo de este test es determinar el tamaño óptimo de los <i>chunks</i> de la base de datos. Para eso se ha ejecutado el test con varios <i>chunk size</i> distintos, así como para varios <i>top n</i> (el número de <i>chunks</i> pasados al <i>LLM</i> para redactar la respuesta). En este caso se ha fijado el <i>LLM</i> de elaboración de la respuesta y varios de sus parámetros. El <i>LLM</i> ha sido <i>Google Gemini 2.0 pro experimental</i>, aunque más adelante se comaprarán varios modelos y se eligirá el mejor. La temperatura se ha fijado a 0 para aumentar la reproducibilidad de los tests y reducir su variabilidad. Además, se ha fijado la <i>repetition penalty</i> a 0 para intentar producir respuestas concisas. Por último, el <i>chunk overlap</i> se ha fijado por defecto al 20%.  

Se han hecho estas pruebas para un <i>chunk size</i> de 50, 75, 100 y 200 tokens, y <i>top n</i> de 3, 5, 10 y 15 <i>chunks</i>. El resultado en detalle de las pruebas puede verse aquí: [_Chunk sze-overlap-topn.xlsx_](https://docs.google.com/spreadsheets/d/1rY1Kpd-hpIVb6air6aGdrMKY75ROrlt2/edit?usp=sharing&ouid=117815217117454739708&rtpof=true&sd=true).  
</p>

| Chunk_size | top_n | OK RAG | OK FUENTES | OK RESPUESTA |
|------------|-------|--------|------------|--------------|
| 50         | 3     | 68,38% | 55,15%     | 47,06%       |
|            | 5     | 78,68% | 61,76%     | 55,15%       |
|            | 10    | 87,50% | 72,79%     | 66,18%       |
|            | 15    | 90,44% | 74,26%     | 69,85%       |
| 75         | 3     | 63,97% | 48,53%     | 42,22%       |
|            | 5     | 72,79% | 55,88%     | 55,88%       |
|            | 10    | 83,09% | 63,97%     | 63,97%       |
|            | 15    | 86,76% | 69,12%     | 69,85%       |
| **100**        | 3     | 66,18% | 51,47%     | 44,85%       |
|            | 5     | 75,74% | 60,29%     | 52,94%       |
|            | **10**    | **84,56%** | **66,91%**     | **66,18%**       |
|            | 15    | 88,24% | 69,85%     | 72,79%       |
| 200        | 3     | 61,03% | 47,06%     | 44,12%       |
|            | 5     | 65,44% | 53,68%     | 52,21%       |
|            | 10    | 74,26% | 53,68%     | 56,62%       |
|            | 15    | 79,41% | 58,09%     | 59,56%       |

![](Images/chunksize-topn.jpg){ width=50%, align=center }

<p style="text-align: justify;">
A raíz de estos resultados se utilizará un <i>chunk size</i> de 100 <i>tokens</i> (por tanto, un <i>chunk overlap</i> de 20 <i>tokens</i>) y un <i>top n</i> de 10 <i>chunks</i>. Se escoge esto por varias razones. En primer lugar, es el que mayor porcentaje de OK arroja en OK de la respuesta para <i>top n</i> de 10, junto a un <i>chunk size</i> de 50. Se elige sobre este porque, para resultados iguales, un <i>chunk size</i> de 100 ofrece más contexto. No se escoge <i>chunk size</i> de 100 y <i>top n</i> de 15 porque la mejora en esta métrica no es demasiada. Además, hay que tener en cuenta el tiempo de respuesta del modelo, una métrica que en este caso no se ha evaludao, pero que es fundamental, ya que esta aplicación de IA es un <i>chatbot</i>.  
En resumen, los parámetros elegidos han sido <b><i>chunk size</i> = 100 <i>tokens</i>, <i>chunk overlap</i> = 20 <i>tokens</i>, <i>top n</i> = 10 <i>chunks</i></b>.  

Por otro lado, como más adelante se va a analizar el <i>LLM</i> a utilizar para responder a las preguntas, así como su temperatura, cabría preguntarse si este análisis fijando el <i>LLM</i> es válido para otros. La realidad es que no, pero se ha hecho así para reducir el número de pruebas a hacer. Aunque los resultados del análisis del <i>chunk size</i>, <i>overlap</i> y <i>top n</i> probablemente cambien de un <i>LLM</i> a otro, se ha supuesto que no serán cambios significativos.  
</p>

### III.II. Modelo de <i>embeddings</i>  

<p style="text-align: justify;">
En el momento que se hicieron los tests III.I pensaba que el modelo de <i>embeddings</i> que estaba utilizando, <i>paraphrase-MiniLM-L6-v2</i>, era el mejor. Sin embargo, en una de las clases me hicieron saber que este modelo no está entrenado en español, por lo que es fundamental encontrar uno que funcione mejor entrenado específicamente en español.  

Lo ideal sería hacer primero este test y después el III.I, ya que es más determinante el modelo de <i>embeddings</i> utilizado. Sin embargo, como el test III.I consume mucho tiempo, asumiremos el error producido por hacerlo en este orden.  

Los modelos de <i>embeddings</i> que vamos a comparar son los siguientes:  

* <i>paraphrase-MiniLM-L6-v2</i> (as-is, entrenado solo en inglés)
* <i>paraphrase-multilingual-MiniLM-L12-v2</i>  
* <i>paraphrase-multilingual-mpnet-base-v2</i> 
* <i>distiluse-base-multilingual-cased-v2</i> 
* <i>stsb-xlm-r-multilingual</i> 
* <i>finetuned_sentence_similarity_spanish</i> 

Por desgracia, y debido a las limitaciones de tener que usar llamadas gratis via API a <i>LLMs</i> ofrecidos por distintos proveedores, el <i>LLM</i> de elaboración de la respuesta que usé en el test III.I, <i>Google: Gemini Pro 2.0 Experimental (free)</i>, ya no está disponible. Debido a esto voy a tener que utilizar otro, <i>Google: Gemini 2.0 Flash Thinking Experimental 01-21 (free)</i>. Aún así, esto no invalida las conclusiones de este test.
</p>




### III.III. <i>LLM</i> de generación de la respuesta  

<p style="text-align: justify;">
A continuación, se evaluará qué <i>LLM</i> se utilizará para generar la respuesta a partir del conocimiento encontrado. Para ello, se evaluarán las tres métricas anteriores (OK RAG, OK FUENTES y OK RESPUESTA), además del tiempo de respuesta de cada modelo.
</p>









# Puesta en producción

# Posibles mejoras


# Líneas a futuro

* Crear un agente que haga búsquedas y decida cuántas hacer, el top-n y cuándo parar (así se podría hacer un chatbot de verdad).
* Usar structured outputs para las citas.
* Creación de una interfaz de chat estilo <i>chatgpt</i>.  
* Aplicación de un filtro de similitud. Por ejemplo, si no hay ningún chunk con similitud mayor que 0.6, responder "Lo siento, no tengo datos suficientes para responder a tu pregunta".
* Detector de <i>chit-chat</i>.  
* Implementación real del *chat* (que no sea únicamente pregunta-respuesta aisladas, que recoja las preguntas anteriores de ese chat).  
* <i>RAG-fusion</i>: creación de preguntas de apoyo mediante un <i>LLM</i>.  
* Implementación de <i>Groundedness</i> para medir alucinaciones y cómo de desviada está una respuesta de los <i>chunks</i> encontrados. Muestra al usuario para prevenir?