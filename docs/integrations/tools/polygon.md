---
custom_edit_url: https://github.com/langchain-ai/langchain/edit/master/docs/docs/integrations/tools/polygon.ipynb
---
# Polygon Stock Market API Tools

>[Polygon](https://polygon.io/) The Polygon.io Stocks API provides REST endpoints that let you query the latest market data from all US stock exchanges.

This notebook uses tools to get stock market data like the latest quote and news for a ticker from Polygon.


```python
import getpass
import os

os.environ["POLYGON_API_KEY"] = getpass.getpass()
```
```output
 ········
```

```python
from langchain_community.tools.polygon.aggregates import PolygonAggregates
from langchain_community.tools.polygon.financials import PolygonFinancials
from langchain_community.tools.polygon.last_quote import PolygonLastQuote
from langchain_community.tools.polygon.ticker_news import PolygonTickerNews
from langchain_community.utilities.polygon import PolygonAPIWrapper
```


```python
api_wrapper = PolygonAPIWrapper()
ticker = "AAPL"
```

### Get latest quote for ticker


```python
# Get the last quote for ticker
last_quote_tool = PolygonLastQuote(api_wrapper=api_wrapper)
last_quote = last_quote_tool.run(ticker)
print(f"Tool output: {last_quote}")
```
```output
Tool output: {"P": 170.5, "S": 2, "T": "AAPL", "X": 11, "i": [604], "p": 170.48, "q": 106666224, "s": 1, "t": 1709945992614283138, "x": 12, "y": 1709945992614268948, "z": 3}
```

```python
import json

# Convert the last quote response to JSON
last_quote = last_quote_tool.run(ticker)
last_quote_json = json.loads(last_quote)
```


```python
# Print the latest price for ticker
latest_price = last_quote_json["p"]
print(f"Latest price for {ticker} is ${latest_price}")
```
```output
Latest price for AAPL is $170.48
```
### Get aggregates (historical prices) for ticker


```python
from langchain_community.tools.polygon.aggregates import PolygonAggregatesSchema

# Define param
params = PolygonAggregatesSchema(
    ticker=ticker,
    timespan="day",
    timespan_multiplier=1,
    from_date="2024-03-01",
    to_date="2024-03-08",
)
# Get aggregates for ticker
aggregates_tool = PolygonAggregates(api_wrapper=api_wrapper)
aggregates = aggregates_tool.run(tool_input=params.dict())
aggregates_json = json.loads(aggregates)
```


```python
print(f"Total aggregates: {len(aggregates_json)}")
print(f"Aggregates: {aggregates_json}")
```
```output
Total aggregates: 6
Aggregates: [{'v': 73450582.0, 'vw': 179.0322, 'o': 179.55, 'c': 179.66, 'h': 180.53, 'l': 177.38, 't': 1709269200000, 'n': 911077}, {'v': 81505451.0, 'vw': 174.8938, 'o': 176.15, 'c': 175.1, 'h': 176.9, 'l': 173.79, 't': 1709528400000, 'n': 1167166}, {'v': 94702355.0, 'vw': 170.3234, 'o': 170.76, 'c': 170.12, 'h': 172.04, 'l': 169.62, 't': 1709614800000, 'n': 1108820}, {'v': 68568907.0, 'vw': 169.5506, 'o': 171.06, 'c': 169.12, 'h': 171.24, 'l': 168.68, 't': 1709701200000, 'n': 896297}, {'v': 71763761.0, 'vw': 169.3619, 'o': 169.15, 'c': 169, 'h': 170.73, 'l': 168.49, 't': 1709787600000, 'n': 825405}, {'v': 76267041.0, 'vw': 171.5322, 'o': 169, 'c': 170.73, 'h': 173.7, 'l': 168.94, 't': 1709874000000, 'n': 925213}]
```
### Get latest news for ticker


```python
ticker_news_tool = PolygonTickerNews(api_wrapper=api_wrapper)
ticker_news = ticker_news_tool.run(ticker)
```


```python
# Convert the news response to JSON array
ticker_news_json = json.loads(ticker_news)
print(f"Total news items: {len(ticker_news_json)}")
```
```output
Total news items: 10
```

```python
# Inspect the first news item
news_item = ticker_news_json[0]
print(f"Title: {news_item['title']}")
print(f"Description: {news_item['description']}")
print(f"Publisher: {news_item['publisher']['name']}")
print(f"URL: {news_item['article_url']}")
```
```output
Title: An AI surprise could fuel a 20% rally for the S&P 500 in 2024, says UBS
Description: If Gen AI causes a big productivity boost, stocks could see an unexpected rally this year, say UBS strategists.
Publisher: MarketWatch
URL: https://www.marketwatch.com/story/an-ai-surprise-could-fuel-a-20-rally-for-the-s-p-500-in-2024-says-ubs-1044d716
```
### Get financials for ticker


```python
financials_tool = PolygonFinancials(api_wrapper=api_wrapper)
financials = financials_tool.run(ticker)
```


```python
# Convert the financials response to JSON
financials_json = json.loads(financials)
print(f"Total reporting periods: {len(financials_json)}")
```
```output
Total reporting periods: 10
```

```python
# Print the latest reporting period's financials metadata
financial_data = financials_json[0]
print(f"Company name: {financial_data['company_name']}")
print(f"CIK: {financial_data['cik']}")
print(f"Fiscal period: {financial_data['fiscal_period']}")
print(f"End date: {financial_data['end_date']}")
print(f"Start date: {financial_data['start_date']}")
```
```output
Company name: APPLE INC
CIK: 0000320193
Fiscal period: TTM
End date: 2023-12-30
Start date: 2022-12-31
```

```python
# Print the latest reporting period's income statement
print(f"Income statement: {financial_data['financials']['income_statement']}")
```
```output
Income statement: {'diluted_earnings_per_share': {'value': 6.42, 'unit': 'USD / shares', 'label': 'Diluted Earnings Per Share', 'order': 4300}, 'costs_and_expenses': {'value': 267270000000.0, 'unit': 'USD', 'label': 'Costs And Expenses', 'order': 600}, 'net_income_loss_attributable_to_noncontrolling_interest': {'value': 0, 'unit': 'USD', 'label': 'Net Income/Loss Attributable To Noncontrolling Interest', 'order': 3300}, 'net_income_loss_attributable_to_parent': {'value': 100913000000.0, 'unit': 'USD', 'label': 'Net Income/Loss Attributable To Parent', 'order': 3500}, 'income_tax_expense_benefit': {'value': 17523000000.0, 'unit': 'USD', 'label': 'Income Tax Expense/Benefit', 'order': 2200}, 'income_loss_from_continuing_operations_before_tax': {'value': 118436000000.0, 'unit': 'USD', 'label': 'Income/Loss From Continuing Operations Before Tax', 'order': 1500}, 'operating_expenses': {'value': 55013000000.0, 'unit': 'USD', 'label': 'Operating Expenses', 'order': 1000}, 'benefits_costs_expenses': {'value': 267270000000.0, 'unit': 'USD', 'label': 'Benefits Costs and Expenses', 'order': 200}, 'diluted_average_shares': {'value': 47151996000.0, 'unit': 'shares', 'label': 'Diluted Average Shares', 'order': 4500}, 'cost_of_revenue': {'value': 212035000000.0, 'unit': 'USD', 'label': 'Cost Of Revenue', 'order': 300}, 'operating_income_loss': {'value': 118658000000.0, 'unit': 'USD', 'label': 'Operating Income/Loss', 'order': 1100}, 'net_income_loss_available_to_common_stockholders_basic': {'value': 100913000000.0, 'unit': 'USD', 'label': 'Net Income/Loss Available To Common Stockholders, Basic', 'order': 3700}, 'preferred_stock_dividends_and_other_adjustments': {'value': 0, 'unit': 'USD', 'label': 'Preferred Stock Dividends And Other Adjustments', 'order': 3900}, 'research_and_development': {'value': 29902000000.0, 'unit': 'USD', 'label': 'Research and Development', 'order': 1030}, 'revenues': {'value': 385706000000.0, 'unit': 'USD', 'label': 'Revenues', 'order': 100}, 'participating_securities_distributed_and_undistributed_earnings_loss_basic': {'value': 0, 'unit': 'USD', 'label': 'Participating Securities, Distributed And Undistributed Earnings/Loss, Basic', 'order': 3800}, 'selling_general_and_administrative_expenses': {'value': 25111000000.0, 'unit': 'USD', 'label': 'Selling, General, and Administrative Expenses', 'order': 1010}, 'nonoperating_income_loss': {'value': -222000000.0, 'unit': 'USD', 'label': 'Nonoperating Income/Loss', 'order': 900}, 'income_loss_from_continuing_operations_after_tax': {'value': 100913000000.0, 'unit': 'USD', 'label': 'Income/Loss From Continuing Operations After Tax', 'order': 1400}, 'basic_earnings_per_share': {'value': 6.46, 'unit': 'USD / shares', 'label': 'Basic Earnings Per Share', 'order': 4200}, 'basic_average_shares': {'value': 46946265000.0, 'unit': 'shares', 'label': 'Basic Average Shares', 'order': 4400}, 'gross_profit': {'value': 173671000000.0, 'unit': 'USD', 'label': 'Gross Profit', 'order': 800}, 'net_income_loss': {'value': 100913000000.0, 'unit': 'USD', 'label': 'Net Income/Loss', 'order': 3200}}
```

```python
# Print the latest reporting period's balance sheet
print(f"Balance sheet: {financial_data['financials']['balance_sheet']}")
```
```output
Balance sheet: {'equity_attributable_to_noncontrolling_interest': {'value': 0, 'unit': 'USD', 'label': 'Equity Attributable To Noncontrolling Interest', 'order': 1500}, 'other_noncurrent_liabilities': {'value': 39441000000.0, 'unit': 'USD', 'label': 'Other Non-current Liabilities', 'order': 820}, 'equity': {'value': 74100000000.0, 'unit': 'USD', 'label': 'Equity', 'order': 1400}, 'liabilities': {'value': 279414000000.0, 'unit': 'USD', 'label': 'Liabilities', 'order': 600}, 'noncurrent_assets': {'value': 209822000000.0, 'unit': 'USD', 'label': 'Noncurrent Assets', 'order': 300}, 'equity_attributable_to_parent': {'value': 74100000000.0, 'unit': 'USD', 'label': 'Equity Attributable To Parent', 'order': 1600}, 'liabilities_and_equity': {'value': 353514000000.0, 'unit': 'USD', 'label': 'Liabilities And Equity', 'order': 1900}, 'other_current_liabilities': {'value': 75827000000.0, 'unit': 'USD', 'label': 'Other Current Liabilities', 'order': 740}, 'inventory': {'value': 6511000000.0, 'unit': 'USD', 'label': 'Inventory', 'order': 230}, 'other_noncurrent_assets': {'value': 166156000000.0, 'unit': 'USD', 'label': 'Other Non-current Assets', 'order': 350}, 'other_current_assets': {'value': 137181000000.0, 'unit': 'USD', 'label': 'Other Current Assets', 'order': 250}, 'current_liabilities': {'value': 133973000000.0, 'unit': 'USD', 'label': 'Current Liabilities', 'order': 700}, 'noncurrent_liabilities': {'value': 145441000000.0, 'unit': 'USD', 'label': 'Noncurrent Liabilities', 'order': 800}, 'fixed_assets': {'value': 43666000000.0, 'unit': 'USD', 'label': 'Fixed Assets', 'order': 320}, 'long_term_debt': {'value': 106000000000.0, 'unit': 'USD', 'label': 'Long-term Debt', 'order': 810}, 'current_assets': {'value': 143692000000.0, 'unit': 'USD', 'label': 'Current Assets', 'order': 200}, 'assets': {'value': 353514000000.0, 'unit': 'USD', 'label': 'Assets', 'order': 100}, 'accounts_payable': {'value': 58146000000.0, 'unit': 'USD', 'label': 'Accounts Payable', 'order': 710}}
```

```python
# Print the latest reporting period's cash flow statement
print(f"Cash flow statement: {financial_data['financials']['cash_flow_statement']}")
```
```output
Cash flow statement: {'net_cash_flow_continuing': {'value': 20000000000.0, 'unit': 'USD', 'label': 'Net Cash Flow, Continuing', 'order': 1200}, 'net_cash_flow_from_investing_activities_continuing': {'value': 7077000000.0, 'unit': 'USD', 'label': 'Net Cash Flow From Investing Activities, Continuing', 'order': 500}, 'net_cash_flow_from_investing_activities': {'value': 7077000000.0, 'unit': 'USD', 'label': 'Net Cash Flow From Investing Activities', 'order': 400}, 'net_cash_flow_from_financing_activities_continuing': {'value': -103510000000.0, 'unit': 'USD', 'label': 'Net Cash Flow From Financing Activities, Continuing', 'order': 800}, 'net_cash_flow_from_operating_activities': {'value': 116433000000.0, 'unit': 'USD', 'label': 'Net Cash Flow From Operating Activities', 'order': 100}, 'net_cash_flow_from_financing_activities': {'value': -103510000000.0, 'unit': 'USD', 'label': 'Net Cash Flow From Financing Activities', 'order': 700}, 'net_cash_flow_from_operating_activities_continuing': {'value': 116433000000.0, 'unit': 'USD', 'label': 'Net Cash Flow From Operating Activities, Continuing', 'order': 200}, 'net_cash_flow': {'value': 20000000000.0, 'unit': 'USD', 'label': 'Net Cash Flow', 'order': 1100}}
```

## Related

- Tool [conceptual guide](/docs/concepts/#tools)
- Tool [how-to guides](/docs/how_to/#tools)
