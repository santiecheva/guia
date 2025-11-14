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
pip install streamlit langchain langchain-openai python-dotenv
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


---

