import speech_recognition as sr
import pyttsx3
import datetime
import requests
import wikipedia
import webbrowser
import time
from googlesearch import search

# ====== SET YOUR API KEYS HERE ======
WEATHER_API_KEY = "YOUR_OPENWEATHERMAP_API_KEY"  # <-- Replace with your OpenWeatherMap API key
NEWS_API_KEY = "YOUR_NEWSAPI_API_KEY"            # <-- Replace with your NewsAPI API key

# ====================================

engine = pyttsx3.init()
recognizer = sr.Recognizer()

def speak(text):
    try:
        print("Assistant:", text)
    except UnicodeEncodeError:
        print("Assistant:", text.encode('utf-8', errors='replace').decode('utf-8'))
    engine.say(text)
    engine.runAndWait()

def get_command():
    with sr.Microphone() as source:
        print("Listening...")
        audio = recognizer.listen(source)
        try:
            command = recognizer.recognize_google(audio)
            print("You said:", command)
            return command.lower()
        except:
            speak("Sorry, I didn't catch that.")
            return ""

def get_weather(city):
    url = f"https://api.openweathermap.org/data/2.5/weather?q={city}&appid={WEATHER_API_KEY}&units=metric"
    response = requests.get(url)
    if response.status_code == 200:
        data = response.json()
        weather = data['weather'][0]['description']
        temp = data['main']['temp']
        return f"The weather in {city} is {weather} with a temperature of {temp}°C."
    else:
        return "Sorry, I couldn't fetch the weather information."

def get_location():
    try:
        response = requests.get("https://ipinfo.io")
        data = response.json()
        city = data.get("city", "Unknown")
        region = data.get("region", "Unknown")
        country = data.get("country", "Unknown")
        return f"You are in {city}, {region}, {country}."
    except:
        return "Sorry, I couldn't fetch your location."

def get_news():
    url = f"https://newsapi.org/v2/top-headlines?country=in&apiKey={NEWS_API_KEY}"
    response = requests.get(url)
    if response.status_code == 200:
        data = response.json()
        articles = data.get("articles", [])
        if articles:
            top_articles = articles[:3]
            news_headlines = [article['title'] for article in top_articles]
            return "Here are the top news headlines: " + "; ".join(news_headlines)
        else:
            return "No news found."
    else:
        return "Sorry, I couldn't fetch the news."

def search_wikipedia(query):
    try:
        summary = wikipedia.summary(query, sentences=2)
        return summary
    except:
        return "Sorry, I couldn't find information on Wikipedia."

def google_search(query):
    try:
        results = []
        for url in search(query, num_results=3):
            results.append(url)
        return "Here are the top search results: " + "; ".join(results)
    except:
        return "Sorry, I couldn't perform the Google search."

def open_website(site):
    if not site.startswith("http"):
        if "." not in site:
            site += ".com"
        url = f"https://{site}"
    else:
        url = site
    webbrowser.open(url)
    return f"Opening {site}"

def set_reminder(reminder, seconds):
    speak(f"Reminder set for {seconds} seconds from now.")
    time.sleep(seconds)
    speak(f"Reminder: {reminder}")

def run_assistant():
    speak("Hello! I am your assistant. How can I help you?")
    while True:
        command = get_command()
        if not command:
            continue
        now = datetime.datetime.now()
        if "time" in command:
            time_str = now.strftime('%I:%M %p')
            speak(f"The current time is {time_str}")
        elif "date" in command:
            date_str = now.strftime('%A, %B %d, %Y')
            speak(f"Today's date is {date_str}")
        elif "weather" in command:
            speak("Please say the city name.")
            city = get_command()
            if city:
                speak(get_weather(city))
            else:
                speak("City not recognized.")
        elif "location" in command:
            speak(get_location())
        elif "news" in command:
            speak(get_news())
        elif "wikipedia" in command:
            speak("What should I search on Wikipedia?")
            topic = get_command()
            if topic:
                speak(search_wikipedia(topic))
            else:
                speak("Topic not recognized.")
        elif "search" in command:
            query = command.replace("search", "").strip()
            if not query:
                speak("What should I search for?")
                query = get_command()
            if query:
                speak(google_search(query))
            else:
                speak("Query not recognized.")
        elif "open" in command:
            site = command.replace("open", "").strip()
            if site:
                speak(open_website(site))
            else:
                speak("Which website should I open?")
        elif "remind" in command or "reminder" in command:
            speak("What should I remind you about?")
            reminder = get_command()
            speak("In how many seconds?")
            try:
                seconds = int(get_command())
                set_reminder(reminder, seconds)
            except:
                speak("Could not set reminder.")
        elif "exit" in command or "stop" in command or "quit" in command:
            speak("Goodbye!")
            break
        else:
            speak("I didn't recognize the command. Should I search this on Google or Wikipedia?")
            choice = get_command()
            if "wikipedia" in choice:
                speak(search_wikipedia(command))
            elif "google" in choice:
                speak(google_search(command))
            else:
                speak("Please say 'Google' or 'Wikipedia' to search your query.")

if __name__ == "__main__":
    print("Advanced Assistant started...")
    run_assistant()
