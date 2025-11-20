-----

## 🚀 Guía Práctica: Convirtiendo tu Chatbot en un Experto (RAG)

El **RAG** (Generación Aumentada por Recuperación) permite que tu modelo de IA local (`deepseek-r1:1.5b`) no solo use su "conocimiento general", sino que también **lea y consulte** documentos externos (como manuales de políticas o guías de *onboarding*) antes de responder.

### 💡 Nuevos Componentes Necesarios

| Componente | Rol (Explicación para RRHH) | Herramienta/Concepto |
| :--- | :--- | :--- |
| **Documentos** | Tus políticas, manuales, etc., que serán la fuente de verdad. | Archivos PDF, DOCX, TXT. |
| **Embeddings** | El proceso de convertir texto complejo en **"vectores"** o números que la IA puede entender y buscar rápidamente. | **Ollama Embeddings** |
| **Base de Datos Vectorial** | El "archivo" donde guardamos la versión numérica (los *embeddings*) de tus documentos para hacer búsquedas rápidas. | **FAISS** (como base de datos simple) |
| **El Conector** | Usamos LangChain para gestionar el proceso de RAG completo. | **LangChain** |

-----

## 🛠️ ETAPA 6: Preparando el Entorno y los Documentos

### 6.1. Instalar Librerías Adicionales

Necesitamos algunas herramientas extra para manejar documentos y la base de datos vectorial.

Abre la terminal de VSCode (con el entorno activado `(venv)`) e instala:

```bash
pip install pypdf faiss-cpu
```

  * **`pypdf`**: Nos permite leer archivos PDF.
  * **`faiss-cpu`**: Es la herramienta que usaremos como nuestra Base de Datos Vectorial (el índice donde se guardan los "vectores" de conocimiento).

### 6.2. Crear Carpeta de Documentos

Crea una carpeta dentro de tu proyecto `chatbot-rh` llamada `docs`.

📁 `C:\Users\tu_usuario\Desktop\chatbot-rh\docs`

Coloca dentro de esta carpeta los documentos (por ejemplo, un PDF de políticas de vacaciones o el manual de *onboarding*) que quieres que el chatbot aprenda.

-----

## 🧠 ETAPA 7: Creando el Cerebro Experto (RAG)

Ahora crearemos un nuevo archivo para procesar los documentos y generar el conocimiento que luego usaremos en el `app.py`.

### 7.1. Crear el Archivo de Procesamiento

Crea un archivo llamado `knowledge_base.py` en la carpeta raíz de tu proyecto.

📄 `knowledge_base.py`

### 7.2. Código para Procesar y Guardar el Conocimiento

Pega este código en `knowledge_base.py`. Este código se ejecuta solo **una vez** para convertir tus documentos a un formato que la IA puede buscar.

```python
import os
from langchain_community.document_loaders import PyPDFLoader
from langchain_community.vectorstores import FAISS
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_ollama import OllamaEmbeddings
from dotenv import load_dotenv

# Modelo de Ollama que se usará para generar los embeddings de los documentos
EMBEDDING_MODEL = "qwen2.5:0.5b"

def create_vector_store():
    """
    Carga todos los PDFs de la carpeta 'docs', los divide en fragmentos,
    genera embeddings con Ollama y guarda el resultado en una base de datos
    vectorial FAISS en disco.
    """
    print("--- 1. Cargando documentos desde la carpeta 'docs/' ---")
    
    docs_path = "./docs"
    documents = []
    
    # Recorre la carpeta 'docs' y carga todos los archivos PDF encontrados
    for filename in os.listdir(docs_path):
        if filename.endswith(".pdf"):
            file_path = os.path.join(docs_path, filename)
            loader = PyPDFLoader(file_path)
            documents.extend(loader.load())
            print(f"  -> Documento cargado: {filename}")

    if not documents:
        print("ADVERTENCIA: No se encontraron documentos PDF. "
              "Asegúrate de tener archivos en la carpeta 'docs'.")
        return

    # 2. División de texto en fragmentos (chunks) manejables
    print("\n--- 2. Dividiendo el texto en fragmentos (chunks) ---")
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000,      # Máximo de caracteres por fragmento
        chunk_overlap=200     # Superposición entre fragmentos para no perder contexto
    )
    texts = text_splitter.split_documents(documents)
    print(f"  -> Total de fragmentos creados: {len(texts)}")
    
    # 3. Creación de embeddings (conversión de texto a vectores numéricos) usando Ollama
    print("\n--- 3. Creando embeddings con Ollama... (puede tardar) ---")
    embeddings = OllamaEmbeddings(model=EMBEDDING_MODEL)
    
    # 4. Creación y guardado de la base de datos vectorial FAISS
    print("\n--- 4. Creando y guardando la base de datos FAISS ---")
    vector_store = FAISS.from_documents(texts, embeddings)
    
    # Guardamos el índice FAISS en disco para poder reutilizarlo desde la app (Streamlit)
    vector_store.save_local("faiss_index_rh")
    print("\n✅ Base de conocimiento guardada como 'faiss_index_rh'. ¡Lista para usar!")

if __name__ == "__main__":
    create_vector_store()

```

