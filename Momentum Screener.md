# Stock-Screener
import numpy as np
import pandas as pd
import yfinance as yf
import matplotlib.pyplot as plt
import requests

#Get tickers from wiki
url = "https://en.wikipedia.org/wiki/List_of_S%26P_500_companies"

headers = {
    "User-Agent": "Hozumi/5.0"
}

response = requests.get(url, headers=headers)
wiki_table = pd.read_html(response.text)
df = wiki_table[0]
Tickers = df["Symbol"].tolist()
Tickers = [t.replace(".","-") for t in Tickers]
print(Tickers[:10])

from datetime import datetime, timedelta
#Setting up data frame for ticker symbols
end_date = datetime.today()
start_date = end_date - timedelta(days = 365)

Ticker_table = pd.DataFrame()

#Get the data from yfinance 
data = yf.download(Tickers, start=start_date, end=end_date, auto_adjust=False).dropna(axis=1)
Ticker_table = data['Adj Close']

#building filter
returns = (Ticker_table.iloc[-1] / Ticker_table.iloc[0]) -1

#general filter
filtered = returns[(returns > 0.05)]
#final picks
Final_picks = filtered.sort_values(ascending=False).head(10)
print(Final_picks)


chart_data = Final_picks * 100
plt.figure(figsize = (10,6))

bar = plt.barh(chart_data.index, chart_data.values, color="skyblue")
plt.title("Top 10 Momentum Stock Pick in S&P500 (Annual Return %)", fontsize=14)
plt.xlabel("Return%")
plt.ylabel("Ticker Symbol")
plt.grid(axis='x', linestyle='--', alpha=0.7)
plt.tight_layout()
plt.show()

