## 🤖 ¡Manos a la Obra\! Guía Práctica para Construir tu Chatbot de RRHH con IA Local

**Bienvenido/a al curso de Inteligencia Artificial para RRHH.** Este proyecto te mostrará paso a paso cómo crear un asistente de chat tipo ChatGPT que funciona **directamente en tu computador** con el modelo **`deepseek-r1:1.5b`**.

-----

### 💡 Nuestro Kit de Herramientas

| Componente | Rol (Explicación para RRHH) | Herramienta |
| :--- | :--- | :--- |
| **El Cerebro** | La inteligencia artificial que piensa y responde. | **deepseek-r1:1.5b** (Vía Ollama) |
| **El Cuerpo** | La interfaz visible donde escribes y lees (la "cara" del chat). | **Streamlit** |
| **El Conector** | El cable o puente que une la interfaz con el cerebro. | **LangChain** |

-----

## ⚙️ ETAPA 1: Preparación Inicial (La Caja de Herramientas)

Tranquilo/a: estos pasos solo se hacen **una vez** en tu computador.

### 1.1. Instalar Python

Python es el idioma que usaremos para darle las instrucciones al chatbot.

1.  Entra a: [https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/)

2.  Descarga la versión recomendada.

3.  Abre el instalador.

4.  **⚠️ CLAVE:** Marca la casilla **✔️ "Add Python to PATH"**.

5.  Haz clic en **Install Now**.

6.  Para verificar, abre el **Símbolo del sistema** (`cmd`) y escribe:

    ```bash
    python --version
    ```

### 1.2. Instalar Visual Studio Code (VSCode)

VSCode es nuestro "editor" o cuaderno digital para escribir y guardar el código del proyecto.