### 7.3. Ejecutar el Procesamiento

En la terminal de VSCode (con el entorno `(venv)` activo), ejecuta el script:

```bash
python knowledge_base.py
```

  * **Importante:** Asegúrate de que **Ollama** esté corriendo en segundo plano antes de ejecutar esto.
  * Si todo sale bien, se creará una carpeta llamada `faiss_index_rh` con el conocimiento de tus documentos.

-----

## 🔌 ETAPA 8: Conectando el RAG al Chatbot

Ahora modificaremos nuestro `app.py` para que use esta base de conocimiento experta.

### 8.1. Actualizar `app.py`

Abre `app.py` y **reemplaza TODO el contenido** por este código final. Los cambios clave están en las funciones `cargar_modelo` y `responder_pregunta`, que ahora usan la base de conocimiento (`faiss_index_rh`).

```python
import os  
import streamlit as st
from langchain_ollama import OllamaEmbeddings
from langchain_ollama import ChatOllama
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage
from langchain_community.vectorstores import FAISS
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

# --- CONFIGURACIÓN DE MODELOS Y RUTA DE LA BD VECTORIAL ---
# LLM que usará Ollama para generar las respuestas
LLM_MODEL = "deepseek-r1:1.5b"
# Modelo que usará Ollama para crear los embeddings
EMBEDDING_MODEL = "qwen2.5:0.5b"
# Carpeta donde se guardó el índice FAISS generado en knowledge_base.py
VECTOR_STORE_PATH = "faiss_index_rh"

# 🎨 Configuración básica de la página de Streamlit
st.set_page_config(
    page_title="Chatbot RH Experto (RAG)",
    page_icon="🧠",
    layout="centered",
    initial_sidebar_state="collapsed"
)

# 🧼 CSS: modo oscuro básico para la app (fondo negro, texto blanco)
st.markdown(
    """
    <style>
    .stApp {background-color: #000000 !important; color: #ffffff !important;}
    * {color: #ffffff !important;}
    #MainMenu {visibility: hidden;}
    footer {visibility: hidden;}
    .block-container {max-width: 900px; padding-top: 2rem; padding-bottom: 2rem;}
    h1, h2, h3, h4, h5, h6 {text-align: center; color: #ffffff !important;}
    section[data-testid="stSidebar"] {background-color: #111111 !important;}
    .stCode, code {color: #ffffff !important; background-color: #222222 !important;}
    .stChatMessage {background-color: #111111 !important; color: #ffffff !important;}
    </style>
    """,
    unsafe_allow_html=True
)

# 🧱 Encabezado principal
st.title("🧠 Chatbot RH Experto (RAG)")
st.caption("Asistente que combina IA local con tus documentos de RRHH para responder sobre políticas internas.")

# 🎯 Sidebar con explicación de la app y ejemplos de uso
with st.sidebar:
    st.subheader("Acerca de este chatbot RAG")
    st.write(
        f"""
        Este asistente usa el modelo de lenguaje local `{LLM_MODEL}` y un índice vectorial
        creado a partir de tus documentos internos de Recursos Humanos.
        
        Cuando haces una pregunta, primero busca fragmentos relevantes en esos documentos
        y luego genera una respuesta basada en ese contexto (pat

```

### 8.2. Ejecutar y Probar el Experto

1.  Asegúrate de haber ejecutado **`python knowledge_base.py`** con documentos en la carpeta `docs` (Etapa 7).

2.  Asegúrate de que **Ollama** esté corriendo en segundo plano.

3.  Ejecuta el chatbot actualizado:

    ```bash
    streamlit run app.py
    ```

Tu chatbot ahora responderá preguntas como: "¿Cuántos días de vacaciones tengo?" o "¿Cómo es el proceso de un reembolso?" basándose en la información **exacta** que tú cargaste en los PDF.

-----

