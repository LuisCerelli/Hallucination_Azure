# English:

# Hallucination Detection and Analysis in NLP Prompts

## Overview

This repository contains a Python-based solution to detect and analyze hallucinations in text prompts. Hallucinations refer to information that is fabricated, inconsistent, or not backed by factual data. By integrating Natural Language Processing (NLP) tools, Azure Cognitive Search, and pre-trained transformer models, this system evaluates text for its factuality, coherence, and relevance against reliable references.

---

## Features

### 1. **Fact-Checking Against Azure Cognitive Search**
- Uses Azure Cognitive Search to retrieve relevant reference documents.
- Evaluates factual accuracy using a transformer-based question-answering model.

### 2. **Internal Consistency Analysis**
- Employs SpaCy to analyze grammatical and semantic consistency in text.
- Identifies potential contradictions or incoherent statements.

### 3. **Hallucination Metrics**
- **`alucinaciones_probabilidad`**: Probability score (0-1) indicating the likelihood of hallucinations.
- **Comments**: Detailed feedback on inconsistencies or unsupported claims.
- **Recommendations**: Suggested actions to improve factuality or relevance.

---

## Installation

### Prerequisites
Ensure you have Python 3.8 or later installed.

### Install Required Packages
```bash
!pip install spacy langchain azure-search-documents azure-core transformers tf-keras torch
```

### Download SpaCy Model
```bash
!python3 -m spacy download en_core_web_sm
```

---

## Configuration

### Setting Up Environment Variables
Create a `.env` file in the root directory and include the following:

```env
# Azure Cognitive Search Configuration
AZURE_SEARCH_ENDPOINT="https://datos-huelladecarbono.search.windows.net"
AZURE_SEARCH_KEY="<your-azure-key>"
INDEX_NAME="documentos-huella"
```

Add the `.env` file to `.gitignore` to protect sensitive information:

```plaintext
.env
```

---

## Code Workflow

### **1. Load Environment Variables**
Environment variables are loaded dynamically using `dotenv`:
```python
from dotenv import load_dotenv, find_dotenv
import os

dotenv_path = find_dotenv()
load_dotenv(dotenv_path)

AZURE_SEARCH_ENDPOINT = os.getenv("AZURE_SEARCH_ENDPOINT", "").strip()
AZURE_SEARCH_KEY = os.getenv("AZURE_SEARCH_KEY", "").strip()
INDEX_NAME = os.getenv("INDEX_NAME", "").strip()
```

### **2. Connect to Azure Cognitive Search**
```python
from azure.search.documents import SearchClient
from azure.core.credentials import AzureKeyCredential

search_client = SearchClient(
    endpoint=AZURE_SEARCH_ENDPOINT,
    index_name=INDEX_NAME,
    credential=AzureKeyCredential(AZURE_SEARCH_KEY)
)
```

### **3. Load NLP Models**
```python
import spacy
from transformers import pipeline

nlp = spacy.load("en_core_web_sm")
factual_pipeline = pipeline("question-answering", model="distilbert-base-cased-distilled-squad", framework="pt")
```

### **4. Hallucination Detection Functions**
#### Fact-Checking
Compares the input text against retrieved documents using the QA pipeline:
```python
def verificar_factualidad(texto, documentos_referencia):
    respuestas = []
    for doc in documentos_referencia:
        respuesta = factual_pipeline({
            "question": texto,
            "context": doc
        })
        respuestas.append(respuesta["score"])
    return max(respuestas) if respuestas else 0
```

#### Consistency Analysis
Identifies potential contradictions:
```python
def analizar_coherencia(texto):
    doc = nlp(texto)
    inconsistencias = []
    for sent in doc.sents:
        tokens = [token.text for token in sent if token.dep_ == "neg"]
        if tokens:
            inconsistencias.append(sent.text)
    return inconsistencias
```

