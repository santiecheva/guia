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
from langchain_community.embeddings import OllamaEmbeddings
from dotenv import load_dotenv

# Aseguramos que la configuración del modelo de embedding sea coherente
EMBEDDING_MODEL = "deepseek-r1:1.5b" # Usamos el mismo modelo que para el chat

def create_vector_store():
    """
    Carga documentos, los divide, crea embeddings y los guarda en una base de datos FAISS.
    """
    print("--- 1. Cargando documentos desde la carpeta 'docs/' ---")
    
    docs_path = "./docs"
    documents = []
    
    # Busca y carga todos los PDF en la carpeta docs
    for filename in os.listdir(docs_path):
        if filename.endswith(".pdf"):
            file_path = os.path.join(docs_path, filename)
            loader = PyPDFLoader(file_path)
            documents.extend(loader.load())
            print(f"  -> Documento cargado: {filename}")

    if not documents:
        print("ADVERTENCIA: No se encontraron documentos PDF. Asegúrate de tener archivos en la carpeta 'docs'.")
        return

    # 2. División de texto (Partir el conocimiento en trozos pequeños)
    print("\n--- 2. Dividiendo el texto en fragmentos (chunks) ---")
    text_splitter = RecursiveCharacterTextSplitter(
        chunk_size=1000,      # Tamaño máximo de cada trozo de texto
        chunk_overlap=200     # Cuántos caracteres se superponen entre trozos para no perder contexto
    )
    texts = text_splitter.split_documents(documents)
    print(f"  -> Total de fragmentos creados: {len(texts)}")
    
    # 3. Creación de Embeddings (Conversión del texto a números)
    print("\n--- 3. Creando embeddings con Ollama... (puede tardar) ---")
    
    # Usamos el modelo de embeddings de Ollama (el mismo modelo deepseek-r1:1.5b)
    # Nota: Ollama usa el mismo modelo LLM para generar embeddings por defecto.
    embeddings = OllamaEmbeddings(model=EMBEDDING_MODEL) 
    
    # 4. Guardado en la Base de Datos Vectorial FAISS
    print("\n--- 4. Guardando en la base de datos FAISS ---")
    vector_store = FAISS.from_documents(texts, embeddings)
    
    # Guardamos la base de datos en un archivo para usarla después
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
import streamlit as st
from langchain_ollama import ChatOllama
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage
from langchain_community.vectorstores import FAISS
from langchain_community.embeddings import OllamaEmbeddings
from langchain.chains import create_retrieval_chain
from langchain.chains.combine_documents import create_stuff_documents_chain
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.runnables import RunnablePassthrough

# --- CONFIGURACIÓN DE MODELOS ---
LLM_MODEL = "deepseek-r1:1.5b"
EMBEDDING_MODEL = "deepseek-r1:1.5b"
VECTOR_STORE_PATH = "faiss_index_rh" # Carpeta donde guardamos el conocimiento

# 🎨 Configuración básica de la página (El CSS y el sidebar permanecen iguales)
st.set_page_config(
    page_title="Chatbot RH Experto (RAG)",
    page_icon="🧠",
    layout="centered",
    initial_sidebar_state="collapsed"
)

