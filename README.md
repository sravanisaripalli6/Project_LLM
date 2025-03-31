# Credit Card Transaction Summarization

# Overview
This project extracts, categorizes, and analyzes credit card transactions from PDF statements using OCR and natural language processing (NLP). Users can query their spending habits through a chatbot interface powered by a language model.

# Features
- Extracts transaction details (date, vendor, amount) from PDF statements using `pytesseract`.
- Categorizes transactions using Named Entity Recognition (NER) and regex.
- Computes monthly and vendor-wise spending summaries.
- Enables users to ask spending-related questions via an LLM-based chatbot.
- Uses Google Maps API for vendor recognition and location-based insights.

# Tech Stack
- **Python** (Core logic and data processing)
- **pytesseract** (OCR for text extraction from PDFs)
- **pdf2image** (Convert PDF statements to images)
- **pandas** (Data cleaning and transformation)
- **Transformers (GPT-2)** (Language model for chatbot interaction)
- **Google Maps API** (Vendor categorization and location details)

# Installation
# Prerequisites
Ensure you have the following installed:
- Python 3.x
- pip
- Git

# Setup Instructions
1. **Clone the Repository**
   ```sh
   git clone <repository_url>
   cd credit_card_summarization
   ```
2. **Install Dependencies**
   ```sh
   pip install -r requirements.txt
   ```
3. **Set up Google Maps API**
   - Replace `'my_api'` in `googlemaps.Client(key='my_api')` with your actual API key.
4. **Run the Application**
   ```sh
   python credit_card_summarization.py
   ```

# Usage
- Place your PDF statements inside the specified folder.
- Run the script to extract and categorize transactions.
- Interact with the chatbot by asking spending-related questions, such as:
  - *"How much did I spend at Starbucks?"*
  - *"What was my total spending in January?"*

# Future Enhancements
- Enhance NLP accuracy with fine-tuned transformer models.
- Improve vendor classification with a machine learning-based approach.
- Implement a web-based UI for better accessibility.



