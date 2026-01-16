import speech_recognition as sr
import pyttsx3
import datetime
import requests
import webbrowser
import random
import time
from urllib.parse import quote

# ============== Configuration ==============
WAKE_WORD = "home"

class HomeAI:
    def __init__(self):
        # 1. Initialize Text-to-Speech Engine
        self.engine = pyttsx3.init()
        self.engine.setProperty('rate', 175)
        self.engine.setProperty('volume', 1.0)
        
        # 2. Initialize Speech Recognition
        self.recognizer = sr.Recognizer()
        self.is_running = True
        
        print("✅ Home AI: Online and Ready.")
        print(f"Listening for the wake word: '{WAKE_WORD}'")

    # --- Core Audio Logic ---
    def speak(self, text):
        """AI speaks the answer and waits until finished."""
        print(f"🤖 AI: {text}")
        # Stop any current speech to clear the buffer
        self.engine.stop()
        self.engine.say(text)
        self.engine.runAndWait()
        # Small delay to let the audio driver reset
        time.sleep(0.4)

    def listen(self):
        """Listens for voice and converts to text."""
        with sr.Microphone() as source:
            self.recognizer.adjust_for_ambient_noise(source, duration=0.6)
            try:
                print("🎤 Listening...")
                audio = self.recognizer.listen(source, timeout=5, phrase_time_limit=5)
                command = self.recognizer.recognize_google(audio).lower()
                print(f"📝 You said: {command}")
                return command
            except:
                return ""

    # --- Feature Methods ---
    def cmd_knowledge(self, query):
        """Fetches facts and reads them aloud."""
        self.speak(f"Looking up {query}...")
        try:
            url = f"https://api.duckduckgo.com/?q={quote(query)}&format=json&no_html=1"
            res = requests.get(url).json()
            answer = res.get("AbstractText")
            
            if answer:
                self.speak(answer)
            else:
                self.speak(f"I couldn't find a summary, opening a web search for {query} instead.")
                webbrowser.open(f"https://www.google.com/search?q={query}")
        except:
            self.speak("I had trouble connecting to my knowledge base.")

    def cmd_calculate(self, text):
        """Processes math words and speaks the result."""
        expression = text.replace("calculate", "").replace("plus", "+").replace("minus", "-").replace("times", "*").replace("multiplied by", "*").replace("divided by", "/")
        # Keep only numbers and operators
        clean_expr = "".join(c for c in expression if c in "0123456789+-*/. ")
        try:
            result = eval(clean_expr.strip())
            self.speak(f"The answer is {result}")
        except:
            self.speak("I couldn't calculate that math problem.")

    # --- Brain Logic ---
    def process_command(self, text):
        """Finds the right feature based on keywords."""
        
        if "time" in text:
            now = datetime.datetime.now().strftime("%I:%M %p")
            self.speak(f"The current time is {now}")

        elif "date" in text:
            today = datetime.date.today().strftime("%A, %B %d, %Y")
            self.speak(f"Today is {today}")

        elif "calculate" in text or "plus" in text or "times" in text:
            self.cmd_calculate(text)

        elif "who is" in text or "what is" in text:
            query = text.replace("who is", "").replace("what is", "").strip()
            self.cmd_knowledge(query)

        elif "search" in text:
            query = text.replace("search", "").replace("for", "").strip()
            self.speak(f"Searching Google for {query}")
            webbrowser.open(f"https://www.google.com/search?q={query}")

        elif "joke" in text:
            jokes = [
                "Why don't scientists trust atoms? Because they make up everything!",
                "What do you call a fake noodle? An impasta!",
                "Why did the computer go to the doctor? Because it had a virus!"
            ]
            self.speak(random.choice(jokes))

        elif "stop" in text or "exit" in text or "goodbye" in text:
            self.speak("System shutting down. Goodbye!")
            self.is_running = False

    # --- Main Loop ---
    def run(self):
        self.speak("Home AI is active.")
        while self.is_running:
            voice_input = self.listen()
            
            if WAKE_WORD in voice_input:
                # Remove the wake word to see if a command was attached
                command = voice_input.replace(WAKE_WORD, "").strip()
                
                if not command:
                    self.speak("Yes? How can I help?")
                    command = self.listen()
                
                if command:
                    self.process_command(command)

if __name__ == "__main__":
    try:
        ai = HomeAI()
        ai.run()
    except Exception as e:
        print(f"Fatal error: {e}")