1.  Ve a: [https://code.visualstudio.com/](https://code.visualstudio.com/)
2.  Descarga la versión para Windows e instálala con las opciones por defecto.
3.  Ábrelo al finalizar.

-----

## 🌳 ETAPA 2: El Taller y los Materiales (Entorno Virtual y Librerías)

Aquí crearemos el espacio de trabajo aislado para nuestro chatbot.

### 2.1. Crear una Carpeta de Trabajo

Crea una carpeta en un lugar fácil de recordar:

📁 `C:\Users\tu_usuario\Desktop\chatbot-rh`

### 2.2. Abrir VSCode en esa Carpeta

1.  Abre VSCode.
2.  Arrastra la carpeta `chatbot-rh` dentro del VSCode.

### 2.3. Crear y Activar un Entorno Virtual (`venv`)

El entorno virtual aísla las librerías de este proyecto.

1.  Abre la terminal de VSCode (**Terminal \> New Terminal**).

2.  Crea el entorno:

    ```bash
    python -m venv venv
    ```

3.  Activa el entorno:

    ```bash
    venv\Scripts\activate.bat
    ```

    *Si tienes un error, prueba con `venv\Scripts\activate`*

    Si funciona, verás **`(venv)`** al inicio de la línea.

### 2.4. Instalar los Materiales (Librerías)

Con el entorno activado, instalaremos los paquetes de Python necesarios:

```bash
pip install streamlit langchain langchain-ollama python-dotenv ollama
```

-----

## 🧠 ETAPA 3: El Cerebro Local (Ollama y `deepseek-r1:1.5b`)

Ollama es la "fábrica de cerebros" que nos permite usar modelos de IA locales.

### 3.1. Descargar e Instalar Ollama (la Fábrica)

1.  Ve a: [https://ollama.com](https://ollama.com)

2.  Haz clic en **Download for Windows** e instálalo con las opciones por defecto.

3.  Ollama se ejecutará en segundo plano automáticamente.

### 3.2. Descargar el Modelo `deepseek-r1:1.5b`

Ahora bajaremos el modelo de IA específico.

1.  Abre una terminal nueva o usa la de VSCode.

2.  Escribe el comando para descargar el modelo:

    ```bash
    ollama pull deepseek-r1:1.5b
    ```

    *La primera vez tardará varios minutos.*

### 3.3. Probar el Modelo (Opcional)

Solo para confirmar que el cerebro funciona, chatea un momento con él. Para salir, presiona **Ctrl + C**.

```bash
ollama run deepseek-r1:1.5b
```

-----

## 🎨 ETAPA 4: El Cuerpo (Interfaz con Streamlit)

Creamos la "cara" del chatbot.

### 4.1. Crear el archivo `app.py` (Demo)

En VSCode, crea un archivo llamado:

📄 `app.py`

Y pega el siguiente código. En esta etapa, el chatbot solo muestra la interfaz y una respuesta de **demostración** (aún no llama a la IA).

```python
import streamlit as st

# 🎨 Configuración básica de la página
st.set_page_config(
    page_title="Chatbot RH",
    page_icon="🤖",
    layout="centered",
    initial_sidebar_state="collapsed"
)

# 🧼 CSS: fondo negro + letra blanca
st.markdown(
    """
    <style>
    /* Fondo negro */
    .stApp {
        background-color: #000000 !important;
        color: #ffffff !important;
    }

    /* Texto general en blanco */
    * {
        color: #ffffff !important;
    }

    /* Ocultar menú y pie de página de Streamlit */
    #MainMenu {visibility: hidden;}
    footer {visibility: hidden;}

    /* Ajustar el ancho del contenedor principal */
    .block-container {
        max-width: 900px;
        padding-top: 2rem;
        padding-bottom: 2rem;
    }

    /* Títulos centrados y blancos */
    h1, h2, h3, h4, h5, h6 {
        text-align: center;
        color: #ffffff !important;
    }

    /* Sidebar oscuro */
    section[data-testid="stSidebar"] {
        background-color: #111111 !important;
    }

    /* Código dentro del sidebar */
    .stCode, code {
        color: #ffffff !important;
        background-color: #222222 !important;
    }

    /* Chat bubbles */
    .stChatMessage {
        background-color: #111111 !important;
        color: #ffffff !important;
    }
    </style>
    """,
    unsafe_allow_html=True
)

# 🧠 Estado de la conversación (Mensaje de bienvenida)
if "messages" not in st.session_state:
    st.session_state.messages = [
        {
            "role": "assistant",
            "content": (
                "👋 ¡Hola! Soy tu Chatbot de Recursos Humanos.\n\n"
                "Puedo ayudarte con cosas como:\n"
                "- Políticas de vacaciones 🏖️\n"
                "- Beneficios y bienestar 🎁\n"
                "- Procesos de selección 🧑‍💼\n"
                "- Onboarding de nuevas personas 🚀\n\n"
                "Cuéntame, ¿qué te gustaría saber hoy?"
            ),
        }
    ]

# 🧱 Encabezado principal
st.title("🤖 Chatbot para Recursos Humanos")
st.caption("Tu asistente amigable de RRHH – cero tecnicismos, respuestas claras y un toque de humor 😄")

# 🎯 Sidebar con información extra
with st.sidebar:
    st.subheader("Acerca de este chatbot")
    st.write(
        """
        Este asistente está pensado para:
        - Responder preguntas frecuentes de RRHH  
        - Servir como ejemplo en un curso de IA  
        - Mostrar cómo se ve un chat tipo ChatGPT con Streamlit  
        """
    )
    st.markdown("---")
    st.write("💡 Tip: prueba con preguntas como:")
    st.code("¿Cuántos días de vacaciones tengo al año?")
    st.code("¿Cómo es el proceso de onboarding?")

# 🤹 Respuesta demo (Función de demostración, no usa la IA)
def generar_respuesta_demo(pregunta: str) -> str:
    respuesta = f"""
He recibido tu pregunta:

> *{pregunta}*

🔍 **Versión demo:** aquí iría la respuesta inteligente del modelo de IA.

Mientras conectamos el modelo real (`deepseek-r1:1.5b`), podemos imaginar que ya soy súper listo 😄
"""
    return respuesta

# 💬 Mostrar historial del chat
for msg in st.session_state.messages:
    avatar = "🤖" if msg["role"] == "assistant" else "🧑‍💼"
    with st.chat_message("assistant" if msg["role"] == "assistant" else "user", avatar=avatar):
        st.markdown(msg["content"])

# 🧾 Input del usuario
prompt = st.chat_input("Escribe tu pregunta sobre Recursos Humanos aquí...")

if prompt:
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user", avatar="🧑‍💼"):
        st.markdown(prompt)

    with st.chat_message("assistant", avatar="🤖"):
        with st.spinner("Pensando la mejor respuesta para ti... 💭"):
            respuesta = generar_respuesta_demo(prompt)
            st.markdown(respuesta)
    st.session_state.messages.append({"role": "assistant", "content": respuesta})
```

### 4.2. Probar la Interfaz

En la terminal de VSCode (con el entorno activado `(venv)`), ejecuta:

```bash
streamlit run app.py
```

Se abrirá un navegador con tu interfaz.

-----

## 🔌 ETAPA 5: Conexión Final (El Chatbot Funcional)

Unimos el **Cuerpo (Streamlit)** con el **Cerebro (deepseek-r1:1.5b vía LangChain)**.

### 5.1. Reemplazar el código de `app.py`

Abre `app.py` y **reemplaza TODO el contenido** por este código final. Aquí se añade la lógica de LangChain para interactuar con Ollama.

```python
import streamlit as st
from langchain_ollama import ChatOllama
from langchain_core.messages import HumanMessage, SystemMessage, AIMessage


# 🎨 Configuración básica de la página
st.set_page_config(
    page_title="Chatbot RH",
    page_icon="🤖",
    layout="centered",
    initial_sidebar_state="collapsed"
)

# 🧼 CSS: fondo negro + letra blanca
st.markdown(
    """
    <style>
    /* Fondo negro */
    .stApp {
        background-color: #000000 !important;
        color: #ffffff !important;
    }

    /* Texto general en blanco */
    * {
        color: #ffffff !important;
    }

    /* Ocultar menú y pie de página de Streamlit */
    #MainMenu {visibility: hidden;}
    footer {visibility: hidden;}

    /* Ajustar el ancho del contenedor principal */
    .block-container {
        max-width: 900px;
        padding-top: 2rem;
        padding-bottom: 2rem;
    }

    /* Títulos centrados y blancos */
    h1, h2, h3, h4, h5, h6 {
        text-align: center;
        color: #ffffff !important;
    }

    /* Sidebar oscuro */
    section[data-testid="stSidebar"] {
        background-color: #111111 !important;
    }

    /* Código dentro del sidebar */
    .stCode, code {
        color: #ffffff !important;
        background-color: #222222 !important;
    }

    /* Chat bubbles */
    .stChatMessage {
        background-color: #111111 !important;
        color: #ffffff !important;
    }
    </style>
    """,
    unsafe_allow_html=True
)

# 🧠 Estado de la conversación (Mensaje de bienvenida)
if "messages" not in st.session_state:
    st.session_state.messages = [
        {
            "role": "assistant",
            "content": (
                "👋 ¡Hola! Soy tu Chatbot de Recursos Humanos.\n\n"
                "Puedo ayudarte con cosas como:\n"
                "- Políticas de vacaciones 🏖️\n"
                "- Beneficios y bienestar 🎁\n"
                "- Procesos de selección 🧑‍💼\n"
                "- Onboarding de nuevas personas 🚀\n\n"
                "Cuéntame, ¿qué te gustaría saber hoy?"
            ),
        }
    ]

# 🧱 Encabezado principal
st.title("🤖 Chatbot para Recursos Humanos")
st.caption("Tu asistente amigable de RRHH – cero tecnicismos, respuestas claras y un toque de humor 😄")

# 🎯 Sidebar con información extra
with st.sidebar:
    st.subheader("Acerca de este chatbot")
    st.write(
        """
        Este asistente está pensado para:
        - Responder preguntas frecuentes de RRHH  
        - Servir como ejemplo en un curso de IA  
        - Mostrar cómo se ve un chat tipo ChatGPT con Streamlit  
        """
    )
    st.markdown("---")
    st.write("💡 Tip: prueba con preguntas como:")
    st.code("¿Cuántos días de vacaciones tengo al año?")
    st.code("¿Cómo es el proceso de onboarding?")

# --- FUNCIONES CLAVE DE LA IA ---

# 1. Función para cargar el modelo de Ollama
@st.cache_resource
def cargar_modelo():
    # Modelo local y gratuito usando Ollama
    modelo = ChatOllama(
        # ¡IMPORTANTE! Usamos el modelo deepseek-r1:1.5b que descargamos en la Etapa 3.
        model="deepseek-r1:1.5b",    
        temperature=0.2          # Qué tan creativo es (0 = muy serio, 1 = muy creativo)
    )
    return modelo

# 2. Función que envía la pregunta al modelo e incluye el historial
def responder_pregunta(pregunta: str) -> str:
    modelo = cargar_modelo()
    
    # 📝 Creamos el "contexto" o historial
    mensajes = [
        SystemMessage(content="Eres un asistente amigable de Recursos Humanos. Responde en español, de forma clara, con cero tecnicismos y un tono servicial. Céntrate solo en temas de RRHH."),
    ]
    
    # Añadimos los mensajes del historial
    for msg in st.session_state.messages:
        if msg["role"] == "user":
            mensajes.append(HumanMessage(content=msg["content"]))
        elif msg["role"] == "assistant":
            # Excluimos el mensaje de bienvenida inicial
            if "¡Hola!" not in msg["content"]: 
                mensajes.append(AIMessage(content=msg["content"]))
    
    # El último mensaje siempre es la pregunta actual
    mensajes.append(HumanMessage(content=pregunta))

    # 🚀 Llamamos al cerebro (al modelo)
    respuesta = modelo.invoke(mensajes)
    return respuesta.content

# -----------------------------------


# 💬 Mostrar historial del chat
for msg in st.session_state.messages:
    avatar = "🤖" if msg["role"] == "assistant" else "🧑‍💼"
    with st.chat_message("assistant" if msg["role"] == "assistant" else "user", avatar=avatar):
        st.markdown(msg["content"])

# 🧾 Input del usuario
prompt = st.chat_input("Escribe tu pregunta sobre Recursos Humanos aquí...")

if prompt:
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user", avatar="🧑‍💼"):
        st.markdown(prompt)

    with st.chat_message("assistant", avatar="🤖"):
        with st.spinner("Pensando la mejor respuesta para ti... 💭"):
            # ¡Aquí llamamos a la IA real!
            respuesta = responder_pregunta(prompt)
            st.markdown(respuesta)
    st.session_state.messages.append({"role": "assistant", "content": respuesta})
```

### 5.2. Ejecutar el Chatbot

Asegúrate de que **Ollama está corriendo** en segundo plano. En la terminal, ejecuta de nuevo:

```bash
streamlit run app.py
```

-----

## 🎯 ¿Qué puedes hacer ahora? (Próximos Pasos)

✔ **Cambiar el Rol:** Modifica el `SystemMessage` para especializar al chatbot.
✔ **Agregar Contexto:** Investiga cómo usar LangChain con **RAG** para que el chatbot pueda leer **documentos internos** de RRHH.
✔ **Ajustar el Modelo:** Experimenta cambiando el valor de `temperature` para un tono más formal o más creativo.

-----