#### Orchestrator
Combines the results of fact-checking and consistency analysis:
```python
def evaluar_prompt(texto):
    documentos = buscar_en_azure(texto)
    if not documentos:
        return {
            "alucinaciones_probabilidad": 1.0,
            "comentarios": "No se encontraron documentos de referencia para el texto.",
            "acciones_recomendadas": ["Agregar documentos relevantes al índice Azure."]
        }

    factualidad = verificar_factualidad(texto, documentos)
    inconsistencias = analizar_coherencia(texto)

    probabilidad_alucinacion = 1 - factualidad if factualidad else 1.0
    comentarios = []

    if probabilidad_alucinacion > 0.5:
        comentarios.append("La información parece ficticia o no respaldada por los documentos.")
    if inconsistencias:
        comentarios.append(f"Inconsistencias detectadas: {', '.join(inconsistencias)}")

    return {
        "alucinaciones_probabilidad": probabilidad_alucinacion,
        "comentarios": " ".join(comentarios),
        "acciones_recomendadas": [
            "Verificar el contenido manualmente.",
            "Agregar más datos relevantes al índice de Azure."
        ]
    }
```

---

## Example Usage

### Input
```python
texto_ejemplo = "La huella de carbono mide emisiones de CO₂ producidas por actividades humanas."
resultado = evaluar_prompt(texto_ejemplo)
print(json.dumps(resultado, indent=4))
```

### Output
```json
{
    "alucinaciones_probabilidad": 0.2,
    "comentarios": "La información parece respaldada por documentos, pero se encontraron inconsistencias detectadas: ...",
    "acciones_recomendadas": ["Verificar el contenido manualmente."]
}
```

---

## Contribution
Contributions are welcome! Please ensure all sensitive keys are stored in `.env` and never committed.

---

## License
This project is licensed under the MIT License.

# Español:  

# Detección y Análisis de Alucinaciones en Prompts de NLP

## Descripción General

Este repositorio contiene una solución basada en Python para detectar y analizar alucinaciones en textos generados. Las alucinaciones se refieren a información inventada, inconsistente o no respaldada por datos factuales. Integrando herramientas de Procesamiento de Lenguaje Natural (NLP), Azure Cognitive Search y modelos preentrenados de transformers, este sistema evalúa textos por su factualidad, coherencia y relevancia en comparación con referencias confiables.

---

## Características

### 1. **Verificación de Hechos contra Azure Cognitive Search**
- Utiliza Azure Cognitive Search para recuperar documentos de referencia relevantes.
- Evalúa la precisión factual utilizando un modelo de preguntas y respuestas basado en transformers.

### 2. **Análisis de Coherencia Interna**
- Emplea SpaCy para analizar la consistencia gramatical y semántica en el texto.
- Identifica posibles contradicciones o declaraciones incoherentes.

### 3. **Métricas de Alucinaciones**
- **`alucinaciones_probabilidad`**: Puntuación de probabilidad (0-1) que indica la posibilidad de alucinaciones.
- **Comentarios**: Retroalimentación detallada sobre inconsistencias o afirmaciones no respaldadas.
- **Recomendaciones**: Acciones sugeridas para mejorar la factualidad o relevancia.

---

## Instalación

### Requisitos Previos
Asegúrate de tener Python 3.8 o posterior instalado.

### Instalar Paquetes Requeridos
```bash
!pip install spacy langchain azure-search-documents azure-core transformers tf-keras torch
```

### Descargar Modelo de SpaCy
```bash
!python3 -m spacy download en_core_web_sm
```

---

## Configuración

### Configuración de Variables de Entorno
Crea un archivo `.env` en el directorio raíz y agrega lo siguiente:

```env
# Configuración de Azure Cognitive Search
AZURE_SEARCH_ENDPOINT="https://datos-huelladecarbono.search.windows.net"
AZURE_SEARCH_KEY="<tu-clave-azure>"
INDEX_NAME="documentos-huella"
```

Agrega el archivo `.env` al archivo `.gitignore` para proteger información sensible:

```plaintext
.env
```

---

## Flujo del Código

