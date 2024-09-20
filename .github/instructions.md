# GitHub Copilot Demo Instructions

## Step 1: Describe the Project

**Prompt:**
I want to create a simple Python application that prompts the user in the console for input of a stock ticker symbol, and then returns the stock price.

**Goal:**
Get Copilot to output some code snippet and description of how to get stock price (Alpha Vantage API).

## Step 2: Modularize the Code

**Prompt:**
I think we should make this modular - can you refactor the code to modularize it and place the logic into separate classes?

**Goal:**
Get Copilot to modularize code, whether in 1 file or 2.

## Step 3: Split into Separate Files

**Prompt:**
Output 2 separate files with code contents to modularize them further.

**Goal:**
2 separate files.

## Step 4: Scaffold a New Python Project

**Prompt:**
@workspace /new scaffold a new python project for me, with a stock_app.py and stock_fetcher.py classes in the directory (replace file names with whatever Copilot called the files).

**Goal:**
New workspace with proper directory structure.

## Step 5: Place Code into Appropriate Files

**Action:**
Now place code into appropriate files. *Make sure to replace code snippet with your API key.*

## Step 5.5: Showcase inline Copilot (Command+i)

**Action:** Command+i in the code editor with a function/snippet of code highlighted. 

**Prompt:** Explain what this code snippet is doing. 

**Goal:** Showcase inline Copilot capabilities with Command+i 


## Step 6: Run the Python Code + Showcase Copilot CLI 

**Prompt:**
How would I run this Python code?

**Goal:**
Output 1 stock price.

*Make sure to replace code snippet with your API key.*
```
pip install -r requirements.txt

cd src

python stock_app.py
```

**Action:** Showcase Copilot CLI by going to terminal and pressing 'Command+i' and ask Copilot how to run the stock_app.py python app. Talk about how you can ask it anything related to the CLI. 

## Step 7: Extend Functionality for Historical Data

**Prompt:**
@workspace #file:stock_fetcher.py #file:stock_app.py (replace file names) how can I extend the functionality to generate the past 7 days worth of stock prices on to the console?

**Goal:**
Historical data output into console.

## Step 8: Explain Functionality

**Prompt:**
@workspace /explain highlighting a file or referencing a file.

**Goal:**
Showcase explain functionality and use case of Copilot Chat.

## Step 9: Run the script with historical data

**Action:** Run the python script. 'python script_name.py' under /src directory 

## Step 10: Secure the Code

**Prompt:**
@workspace #file:stock_fetcher.py #file:stock_app.py can I make this more secure? I don't think we sanitize user inputs.

**Goal:**
Validate ticker symbol input and handle API key securely via environment variable.

## Step 11: Document existing code 

**Prompt:** @workspace #file:stock_fetcher.py can you provide inline comments to document this file. Retain all original code and only add inline comments to help someone not familar with the codebase to understand it. 

**Goal:** Get an output of inline commented file. 

## Step 12: Set API Key as Environment Variable

**Prompt:**
How can I set my API key as an environment variable? I'm on a GitHub Codespace which is Linux under the hood.

**Goal:**
`export ALPHA_VANTAGE_API_KEY=your_api_key_here` to secure API call.

## Step 13: Generate Unit Tests 

**Prompt:**
@workspace create a unit test file for the StockFetcher class. Be sure to include edge cases and provide decent test coverage. 

**Goal:**
Generate a unit test file in its entirety with good number of test cases. 

## Step 14: Showcase Copilot PR Outline, Summary, Code Review and Files Changed Explanation 

**Action:** 
Generate a PR after committing code changes as separate branch. Showcase Copilot PR Outline/Summary feature. 

**Action:** 
Navigate to files changed tab, and showcase Copilot explanations.

**Action:** 
Navigate back to PR comments and ask Copilot to review code. 

**Action:** 
Showcase Copilot Enterprise Chat feature and repository indexing. 


## Step 15: Solution with 1 stock price output
**stock_fetcher.py:**
```
import requests

class StockFetcher:
    def __init__(self, api_key):
        self.api_key = api_key

    def get_stock_price(self, ticker):
        url = f'https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol={ticker}&interval=1min&apikey={self.api_key}'
        response = requests.get(url)
        data = response.json()
        
        try:
            # Extract the latest stock price
            latest_time = list(data['Time Series (1min)'].keys())[0]
            stock_price = data['Time Series (1min)'][latest_time]['1. open']
            return stock_price
        except KeyError:
            return "Invalid ticker symbol or API limit reached."
```
**stock_app.py:**
```
from stock_fetcher import StockFetcher

class StockApp:
    def __init__(self, stock_fetcher):
        self.stock_fetcher = stock_fetcher

    def run(self):
        ticker = input("Enter the stock ticker symbol: ").upper()
        price = self.stock_fetcher.get_stock_price(ticker)
        print(f"The current price of {ticker} is: {price}")

if __name__ == "__main__":
    api_key = 'YOUR_API_KEY'  # Replace with Alpha Vantage API key
    stock_fetcher = StockFetcher(api_key)
    app = StockApp(stock_fetcher)
    app.run()
```


