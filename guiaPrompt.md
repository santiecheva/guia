-----

## 🛠️ Guía Rápida: Ingeniería de Prompts para Especialistas en RRHH

El **Prompt Engineering** es la clave para que la inteligencia artificial se comporte exactamente como esperamos. Nuestro objetivo es modificar el **Mensaje del Sistema (`SystemMessage`)** para que el chatbot se convierta en un asistente de RRHH riguroso, útil y confiable, maximizando la utilidad del **RAG**.

### 📍 Ubicación del Código a Modificar

El código que debe ser ajustado se encuentra dentro de la función `responder_pregunta` en el archivo **`app.py`**.

### 🧠 Aplicando las 4 Mejores Prácticas de Prompt Engineering

Ajustaremos el `SystemMessage` en cuatro áreas clave: Rol, Contexto RAG, Restricciones y Tono/Formato.

#### 1\. 🧑‍💻 Asignación de Rol Precisa (El "Quién Eres")

Dale al LLM una **autoridad específica** para que filtre mejor sus respuestas.

  * **Práctica:** Asigna un rol formal y especializado.
  * **Ejemplo:** Cambiar de "asistente amigable" a **"Consultor Experto en Políticas de RRHH"**.

#### 2\. 📖 Directiva de Contexto RAG (La Clave del Experto)

Esta es la práctica más importante para el RAG. Obliga al modelo a usar la información de los documentos (`context`) y evita que **alucine** (invente hechos).

  * **Práctica:** Usa frases imperativas para el *grounding*.
  * **Ejemplo:** "**Utiliza exclusivamente el contexto de documentos proporcionado (RAG)** para formular tu respuesta. Nunca inventes o extrapoles información."

#### 3\. 🚧 Guardarraíles y Restricciones (El "Qué NO Hacer")

Establece límites claros para evitar que el chatbot responda a temas fuera de su *expertise* (ej. legales o financieros complejos).

  * **Práctica:** Define una **instrucción de fallo seguro**.
  * **Ejemplo:** "Si la pregunta no se puede responder con el contexto o si es sobre temas legales/financieros, responde: 'Lamento no poder ofrecerte una respuesta específica sobre ese tema. Por favor, consulta al equipo legal o al manual completo.'"

#### 4\. 📝 Especificación de Tono y Formato (El "Cómo Responder")

Asegura una experiencia de usuario clara y profesional.

  * **Práctica:** Indica el tono y el uso de elementos de formato.
  * **Ejemplo:** "Responde en un tono **profesional pero empático**. Usa **listas o negritas** para hacer la información fácil de digerir."

-----

## 💻 Código Final `app.py` (Con Prompt Engineering)

