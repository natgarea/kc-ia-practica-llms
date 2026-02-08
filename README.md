# Práctica LLMs: Donor Copilot

Este proyecto es un recomendador de organizaciones benéficas basado en **Retrieval Augmented Generation (RAG)**. Mi objetivo fue crear un asesor que ayude a potenciales donantes a encontrar ONGs fiables basándose en evidencia real y datos verificables.

## Índice

* [Estructura del repositorio](#estructura-del-repositorio)
* [Cómo ejecutar](#cómo-ejecutar)
* [Donor Copilot](#donor-copilot-recomendador-de-organizaciones-benéficas-con-retrieval-augmented-generation-rag)
    * [Problema](#el-problema-que-quiero-resolver)
    * [Alcance](#alcance-del-proyecto)
    * [Dataset y fuentes](#dataset-y-fuentes)
    * [Desarrollo del sistema RAG](#desarrollo-del-sistema-rag)
    * [Conclusiones](#conclusiones)

---

## Estructura del repositorio

He organizado el trabajo en cuatro notebooks principales:

- `01_dataset.ipynb` – Exploración del dataset.
- `02_indexing.ipynb` – Creación del índice vectorial.
- `03_rag_querying.ipynb` – Ejecución de consultas RAG.
- `04_evaluation.ipynb` – Evaluación del sistema.
- `data/` – Contiene los archivos CSV con organizaciones y los resultados de la evaluación.
- `presentacion.pptx` – Contiene una presentación que resume este README.

---

## Cómo ejecutar
Diseñé este proyecto para ejecutarse en local mediante notebooks.
⚠️ Usar el LLM local sin GPU, tanto el querying como la evaluación puede ser extremadamente lento.

1. **Crear el entorno virtual:**
```bash
python3 -m venv llms
source llms/bin/activate
```

2. **Instalar requisitos:**
```bash
pip install -r requirements.txt
```

3. **Lanzar JupyterLab**
```bash
jupyter lab
```

Los notebooks se deben ejecutar en orden.

---

## Donor Copilot: recomendador de organizaciones benéficas con Retrieval Augmented Generation (RAG)

### El problema que quiero resolver
Cuando uno quiere donar a una organización benéfica, se puede preguntar si realmente es una organización fiable, si hace un uso eficiente de los recursos y en qué medida sus programas alcanzan realmente sus objetivos. Para resolver esto, existen organizaciones que investigan y puntúan estas entidades, como:
* [GiveWell](https://www.givewell.org/)
* [The Life You Can Save](https://www.thelifeyoucansave.org/)
* [Founders Pledge](https://www.founderspledge.com/)
* [Evidence Action](https://www.evidenceaction.org/)

Mi objetivo con este proyecto es construir un **sistema RAG (Retrieval Augmented Generation)** que actúe como un *asesor*, capaz de recomendar organizaciones benéficas basándose en fuentes verificables, citando la evidencia utilizada y rechazando responder cuando no hay información suficiente.

He elegido este tema porque he estado leyendo estas semanas el libro *Moral Ambition* de Rutger Bregman y hace tiempo hice un proyecto final para un bootcamp de desarrollo sobre conectar personas con ONGs relacionadas con sus intereses: [causaimpacto](https://github.com/natgarea/causaimpacto).

---

### Alcance del proyecto
Este proyecto es una prueba de concepto (PoC) centrada en:
* Recomendación de organizaciones relacionada con **pobreza extrema y salud global**.
* Uso de un dataset generado con **[Claude](https://claude.ai/)** como base de conocimiento.
* Evaluación del sistema.

---

### Dataset y fuentes

#### Cómo generé el dataset
Como no encontré un dataset público preparado para este proyecto, generé uno propio usando un LLM ([Claude](https://claude.ai/)) que cumpliese los siguientes requisitos: organizaciones reales, alcance limitado a pobreza extrema, referencias a fuentes basadas en evidencia y salida en formato CSV.

Al ser datos generados sintéticamente, soy consciente de que pueden contener errores, pero considero que es suficiente para una PoC que demuestre que podemos construir el asesor con RAG.

#### Fuentes utilizadas
El dataset incluye referencias a estas fuentes como `source_primary`:
* [GiveWell](https://www.givewell.org/)
* [The Life You Can Save](https://www.thelifeyoucansave.org/)
* [Founders Pledge](https://www.founderspledge.com/)
* [Evidence Action](https://www.evidenceaction.org/)

---

### Desarrollo del sistema RAG

El sistema se construyó usando:
* **[LlamaIndex](https://www.llamaindex.ai/)** como framework de indexación y recuperación.
* **[Chroma](https://www.trychroma.com/)** como vectorstore persistente.
* **Embeddings locales** ([`all-MiniLM-L6-v2`](https://huggingface.co/sentence-transformers/all-MiniLM-L6-v2)).
* Un **modelo instruct** ([`phi-3-mini-4k-instruct`](https://huggingface.co/microsoft/Phi-3-mini-4k-instruct)) para la generación.

#### Retos encontrados
1. **Afinado del prompt**: En las primeras versiones, el modelo no respetaba el formato y generaba texto extra. Lo solucioné con instrucciones más estrictas y usando un token de parada.
2. **Escalado del dataset**: Al pasar de 10 a 100 organizaciones, el contexto aumentó y las respuestas se cortaban. Ajusté el número de tokens y apliqué reglas que hiciesen más concisa la respuesta del modelo.
3. **Evaluación en local**: Inicialmente, la inferencia era dolorosamente lenta porque solo usaba la CPU. Al realizar la evaluación, el proceso no terminaba nunca. Tras investigar y ajustar los `Settings.llm`, logré optimizar el rendimiento de mi **MacBook M4** configurando el `device_map="mps"` para forzar el uso de la GPU (Metal Performance Shaders) y utilizando `float16` para reducir el consumo de RAM.  Esto junto con la limpieza de kernels, permitió que la evaluación fuera muchísimo más rápida, pudiendo ampliar el número de casos evaluados de forma significativa.
4. **Uso de Google Colab**: Intenté usarlo para solventar la falta de recursos, pero surgieron problemas de compatibilidad de versiones. Prioricé la ejecución local optimizada.

---

## Conclusiones

Este proyecto demuestra que un sistema basado en RAG puede utilizarse para construir un recomendador de organizaciones benéficas más controlado que un LLM genérico, siempre que se introduzcan restricciones explícitas.

* El uso de RAG reduce el riesgo de alucinaciones al forzar al modelo a basarse en contexto recuperado y citar sus fuentes.
* La evaluación mostró que el sistema mantiene consistentemente las citas y respeta las instrucciones frente a intentos de prompt injection.
* Es buena idea hacer explícitos los guardrails para señalar qué consultas están fuera de alcance, porque como puede recuperar contexto, aunque no sea relevante, lo está mostrando igual cuando evaluamos querys fuera del alcance.

### Trabajo a futuro
* Ampliar y mejorar el dataset.
* Incorporar mecanismos explícitos de detección de consultas fuera de alcance.
* Ejecutar evaluaciones más amplias en entornos con GPU.
* Crear un chatbot con Streamlit o algún tipo de UI.