# 🧼 CSS: fondo negro + letra blanca (Se mantiene el CSS completo de la guía anterior)
st.markdown(
    """
    <style>
    /* ... [Código CSS completo de la Etapa 5 aquí] ... */
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
st.caption("Asistente alimentado por la IA local y tus documentos de RRHH. ¡Pregúntale sobre políticas!")

# 🎯 Sidebar con información extra (Actualizamos el mensaje para RAG)
with st.sidebar:
    st.subheader("Acerca de este chatbot RAG")
    st.write(
        f"""
        Este asistente utiliza el modelo `{LLM_MODEL}` y ha sido entrenado
        con tus documentos internos de RRHH.
        """
    )
    st.markdown("---")
    st.write("💡 Tip: Pregunta cosas específicas de tus documentos como:")
    st.code("¿Cuál es la política de WFH para el equipo de ventas?")
    st.code("¿Qué necesito para solicitar un beneficio de bienestar?")

# 🧠 Estado de la conversación (Mensaje de bienvenida)
if "messages" not in st.session_state:
    st.session_state.messages = [
        {
            "role": "assistant",
            "content": (
                "👋 ¡Hola! Soy tu Chatbot Experto en Políticas de Recursos Humanos.\n\n"
                "He leído todos nuestros documentos. Cuéntame, ¿qué política o procedimiento te gustaría consultar hoy?"
            ),
        }
    ]


# --- FUNCIONES CLAVE DE LA IA CON RAG ---

@st.cache_resource
def get_rag_chain():
    """Carga el LLM y la base de datos vectorial para crear la cadena RAG."""
    
    # 1. Cargar el LLM
    llm = ChatOllama(model=LLM_MODEL, temperature=0.1)

    # 2. Cargar el componente de Embeddings
    embeddings = OllamaEmbeddings(model=EMBEDDING_MODEL)

    # 3. Cargar la Base de Datos Vectorial FAISS
    # Si no existe, muestra un error.
    if not os.path.exists(VECTOR_STORE_PATH):
        st.error(f"Error: La base de datos vectorial '{VECTOR_STORE_PATH}' no se encontró.")
        st.stop()
    
    vector_store = FAISS.load_local(VECTOR_STORE_PATH, embeddings, allow_dangerous_deserialization=True)
    retriever = vector_store.as_retriever(search_kwargs={"k": 3}) # El retriever busca los 3 trozos de info más relevantes
    
    # 4. Definir el Prompt para la IA (Instrucciones específicas)
    template = """
    Eres un asistente de Recursos Humanos experto en las políticas internas.
    Tu tarea es responder a la pregunta basándote **únicamente** en el contexto proporcionado por los documentos.
    Si la respuesta no se encuentra en el contexto, indica amablemente que no tienes la información sobre ese tema específico.
    Responde en español, con un tono profesional y servicial.
    
    CONTEXTO:
    {context}
    
    PREGUNTA:
    {input}
    """
    prompt = ChatPromptTemplate.from_template(template)
    
    # 5. Crear la Cadena RAG (Unir el Retreiver, el Prompt y el LLM)
    
    # Define cómo el LLM debe combinar el contexto
    document_chain = create_stuff_documents_chain(llm, prompt)
    
    # Crea la cadena final que primero busca (retrieval) y luego genera (generation)
    rag_chain = create_retrieval_chain(retriever, document_chain)
    
    return rag_chain

def responder_pregunta_rag(pregunta: str) -> str:
    """Invoca la cadena RAG con la pregunta del usuario."""
    rag_chain = get_rag_chain()
    
    # La cadena RAG ya maneja el contexto de recuperación
    response = rag_chain.invoke({"input": pregunta})
    
    # El resultado de create_retrieval_chain viene en un diccionario con la clave 'answer'
    return response['answer']

# -----------------------------------

# 💬 Mostrar historial del chat
for msg in st.session_state.messages:
    avatar = "🤖" if msg["role"] == "assistant" else "🧑‍💼"
    with st.chat_message("assistant" if msg["role"] == "assistant" else "user", avatar=avatar):
        st.markdown(msg["content"])

# 🧾 Input del usuario
prompt = st.chat_input("Escribe tu pregunta sobre políticas de RRHH aquí...")

if prompt:
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user", avatar="🧑‍💼"):
        st.markdown(prompt)

    with st.chat_message("assistant", avatar="🤖"):
        with st.spinner("Buscando en las políticas de RRHH y pensando... 💭"):
            try:
                # ¡Aquí llamamos a la función RAG!
                respuesta = responder_pregunta_rag(prompt)
                st.markdown(respuesta)
            except Exception as e:
                 # Manejo de error si la base de datos no carga
                st.error("Error al acceder a la base de conocimiento. ¿Ejecutaste 'python knowledge_base.py' con Ollama activo?")
                print(f"Detalle del error: {e}")
                respuesta = "Lo siento, no pude acceder a las políticas internas."
                
    st.session_state.messages.append({"role": "assistant", "content": respuesta})
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