## Step 16: Solution with historical data
**stock_fetcher.py:**
```
import requests
from datetime import datetime, timedelta

class StockFetcher:
    def __init__(self, api_key):
        self.api_key = api_key

    def get_stock_price(self, ticker):
        url = f'https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol={ticker}&interval=1min&apikey={self.api_key}'
        response = requests.get(url)
        data = response.json()
        
        try:
            # Extract the latest stock price
            latest_time = list(data['Time Series (1min)'].keys())[0]
            stock_price = data['Time Series (1min)'][latest_time]['1. open']
            return stock_price
        except KeyError:
            return "Invalid ticker symbol or API limit reached."

    def get_past_week_prices(self, ticker):
        url = f'https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol={ticker}&apikey={self.api_key}'
        response = requests.get(url)
        data = response.json()
        
        try:
            # Extract the past 7 days' stock prices
            time_series = data['Time Series (Daily)']
            past_week_prices = {}
            for i in range(7):
                date = (datetime.now() - timedelta(days=i)).strftime('%Y-%m-%d')
                if date in time_series:
                    past_week_prices[date] = time_series[date]['1. open']
            return past_week_prices
        except KeyError:
            return "Invalid ticker symbol or API limit reached."
```

**stock_app.py:**
```
from stock_fetcher import StockFetcher

class StockApp:
    def __init__(self, stock_fetcher):
        self.stock_fetcher = stock_fetcher

    def run(self):
        ticker = input("Enter the stock ticker symbol: ").upper()
        price = self.stock_fetcher.get_stock_price(ticker)
        print(f"The current price of {ticker} is: {price}")

        past_week_prices = self.stock_fetcher.get_past_week_prices(ticker)
        if isinstance(past_week_prices, dict):
            print(f"The past week's prices for {ticker} are:")
            for date, price in past_week_prices.items():
                print(f"{date}: {price}")
        else:
            print(past_week_prices)

if __name__ == "__main__":
    api_key = 'YOUR_API_KEY'  # Replace with Alpha Vantage API key
    stock_fetcher = StockFetcher(api_key)
    app = StockApp(stock_fetcher)
    app.run()
```

## Step 17: Solution with sanitized code input 

**stock_fetcher.py:**
```
import requests
from datetime import datetime, timedelta

class StockFetcher:
    def __init__(self, api_key):
        self.api_key = api_key

    def get_stock_price(self, ticker):
        url = f'https://www.alphavantage.co/query?function=TIME_SERIES_INTRADAY&symbol={ticker}&interval=1min&apikey={self.api_key}'
        response = requests.get(url)
        data = response.json()
        
        try:
            # Extract the latest stock price
            latest_time = list(data['Time Series (1min)'].keys())[0]
            stock_price = data['Time Series (1min)'][latest_time]['1. open']
            return stock_price
        except KeyError:
            return "Invalid ticker symbol or API limit reached."

    def get_past_week_prices(self, ticker):
        url = f'https://www.alphavantage.co/query?function=TIME_SERIES_DAILY&symbol={ticker}&apikey={self.api_key}'
        response = requests.get(url)
        data = response.json()
        
        try:
            # Extract the past 7 days' stock prices
            time_series = data['Time Series (Daily)']
            past_week_prices = {}
            for i in range(7):
                date = (datetime.now() - timedelta(days=i)).strftime('%Y-%m-%d')
                if date in time_series:
                    past_week_prices[date] = time_series[date]['1. open']
            return past_week_prices
        except KeyError:
            return "Invalid ticker symbol or API limit reached."
```



**stock_app.py:** 
```
import os
import re
from stock_fetcher import StockFetcher

class StockApp:
    def __init__(self, stock_fetcher):
        self.stock_fetcher = stock_fetcher

    def validate_ticker(self, ticker):
        # Ticker symbols are typically 1-5 uppercase letters
        return re.match(r'^[A-Z]{1,5}$', ticker) is not None

    def run(self):
        ticker = input("Enter the stock ticker symbol: ").upper().strip()
        if not self.validate_ticker(ticker):
            print("Invalid ticker symbol. Please enter a valid ticker symbol (1-5 uppercase letters).")
            return

        price = self.stock_fetcher.get_stock_price(ticker)
        print(f"The current price of {ticker} is: {price}")

        past_week_prices = self.stock_fetcher.get_past_week_prices(ticker)
        if isinstance(past_week_prices, dict):
            print(f"The past week's prices for {ticker} are:")
            for date, price in past_week_prices.items():
                print(f"{date}: {price}")
        else:
            print(past_week_prices)

if __name__ == "__main__":
    api_key = os.getenv('ALPHA_VANTAGE_API_KEY')  # Get API key from environment variable
    if not api_key:
        print("API key not found. Please set the ALPHA_VANTAGE_API_KEY environment variable.")
        exit(1)

    stock_fetcher = StockFetcher(api_key)
    app = StockApp(stock_fetcher)
    app.run()
```


**requirements.txt**
```
requests
pytest
```
