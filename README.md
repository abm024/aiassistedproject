# aiassistedproject
MGT 390
import streamlit as st
import openai
import pyairtable
from datetime import datetime

# Configuration
openai.api_key = "your-api-key"
AIRTABLE_API_KEY = "your-airtable-key"
BASE_ID = "your-base-id"
TABLE_NAME = "ChatInteractions"

# Initialize Airtable
table = pyairtable.Table(AIRTABLE_API_KEY, BASE_ID, TABLE_NAME)

def get_openai_response(prompt):
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": "You are a helpful customer service representative for an e-commerce company."},
            {"role": "user", "content": prompt}
        ]
    )
    return response.choices[0].message.content

def categorize_ticket(prompt):
    categories = ["Shipping", "Returns", "Product Info", "Technical", "Billing"]
    response = openai.ChatCompletion.create(
        model="gpt-3.5-turbo",
        messages=[
            {"role": "system", "content": f"Categorize this customer query into one of these categories: {categories}"},
            {"role": "user", "content": prompt}
        ]
    )
    return response.choices[0].message.content

def log_interaction(query, response, category):
    table.create({
        "Query": query,
        "Response": response,
        "Category": category,
        "Timestamp": datetime.now().isoformat()
    })

# Streamlit Interface
st.title("Customer Service Chatbot")

if "messages" not in st.session_state:
    st.session_state.messages = []

for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

if prompt := st.chat_input("How can I help you today?"):
    st.session_state.messages.append({"role": "user", "content": prompt})
    with st.chat_message("user"):
        st.markdown(prompt)

    # Get response and category
    response = get_openai_response(prompt)
    category = categorize_ticket(prompt)
    
    # Log interaction
    log_interaction(prompt, response, category)
    
    # Display response
    with st.chat_message("assistant"):
        st.markdown(response)
    st.session_state.messages.append({"role": "assistant", "content": response})