Este es el código final completo del archivo `app.py` con las mejores prácticas de Prompt Engineering aplicadas en el `SystemMessage` y con la lógica de RAG ya integrada.

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
        y luego genera una respuesta basada en ese contexto.
        """
    )
    st.markdown("---")
    st.write("💡 Tip: Pregunta por tus documentos cargados en la carpeta `docs`.")
    st.code("¿Cuál es el proceso para solicitar tiempo libre según el PDF de políticas?")

# --- FUNCIONES CLAVE DE LA IA ---

# 1. Función para cargar el modelo de Ollama (LLM y Embeddings)
@st.cache_resource
def cargar_recursos():
    # 🧠 Carga del LLM (Generador de Respuesta)
    llm = ChatOllama(
        model=LLM_MODEL, 
        temperature=0.2 
    )
    
    # 📚 Carga de Embeddings (Buscador de Vectores)
    embeddings = OllamaEmbeddings(model=EMBEDDING_MODEL)
    
    # 🗃️ Carga del Almacén Vectorial (FAISS Index)
    if os.path.exists(VECTOR_STORE_PATH):
        vector_store = FAISS.load_local(
            VECTOR_STORE_PATH, 
            embeddings, 
            allow_dangerous_deserialization=True # Necesario para cargar FAISS de disco
        )
        # El retriever es el componente que busca los documentos más relevantes
        retriever = vector_store.as_retriever(search_kwargs={"k": 3}) # Busca los 3 fragmentos más relevantes
        return llm, retriever
    else:
        st.error(f"¡Error! No se encontró la base de conocimiento en '{VECTOR_STORE_PATH}'. Ejecuta 'python knowledge_base.py' primero.")
        return llm, None

# 2. Función que envía la pregunta al modelo incluyendo el RAG
def responder_pregunta_rag(pregunta: str, llm, retriever) -> str:
    if not retriever:
        return "El sistema RAG no está inicializado. Por favor, verifica la base de conocimiento y reinicia."

    # --- PLANTILLA DEL PROMPT OPTIMIZADA CON MEJORES PRÁCTICAS DE PE ---
    
    # Esta plantilla define el rol, las reglas RAG y el formato.
    prompt_template = ChatPromptTemplate.from_messages(
        [
            SystemMessage(content="""
                Eres un **Consultor Experto en Políticas de RRHH** y Beneficios.

                **Instrucción Clave (RAG):**
                1. Utiliza **exclusivamente el contexto de documentos proporcionado** (RAG) en la sección 'context' para formular tu respuesta. Nunca inventes o extrapoles información.
                2. Mantén la coherencia con el historial de conversación, pero prioriza la información del contexto RAG.

                **Reglas de Respuesta:**
                * Responde en español, con un tono profesional pero empático.
                * Siempre utiliza **listas, viñetas o negritas** para hacer la información concisa y fácil de leer.

                **Guardarraíl:**
                * Si la pregunta no se puede responder con el contexto de documentos o es sobre temas legales/financieros, responde: "Lamento no poder ofrecerte una respuesta específica sobre ese tema. Por favor, consulta al equipo legal o al manual completo."

                **Contexto de Documentos (RAG):**
                {context}
                """),
            *convertir_historial_a_mensajes(st.session_state.messages),
            HumanMessage(content="{input}"),
        ]
    )

    # 3. Creación de la cadena de documentos (combina la pregunta con los documentos recuperados)
    document_chain = create_stuff_documents_chain(llm, prompt_template)
    
    # 4. Creación de la cadena de recuperación (combina el retriever con la cadena de documentos)
    retrieval_chain = create_retrieval_chain(retriever, document_chain)
    
    # 5. Llamamos al cerebro RAG
    # El historial se pasa implícitamente en el prompt_template
    respuesta = retrieval_chain.invoke({"input": pregunta})
    
    return respuesta["answer"]

# Función auxiliar para convertir el historial de Streamlit a un formato que LangChain entienda
def convertir_historial_a_mensajes(mensajes_historial):
    mensajes_langchain = []
    # Ignoramos el primer mensaje (el mensaje de bienvenida)
    for msg in mensajes_historial[1:]: 
        if msg["role"] == "user":
            mensajes_langchain.append(HumanMessage(content=msg["content"]))
        elif msg["role"] == "assistant":
            mensajes_langchain.append(AIMessage(content=msg["content"]))
    return mensajes_langchain

# 🧠 Estado de la conversación (Mensaje de bienvenida)
if "messages" not in st.session_state:
    st.session_state.messages = [
        {
            "role": "assistant",
            "content": (
                "👋 ¡Hola! Soy tu Chatbot Experto de Recursos Humanos.\n\n"
                "Estoy listo para responder preguntas sobre nuestras políticas internas cargadas. Pregúntame con confianza."
            ),
        }
    ]

# Intentar cargar recursos al inicio
llm, retriever = cargar_recursos()

# 💬 Mostrar historial del chat
for msg in st.session_state.messages:
    avatar = "🧠" if msg["role"] == "assistant" else "🧑‍💼"
    with st.chat_message("assistant" if msg["role"] == "assistant" else "user", avatar=avatar):
        st.markdown(msg["content"])

# 🧾 Input del usuario
prompt = st.chat_input("Escribe tu pregunta sobre políticas internas aquí...")

if prompt:
    # 1. Añadir pregunta del usuario al historial
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user", avatar="🧑‍💼"):
        st.markdown(prompt)

    # 2. Generar respuesta con RAG
    with st.chat_message("assistant", avatar="🧠"):
        with st.spinner("Buscando en los documentos y pensando la mejor respuesta para ti... 💭"):
            if retriever:
                # ¡Aquí llamamos a la IA real con RAG!
                respuesta = responder_pregunta_rag(prompt, llm, retriever)
            else:
                respuesta = "El sistema RAG no pudo inicializarse correctamente. Por favor, revisa la consola para ver los errores."
            
            st.markdown(respuesta)
            
    # 3. Añadir respuesta de la IA al historial
    st.session_state.messages.append({"role": "assistant", "content": respuesta})

```
