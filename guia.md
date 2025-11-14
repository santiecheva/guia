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

---

## 2.4. Activar el entorno

En la misma terminal escribe:

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

```
pip install streamlit langchain langchain-openai python-dotenv
```

Si deseas usar **OpenAI**, instala también:

```
pip install openai
```

---

---

# 4. 🎨 **Crear la Interfaz con Streamlit (PASO 1)**

Vamos a crear primero **lo que las personas verán**, antes del cerebro del chatbot.

En VSCode crea un archivo llamado:

📄 `app.py`

Y pega esto:

```python
import streamlit as st

st.set_page_config(page_title="Chatbot RH", page_icon="🤖")

st.title("🤖 Chatbot para Recursos Humanos")

# Área donde el usuario escribe su pregunta
pregunta = st.text_input("Escribe tu pregunta aquí:")

# Botón
if st.button("Enviar"):
    if pregunta.strip() == "":
        st.warning("Por favor escribe una pregunta.")
    else:
        st.write("💬 Respuesta del modelo aparecerá aquí...")
```

Para probar la interfaz:

En la terminal (con el entorno activado):

```
streamlit run app.py
```

Se abrirá un navegador con tu interfaz ✨

---

---

# 5. 🧠 **Crear el Modelo con LangChain (PASO 2)**

Ahora crearemos “el cerebro” del chatbot.

Crea un archivo:

📄 `modelo.py`

Y pega esto:

```python
import os
from langchain_openai import ChatOpenAI

def cargar_modelo():
    # Modelo base
    modelo = ChatOpenAI(
        model="gpt-4o-mini",
        temperature=0.2
    )
    return modelo

def responder_pregunta(pregunta):
    modelo = cargar_modelo()

    respuesta = modelo.invoke(pregunta)

    return respuesta.content
```

IMPORTANTE:
Crea un archivo **.env** con tu API key de OpenAI:

📄 `.env`

```
OPENAI_API_KEY=tu_api_key
```

*Tu API Key se consigue en [https://platform.openai.com](https://platform.openai.com)*

---

---

# 6. 🔌 **Conectar la Interfaz con el Modelo (PASO FINAL)**

Ahora vuelve a abrir `app.py`
y cámbialo por esta versión:

```python
from langchain_ollama import ChatOllama

def cargar_modelo():
    # Modelo local y gratuito usando Ollama
    modelo = ChatOllama(
        model="llama3.1",   # nombre del modelo que bajaste con `ollama pull`
        temperature=0.2
    )
    return modelo

def responder_pregunta(pregunta: str) -> str:
    modelo = cargar_modelo()

    # Igual que antes, invoke devuelve un mensaje con .content
    respuesta = modelo.invoke(pregunta)

    return respuesta.content
```

---

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

Si quieres, te puedo ayudar a:

👉 agregar memoria
👉 agregar documentos PDF para que responda sobre ellos
👉 hacerlo multilingüe
👉 ponerlo en la nube (AWS, Azure o GCP)

---

