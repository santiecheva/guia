La estructura de la guía será:

1. **Instalar herramientas básicas (Windows).**
2. **Crear un entorno virtual y preparar el proyecto.**
3. **Construir la interfaz con Streamlit.**
4. **Crear el modelo de IA con LangChain.**
5. **Conectar la interfaz con el modelo para tener un chatbot funcional.**

---

# 🧠 **GUÍA COMPLETA PARA CREAR UN CHATBOT CON LANGCHAIN + STREAMLIT**

---

---

# 1. 🛠️ **Instalación de Herramientas en Windows**

Tranquilo/a: estos pasos son más técnicos, pero solo se hacen **una vez**.

---

## 1.1. **Instalar Python**

1. Entra a: [https://www.python.org/downloads/windows/](https://www.python.org/downloads/windows/)
2. Descarga la versión recomendada (normalmente aparece como *“Download Python 3.x.x”*).
3. Abre el instalador.
4. **IMPORTANTE**: marca la casilla
   ✔️ **"Add Python to PATH"**
5. Haz clic en **Install Now**.
6. Espera 1–2 minutos a que termine.

Para verificar que quedó bien instalado:

* Abre el **Símbolo del sistema** (Windows + buscar “cmd”).
* Escribe:

```
python --version
```

Debe aparecer algo como `Python 3.x.x`.


---
<img width="1160" height="581" alt="image" src="https://github.com/user-attachments/assets/8433998c-ac35-4576-bc31-5c28e9666016" />


## 1.2. **Instalar Visual Studio Code (VSCode)**

VSCode será tu "editor" para escribir el proyecto.

1. Ir a: [https://code.visualstudio.com/](https://code.visualstudio.com/)
2. Descargar versión para Windows.
3. Instalar con opciones por defecto.
4. Abrir VSCode al final.
<img width="1600" height="863" alt="image" src="https://github.com/user-attachments/assets/2c346a31-492b-44ea-bacd-8beed3759125" />

---

# 2. 🧪 **Crear un Entorno Virtual (Recomendado)**

Un **entorno virtual** es una carpeta que guarda todas las librerías que tu chatbot va a usar.
Así no daña otros programas del computador.

---

## 2.1. Crear una carpeta de trabajo

Crea una carpeta en tu escritorio:

📁 `C:\Users\tu_usuario\Desktop\chatbot-rh`

<img width="212" height="198" alt="image" src="https://github.com/user-attachments/assets/e8237e0b-464d-4525-b3c8-7a2e122760cd" />



---

## 2.2. Abrir VSCode en esa carpeta

1. Abre VSCode.
2. Arrastra la carpeta dentro del VSCode.
   (o usa **File > Open Folder**)

<img width="1600" height="861" alt="image" src="https://github.com/user-attachments/assets/e312ddb2-be49-4379-a1fa-1758e127cdf0" />


---

## 2.3. Crear un entorno virtual

Dentro de VSCode abre la terminal:
**Terminal > New Terminal**

Escribe:

```
python -m venv venv
```

Esto creará una carpeta llamada **venv**.

<img width="1275" height="978" alt="image" src="https://github.com/user-attachments/assets/da399180-eb60-404c-8d88-f5aa1f13827f" />

---

## 2.4. Activar el entorno

En la misma terminal escribe:

```
venv\Scripts\activate.bat
```
<img width="1292" height="957" alt="image" src="https://github.com/user-attachments/assets/80fe2f48-97ca-4c9c-ac57-6f6b46637ba7" />


si tienes algùn error, por ejemplo un mensaje en rojo, ejecuta el comando así:

```
venv\Scripts\activate
```

Si funciona, verás algo así:

```
(venv) C:\Users\...
```

---

---

# 3. 📦 **Instalar las Librerías Necesarias**

Con el entorno activado, instala:

<img width="1448" height="981" alt="image" src="https://github.com/user-attachments/assets/7af4e76f-3393-4161-8759-0bfdb1ee5020" />


```
pip install streamlit langchain langchain-ollama python-dotenv ollama

```



---

---

# 4. 🎨 **Crear la Interfaz con Streamlit (PASO 1)**

Vamos a crear primero **lo que las personas verán**, antes del cerebro del chatbot.

En VSCode dentro de la carpeta que estàs trabajando,  crea un archivo llamado:

📄 `app.py`


<img width="1600" height="855" alt="image" src="https://github.com/user-attachments/assets/9cec2cc1-6e8b-4944-a96d-ddd97c2ae70a" />


Y pega esto:

```python
import streamlit as st

# 🎨 Configuración básica de la página
st.set_page_config(
    page_title="Chatbot RH",
    page_icon="🤖",
    layout="centered",
    initial_sidebar_state="collapsed"
)

# 🧼 Un poco de CSS para que se vea más pro (tipo chat)
st.markdown(
    """
    <style>
    /* Fondo suave tipo app moderna */
    .stApp {
        background: linear-gradient(135deg, #f3f4ff 0%, #eef9ff 50%, #fff8ff 100%);
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

    /* Título centrado */
    h1 {
        text-align: center;
    }
    </style>
    """,
    unsafe_allow_html=True
)

# 🧠 Estado de la conversación
if "messages" not in st.session_state:
    st.session_state.messages = [
        {
            "role": "assistant",
            "content": (
                "👋 ¡Hola! Soy tu *Chatbot de Recursos Humanos*.\n\n"
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

# 🎯 Sidebar con información extra (opcional)
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
    st.write("💡 *Tip:* prueba con preguntas como:")
    st.code("¿Cuántos días de vacaciones tengo al año?")
    st.code("¿Cómo es el proceso de onboarding?")

# 🤹 Función de respuesta DEMO (luego aquí conectas tu modelo real)
def generar_respuesta_demo(pregunta: str) -> str:
    # Esta respuesta es solo para que el chat se vea vivo.
    # Aquí luego puedes llamar a tu modelo con LangChain + Ollama.
    respuesta = f"""
He recibido tu pregunta:

> **{pregunta}**

🔍 *Versión demo:* aquí iría la respuesta inteligente del modelo de IA.

Como soy un bot de RRHH en modo demostración, te puedo decir algo general:

- Revisaría las políticas internas relacionadas con este tema.
- Te daría una explicación clara y en lenguaje sencillo.
- Si aplica, te indicaría con quién podrías hablar en RRHH para más detalle.

Mientras conectas el modelo real, podemos imaginar que ya soy súper listo 😄
"""
    return respuesta

# 💬 Mostrar historial de mensajes
for msg in st.session_state.messages:
    avatar = "🤖" if msg["role"] == "assistant" else "🧑‍💼"
    with st.chat_message("assistant" if msg["role"] == "assistant" else "user", avatar=avatar):
        st.markdown(msg["content"])

# 🧾 Caja de entrada tipo ChatGPT
prompt = st.chat_input("Escribe tu pregunta sobre Recursos Humanos aquí...")

if prompt:
    # 1) Agregamos el mensaje del usuario al historial
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user", avatar="🧑‍💼"):
        st.markdown(prompt)

    # 2) Generamos la respuesta (demo o modelo real)
    with st.chat_message("assistant", avatar="🤖"):
        with st.spinner("Pensando la mejor respuesta para ti... 💭"):
            respuesta = generar_respuesta_demo(prompt)
            st.markdown(respuesta)
    st.session_state.messages.append({"role": "assistant", "content": respuesta})

```

<img width="1267" height="973" alt="image" src="https://github.com/user-attachments/assets/80d5d110-5084-49db-99b2-9204e888775a" />


Para probar la interfaz:

<img width="1127" height="976" alt="image" src="https://github.com/user-attachments/assets/bbc7ec39-8b19-4b73-a7e1-325484954e64" />


En la terminal (con el entorno activado):

```
streamlit run app.py
```

Se abrirá un navegador con tu interfaz ✨

<img width="1600" height="729" alt="image" src="https://github.com/user-attachments/assets/f4f94a38-1801-42d0-a227-26cef4e832f5" />

---


# 5. 🐪 **Instalar Ollama y Descargar el Modelo (ANTES DE CONECTAR EL CHATBOT)**

Ollama es el programa que permite tener modelos de IA **locales y gratuitos** en tu computador.
Piensa en Ollama como “la fábrica de cerebros” donde vas a descargar el modelo que usará el chatbot.

---

## 5.1. Descargar e instalar Ollama

1. Ve a: [https://ollama.com](https://ollama.com)
2. Haz clic en **Download for Windows**.
3. Abre el instalador (`Ollama Setup.exe`) y sigue los pasos por defecto (Siguiente, Siguiente…).
4. Cuando termine, Ollama quedará instalado y normalmente se ejecuta solo en segundo plano.

<img width="1600" height="855" alt="image" src="https://github.com/user-attachments/assets/05787bd3-347a-4379-b1b3-5c25c5fd781c" />
<img width="1600" height="829" alt="image" src="https://github.com/user-attachments/assets/b9374ede-5340-433a-8001-a798f3bee654" />

<img width="872" height="661" alt="image" src="https://github.com/user-attachments/assets/330bdabe-bbbf-49af-87d4-8cbf80134577" />

---

## 5.2. Verificar que Ollama funciona

1. Abre el **Símbolo del sistema** (cmd) o una terminal nueva.
2. Escribe:

```bash
ollama --version
```

Si todo está bien, verás un número de versión (por ejemplo `0.3.x`).
Si ves “command not found” o similar, asegúrate de cerrar y abrir de nuevo la terminal o reiniciar el equipo.

---

## 5.3. Descargar (hacer *pull*) del modelo `llama3.1`

Ahora vamos a **bajar el modelo de IA** que usará el chatbot.
En la misma terminal escribe:

```bash
ollama pull llama3.1
```

* La **primera vez** puede tardar varios minutos (está descargando el modelo).
* Solo tienes que hacerlo **una vez**. Después ya queda guardado.

---

## 5.4. Probar el modelo rápidamente (opcional)

Solo para ver que funciona:

```bash
ollama run llama3.1
```

Escribe algo como:

> Hola, ¿qué puedes hacer?

Te responderá en la misma terminal.
Para salir, presiona **Ctrl + C**.

Con esto ya tienes:

✅ Ollama instalado
✅ Modelo `llama3.1` descargado y listo para usar

Ahora sí, vamos a conectar **Streamlit + LangChain + Ollama**.

---

# 6. 🔌 **Conectar la Interfaz con el Modelo (PASO FINAL)**

Ahora vamos a unir todo en **un solo archivo `app.py`**:

* La interfaz (Streamlit).
* El modelo (Ollama vía LangChain).
* La lógica para responder preguntas.

Abre `app.py` y **reemplaza todo el contenido** por este:

```python
import streamlit as st
from langchain_ollama import ChatOllama

# 1. Configuración de la página
st.set_page_config(page_title="Chatbot RH", page_icon="🤖")

st.title("🤖 Chatbot para Recursos Humanos")
st.write("Haz tus preguntas sobre procesos de talento humano, cultura, bienestar, etc.")

# 2. Función para cargar el modelo de Ollama
@st.cache_resource
def cargar_modelo():
    # Modelo local y gratuito usando Ollama
    modelo = ChatOllama(
        model="llama3.1",   # nombre del modelo que bajaste con `ollama pull`
        temperature=0.2     # qué tan creativo es (0 = muy serio, 1 = muy creativo)
    )
    return modelo

# 3. Función que envía la pregunta al modelo
def responder_pregunta(pregunta: str) -> str:
    modelo = cargar_modelo()
    respuesta = modelo.invoke(pregunta)
    return respuesta.content

# 4. Interfaz de Streamlit
pregunta = st.text_input("✍️ Escribe tu pregunta aquí:")

if st.button("Enviar"):
    if pregunta.strip() == "":
        st.warning("Por favor escribe una pregunta.")
    else:
        with st.spinner("Pensando la mejor respuesta..."):
            respuesta = responder_pregunta(pregunta)
        st.success("Respuesta del chatbot:")
        st.write(respuesta)
```

Puntos clave para explicar a alguien de RRHH:

* `cargar_modelo()` 👉 es donde definimos qué modelo usamos (`llama3.1`).
* `responder_pregunta()` 👉 es la función que envía la pregunta al modelo y devuelve la respuesta.
* La parte de **Streamlit** (abajo) muestra el cuadro de texto, el botón y la respuesta.

---

# 7. ▶️ **Ejecutar el chatbot**

En la terminal escribe:

```
streamlit run app.py
```

Y tendrás un chatbot funcional:

* bonito
* usable
* pensado para personas sin conocimientos técnicos
* funcionando con LangChain + OpenAI
* con interfaz sencilla en Streamlit

---

# 8. 🎯 ¿Qué puedes hacer ahora?

✔ Cambiar el rol del chatbot (RRHH, bienestar, cultura, selección).
✔ Agregar contexto (manual del empleado, políticas internas).
✔ Guardar conversaciones.
✔ Convertirlo en una app web.
✔ Integrarlo en Teams o Slack.


---






