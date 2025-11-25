# Task-download-data-in-csv-file-api-code-
This project is a Flask-based REST API that downloads stock market OHLCV data using yfinance, cleans it, resamples it into any desired timeframe (1m → 5m, 1h → 1d, etc.), and returns the output in clean JSON format.

# Features

✔ Download stock OHLCV data using YFinance
Supports NSE, BSE, NASDAQ, NYSE symbols
Example: SBIN.NS, RELIANCE.NS, AAPL, TSLA
✔ Automatic Column Cleaning
YFinance sometimes returns MultiIndex columns. This API automatically fixes that.
✔ Auto-detects datetime column
Works even if yfinance changes its internal format.
✔ Auto-detects OHLCV columns
Finds correct open, high, low, close, volume even if names change.
✔ Resample to any timeframe
Convert data using Pandas:
Input Interval	Output (Pandas)	Description
1m	1T	1 Minute Candle
5m	5T	5 Minute Candle
1h	1H	Hourly Candle
1d	1D	Daily Candle
1wk	1W	Weekly Candle
1mo	1M	Monthly Candle
✔ JSON-Friendly Output
Datetime is converted to:
"date": "2025-01-01",
"time": "09:30:00"

# How the Code Works (Step-by-Step)
✅ STEP 1 — Flask App Created
app = Flask(__name__)
You start a Flask server so users can send POST/GET requests.

✅ STEP 2 — Function: download_and_resample()
This is your main logic function.

2.1 → Download Data
df = yf.download(symbol, start=start_date, end=end_date, interval=interval, progress=False)
✔ Downloads intraday or daily OHLCV data
✔ If no data → returns error

2.2 → Reset Index
df.reset_index(inplace=True)
Because yfinance returns datetime as index.

2.3 → Flatten MultiIndex
Sometimes yfinance gives columns like:
('Open', 'SBIN.NS')
('Close', 'SBIN.NS')
You convert them into simple lowercase:
open
close
high
low
volume

2.4 → Detect DateTime Column Automatically
datetime_col = next((c for c in df.columns if c.startswith("date") or c.startswith("datetime")), None)
✔ Auto-detects which column is datetime
✔ Prevents errors when yfinance changes its format

2.5 → Convert to Real Datetime Type
df['datetime'] = pd.to_datetime(df[datetime_col])

2.6 → Detect OHLCV Columns
col_map = {
    'open': next((c for c in df.columns if c.startswith("open")), None),
    'high': ...
}
✔ Automatically finds open/high/low/close/volume even if names change
✔ Prevents "column not found" errors

2.7 → Create Clean DataFrame
df = df[['datetime','open','high','low','close','volume']]
df.set_index('datetime', inplace=True)

2.8 → Convert Interval to Pandas Format
Example:
yfinance	pandas
1m	1T
5m	5T
1h	1H
1d	1D
Your code:
interval_map = {
 '1m': '1T', '5m': '5T', '1h': '1H', '1d': '1D'
}

2.9 → RESAMPLE DATA
Example:
If user selects interval = "5m" →
1-minute data becomes 5-minute candles
resampled = df.resample('5T').agg({
    'open': 'first',
    'high': 'max',
    'low': 'min',
    'close': 'last',
    'volume': 'sum'
})

2.10 → FIX JSON SERIALIZATION
Python datetime cannot be returned in JSON.
So convert date & time to string:
resampled['date'] = resampled['datetime'].dt.date.astype(str)
resampled['time'] = resampled['datetime'].dt.time.astype(str)