### **1. Cargar Variables de Entorno**
Las variables de entorno se cargan dinámicamente utilizando `dotenv`:
```python
from dotenv import load_dotenv, find_dotenv
import os

dotenv_path = find_dotenv()
load_dotenv(dotenv_path)

AZURE_SEARCH_ENDPOINT = os.getenv("AZURE_SEARCH_ENDPOINT", "").strip()
AZURE_SEARCH_KEY = os.getenv("AZURE_SEARCH_KEY", "").strip()
INDEX_NAME = os.getenv("INDEX_NAME", "").strip()
```

### **2. Conectar a Azure Cognitive Search**
```python
from azure.search.documents import SearchClient
from azure.core.credentials import AzureKeyCredential

search_client = SearchClient(
    endpoint=AZURE_SEARCH_ENDPOINT,
    index_name=INDEX_NAME,
    credential=AzureKeyCredential(AZURE_SEARCH_KEY)
)
```

### **3. Cargar Modelos de NLP**
```python
import spacy
from transformers import pipeline

nlp = spacy.load("en_core_web_sm")
factual_pipeline = pipeline("question-answering", model="distilbert-base-cased-distilled-squad", framework="pt")
```

### **4. Funciones para Detección de Alucinaciones**
#### Verificación de Hechos
Compara el texto de entrada con documentos recuperados utilizando el pipeline de QA:
```python
def verificar_factualidad(texto, documentos_referencia):
    respuestas = []
    for doc in documentos_referencia:
        respuesta = factual_pipeline({
            "question": texto,
            "context": doc
        })
        respuestas.append(respuesta["score"])
    return max(respuestas) if respuestas else 0
```

#### Análisis de Coherencia
Identifica posibles contradicciones:
```python
def analizar_coherencia(texto):
    doc = nlp(texto)
    inconsistencias = []
    for sent in doc.sents:
        tokens = [token.text for token in sent if token.dep_ == "neg"]
        if tokens:
            inconsistencias.append(sent.text)
    return inconsistencias
```

#### Orquestador
Combina los resultados de la verificación de hechos y el análisis de coherencia:
```python
def evaluar_prompt(texto):
    documentos = buscar_en_azure(texto)
    if not documentos:
        return {
            "alucinaciones_probabilidad": 1.0,
            "comentarios": "No se encontraron documentos de referencia para el texto.",
            "acciones_recomendadas": ["Agregar documentos relevantes al índice Azure."]
        }

    factualidad = verificar_factualidad(texto, documentos)
    inconsistencias = analizar_coherencia(texto)

    probabilidad_alucinacion = 1 - factualidad if factualidad else 1.0
    comentarios = []

    if probabilidad_alucinacion > 0.5:
        comentarios.append("La información parece ficticia o no respaldada por los documentos.")
    if inconsistencias:
        comentarios.append(f"Inconsistencias detectadas: {', '.join(inconsistencias)}")

    return {
        "alucinaciones_probabilidad": probabilidad_alucinacion,
        "comentarios": " ".join(comentarios),
        "acciones_recomendadas": [
            "Verificar el contenido manualmente.",
            "Agregar más datos relevantes al índice de Azure."
        ]
    }
```

---

## Ejemplo de Uso

### Entrada
```python
texto_ejemplo = "La huella de carbono mide emisiones de CO₂ producidas por actividades humanas."
resultado = evaluar_prompt(texto_ejemplo)
print(json.dumps(resultado, indent=4))
```

### Salida
```json
{
    "alucinaciones_probabilidad": 0.2,
    "comentarios": "La información parece respaldada por documentos, pero se encontraron inconsistencias detectadas: ...",
    "acciones_recomendadas": ["Verificar el contenido manualmente."]
}
```

---

## Contribuciones
¡Las contribuciones son bienvenidas! Por favor, asegúrate de almacenar todas las claves sensibles en `.env` y nunca las incluyas en commits.

---

## Licencia
Este proyecto está licenciado bajo la Licencia MIT.

