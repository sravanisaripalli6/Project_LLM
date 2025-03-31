import os
import pytesseract
import re
import pandas as pd
from pdf2image import convert_from_path
import googlemaps
from datetime import datetime
#from transformers import GPT2LMHeadModel, GPT2Tokenizer
#import torch

# Set up Google Maps API client (replace with your actual key)
gmaps = googlemaps.Client(key="Google-API-Key")

# Folder containing PDFs
pdf_folder = "Path/to/pdfs/credit_card_statement/"

# Initialize empty list to store all transactions
all_transactions = []

# Loop through all PDFs in the folder
for pdf_file in os.listdir(pdf_folder):
    if pdf_file.endswith(".pdf"):  # Process only PDF files
        pdf_path = os.path.join(pdf_folder, pdf_file)
        
        # Convert PDF to images
        pages = convert_from_path(pdf_path, dpi=300)

        # Extract text from images
        text = ""
        for page in pages:
            text += pytesseract.image_to_string(page) + "\n"

        # Define regex pattern for extracting transactions
        transaction_pattern = r"(\w{3} \d{1,2})\s+(\w{3} \d{1,2})\s+(.+?)\s+\$([\d,]+\.\d{2})"

        # Extract matches
        transactions = re.findall(transaction_pattern, text)

        # Append extracted transactions to list
        all_transactions.extend(transactions)

# Convert all transactions into a DataFrame
df = pd.DataFrame(all_transactions, columns=["Transaction Date", "Post Date", "Vendor", "Amount"])

# Convert Amount column to float and remove commas
df["Amount"] = df["Amount"].str.replace(",", "").astype(float)

# Get the current year dynamically
current_year = datetime.now().year

# Convert the 'Transaction Date' to datetime format for easy grouping by month
df["Transaction Date"] = pd.to_datetime(df["Transaction Date"] + f" {current_year}", format="%b %d %Y")

# Add a Month column
df["Month"] = df["Transaction Date"].dt.strftime("%B")  # Full month name

class TransactionChatbot:
    def __init__(self, transaction_df):
        self.df = transaction_df
        
        # Ensure no missing values in grouped data
        self.df.fillna({"Vendor": "Unknown"}, inplace=True)

        # Create useful pre-computed data views
        self.vendor_totals = self.df.groupby("Vendor")["Amount"].sum().to_dict()
        self.monthly_totals = self.df.groupby("Month")["Amount"].sum().to_dict()
        self.vendor_monthly = self.df.groupby(["Vendor", "Month"])["Amount"].sum().to_dict()
        
        # Calculate some basic stats
        self.total_spent = self.df["Amount"].sum()
        self.avg_transaction = self.df["Amount"].mean()
        self.max_transaction = self.df["Amount"].max()
        self.transaction_count = len(self.df)
        
        # Common patterns to recognize in questions
        self.patterns = [
            (r"spend (?:at|on|in) ([\w\s]+)", self.handle_vendor_spend),
            (r"([\w\s]+) spend", self.handle_vendor_spend),
            (r"total (?:at|on|in) ([\w\s]+)", self.handle_vendor_spend),
            (r"spend (?:in|during) (january|february|march|april|may|june|july|august|september|october|november|december)", self.handle_month_spend),
            (r"(january|february|march|april|may|june|july|august|september|october|november|december) spend", self.handle_month_spend),
            (r"average", self.handle_average),
            (r"total spend", self.handle_total),
            (r"most spend", self.handle_most_spent),
            (r"highest", self.handle_most_spent),
        ]
    
    def handle_vendor_spend(self, match, query):
        vendor_name = match.group(1).strip().title()
        
        # Find closest matching vendor
        matching_vendors = [v for v in self.vendor_totals.keys() if vendor_name.lower() in v.lower()]
        
        if matching_vendors:
            vendor = matching_vendors[0]
            amount = self.vendor_totals[vendor]
            return f"You spent ${amount:.2f} at {vendor}."
        else:
            return f"I couldn't find any transactions for {vendor_name}."
    
    def handle_month_spend(self, match, query):
        month = match.group(1).strip().title()
        if month in self.monthly_totals:
            amount = self.monthly_totals[month]
            return f"In {month}, you spent a total of ${amount:.2f}."
        else:
            return f"I couldn't find any transactions for {month}."
    
    def handle_average(self, match, query):
        if "transaction" in query.lower():
            return f"Your average transaction amount is ${self.avg_transaction:.2f}."
        else:
            monthly_avg = self.total_spent / len(self.monthly_totals)
            return f"Your average monthly spending is ${monthly_avg:.2f}."
    
    def handle_total(self, match, query):
        return f"Your total spending is ${self.total_spent:.2f} across {self.transaction_count} transactions."
    
    def handle_most_spent(self, match, query):
        if "month" in query.lower():
            top_month = max(self.monthly_totals.items(), key=lambda x: x[1])
            return f"Your highest spending month was {top_month[0]} with ${top_month[1]:.2f}."
        else:
            top_vendor = max(self.vendor_totals.items(), key=lambda x: x[1])
            return f"Your highest spending category was {top_vendor[0]} with ${top_vendor[1]:.2f}."
    
    def answer_query(self, query):
        query = query.lower().strip()

        # Check for exact vendor queries
        for vendor in self.vendor_totals.keys():
            if vendor.lower() == query.lower():
                return f"You spent ${self.vendor_totals[vendor]:.2f} at {vendor}."

        # Match against patterns
        for pattern, handler in self.patterns:
            match = re.search(pattern, query, re.IGNORECASE)
            if match:
                return handler(match, query)

        # Default response
        return "I can answer questions about your spending. Try asking 'How much did I spend at Amazon?' or 'What was my total spending in March?'"

# Initialize chatbot
chatbot = TransactionChatbot(df)

# Start interactive chat
while True:
    try:
        user_input = input("\nAsk about your transactions: ")
        if user_input.lower() in ["exit", "quit", "q"]:
            print("Exiting chat. Goodbye!")
            break
            
        response = chatbot.answer_query(user_input)
        print(f"\nResponse: {response}")
        
    except KeyboardInterrupt:
        print("\nExiting chat. Goodbye!")
        break
