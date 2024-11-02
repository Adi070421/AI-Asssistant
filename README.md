# AI-Asssistant
import pyttsx3
import speech_recognition as sr
import requests
import wikipedia
import pywhatkit as kit
from decouple import config

# Initialize the speech engine
engine = pyttsx3.init('sapi5')
engine.setProperty('rate', 190)
engine.setProperty('volume', 1.0)

def speak(text):
    """Convert text to speech."""
    engine.say(text)
    engine.runAndWait()

def take_user_input():
    """Listen to the user's command and return it as text."""
    r = sr.Recognizer()
    with sr.Microphone() as source:
        print("Listening...")
        audio = r.listen(source)
    try:
        command = r.recognize_google(audio)
        print(f"You said: {command}")
        return command.lower()
    except sr.UnknownValueError:
        speak("Sorry, I did not understand that.")
        return ""
    except sr.RequestError:
        speak("Could not request results from the speech recognition service.")
        return ""

def greet_user():
    """Greet the user based on the time of day."""
    from datetime import datetime
    hour = datetime.now().hour
    if hour < 12:
        speak("Good morning sir!")
    elif hour < 18:
        speak("Good afternoon sir!")
    else:
        speak("Good evening!")
    speak(" how can i assist you today")

def main():
    greet_user()
    while True:
        command = take_user_input()
        if 'exit' in command or 'great my boy' in command:
            speak(" thankyou, have a nice day sir!")
            break
        elif 'play' in command:
            video = command.replace('play', '').strip()
            kit.playonyt(video)
        elif 'search' in command:
            query = command.replace('search', '').strip()
            results = wikipedia.summary(query, sentences=2)
            speak(results)
        elif 'weather' in command:
            city = command.replace('weather in', '').strip()
            api_key = config('OPENWEATHER_APP_ID')  # Add your OpenWeather API key in .env
            weather_data = requests.get(f"http://api.openweathermap.org/data/2.5/weather?q={city}&appid={api_key}&units=metric").json()
            if weather_data['cod'] == 200:
                weather = weather_data['weather'][0]['description']
                temperature = weather_data['main']['temp']
                speak(f"The weather in {city} is {weather} with a temperature of {temperature} degrees Celsius.")
            else:
                speak("City not found.")
        # Add more commands as needed

if __name__ == "__main__":
    main()
