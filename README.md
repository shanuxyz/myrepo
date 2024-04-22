import streamlit as st
import requests
from textblob import TextBlob  # For sentiment analysis

# Google Custom Search API Key and search engine ID
API_KEY = 'AIzaSyA2xGfWbkKexgWse4GGIQsq4idAU_tkoPA'
SEARCH_ENGINE_ID = '5740085756d1c4498'


# Function to search the web using Google Custom Search API
def search_web(query):
    url = f'https://www.googleapis.com/customsearch/v1?q={query}&key={API_KEY}&cx={SEARCH_ENGINE_ID}'
    response = requests.get(url)
    data = response.json()
    if 'items' in data:
        search_results = data['items']
        snippets = [result['snippet'] for result in search_results]
        return snippets[:10]  # Return at most 10 snippets
    else:
        return ["Sorry, I couldn't find any relevant information."]

# Function to analyze sentiment of text using TextBlob
def analyze_sentiment(text):
    blob = TextBlob(text)
    sentiment_score = blob.sentiment.polarity
    if sentiment_score > 0:
        return "positive"
    elif sentiment_score < 0:
        return "negative"
    else:
        return "neutral"

# Streamlit app
def main():
    # Page title and description
    st.title("Welcome to Chatbot")
    st.markdown("""
    I'm a Chatbot. I can help you with some information.
    If you want to exit, type 'bye'.
    """)

    # Sidebar options
    st.sidebar.title("Options")
    function = st.sidebar.selectbox("Select Functionality", ["Chat", "Sentiment Analysis"])

    # Main content based on selected functionality
    if function == "Chat":
        # Initialize the chat
        st.subheader("Chat")
        st.write("Let's chat!")
        user_input = st.text_input("You:", "").strip()

        # Main loop for chatting
        chat_history = []
        while user_input.lower() not in ("quit", "exit", "bye"):
            if not user_input:
                continue
            elif user_input.lower() in ("quit", "exit", "bye"):
                st.write("Chatbot: Bye! Take care.")
            else:
                with st.spinner('Searching...'):
                    search_results = search_web(user_input)
                st.write("Chatbot (Web Search):")
                for i, snippet in enumerate(search_results, start=1):
                    st.write(f"{i}. {snippet}")
                    chat_history.append(("Chatbot:", snippet))

            # Continue conversation
            user_input = st.text_input("You:", "").strip()

        # Display chat history
        st.subheader("Chat History")
        for chat in chat_history:
            st.write(chat[0], chat[1])

    elif function == "Sentiment Analysis":
        # Sentiment analysis functionality
        st.subheader("Sentiment Analysis")
        text_input = st.text_area("Enter text for sentiment analysis:", "")
        if text_input:
            sentiment = analyze_sentiment(text_input)
            st.write("Sentiment:", sentiment)

if __name__ == "__main__":
    main()
