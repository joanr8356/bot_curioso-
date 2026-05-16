# bot_curioso-
import discord
import requests
import pyttsx3
from deep_translator import GoogleTranslator
 
TOKEN = ""
FACTS_API = "https://uselessfacts.jsph.pl/random.json?language=en"
 
intents = discord.Intents.default()
intents.message_content = True
client = discord.Client(intents=intents)

 
def translate_to_spanish(text: str) -> str:
    return GoogleTranslator(source="en", target="es").translate(text)
 
def speak(text: str) -> None:
    engine = pyttsx3.init()
    engine.setProperty("rate", 160)
    voices = engine.getProperty("voices")
    for voice in voices:
        if "spanish" in voice.name.lower() or "es" in voice.id.lower():
            engine.setProperty("voice", voice.id)
            break
    engine.say(text)
    engine.runAndWait()
 
def get_fact() -> str:
    response = requests.get(FACTS_API, timeout=10)
    print(response.json())
    if response.status_code != 200:
        return f"⚠️ No pude obtener un hecho en este momento (código HTTP {response.status_code}). Intenta más tarde."
    data = response.json()
    fact_en = data.get("text", "⚠️ La API no devolvió ningún hecho.")
    return translate_to_spanish(fact_en)
 
@client.event
async def on_ready():
    print(f"✅ Bot conectado como {client.user}")
 
@client.event
async def on_message(message: discord.Message):
    if message.author == client.user:
        return
    if message.content.strip() == "!start":
        await message.channel.send(f"👋 ¡Hola, {message.author.display_name}! Escribe **!fact** para recibir un dato interesante. 🤓")
    elif message.content.strip() == "!fact":
        fact = get_fact()
        await message.channel.send(f"🧠 **Dato curioso:** {fact}")
        speak(fact)
 
client.run(TOKEN)
