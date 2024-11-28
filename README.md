# Stock-Trading-Dashboard-with-Real-time-Stock-Data-and-Portfolio-Management


import tkinter as tk
from tkinter import ttk
from matplotlib.backends.backend_tkagg import FigureCanvasTkAgg
import matplotlib.pyplot as plt
import yfinance as yf

# Configuration
stock_symbols = {
    "AAPL": "Apple Inc.",
    "GOOGL": "Alphabet Inc.",
    "MSFT": "Microsoft Corporation",
    "TSLA": "Tesla Inc.",
    "AMZN": "Amazon.com Inc.",
}

portfolio = {"cash": 10000, "stocks": {symbol: 0 for symbol in stock_symbols}}

# Fetch stock price
def get_stock_price(symbol):
    try:
        stock = yf.Ticker(symbol)
        return stock.history(period="1d")["Close"].iloc[-1]
    except Exception as e:
        print(f"Error fetching price for {symbol}: {e}")
        return None

# Fetch stock data for graph
def get_stock_graph(symbol):
    try:
        stock = yf.Ticker(symbol)
        data = stock.history(period="1mo")["Close"]
        return data
    except Exception as e:
        print(f"Error fetching graph data for {symbol}: {e}")
        return None

# Show notification at the top-right
def show_notification(message):
    notification_label.config(text=message)
    notification_frame.place(x=root.winfo_width() - 320, y=10)  # Position at top-right
    root.after(3000, hide_notification)  # Hide after 3 seconds

def hide_notification():
    notification_frame.place_forget()

# Buy stocks
def buy_stock(symbol, entry):
    try:
        shares_to_buy = int(entry.get())
        price = get_stock_price(symbol)
        cost = shares_to_buy * price
        if portfolio["cash"] >= cost:
            portfolio["cash"] -= cost
            portfolio["stocks"][symbol] += shares_to_buy
            update_portfolio_display()
            show_notification(f"Bought {shares_to_buy} shares of {symbol} for ₹{cost:.2f}.")
        else:
            show_notification("Not enough cash to complete the purchase.")
    except ValueError:
        show_notification("Invalid input. Please enter a valid number of shares.")

# Sell stocks
def sell_stock(symbol, entry):
    try:
        shares_to_sell = int(entry.get())
        price = get_stock_price(symbol)
        if portfolio["stocks"][symbol] >= shares_to_sell:
            portfolio["stocks"][symbol] -= shares_to_sell
            portfolio["cash"] += shares_to_sell * price
            update_portfolio_display()
            show_notification(f"Sold {shares_to_sell} shares of {symbol} for ₹{shares_to_sell * price:.2f}.")
        else:
            show_notification("Not enough shares to complete the sale.")
    except ValueError:
        show_notification("Invalid input. Please enter a valid number of shares.")

# Add funds
def add_funds():
    try:
        amount = int(add_funds_entry.get())
        portfolio["cash"] += amount
        update_portfolio_display()
        show_notification(f"Added ₹{amount} to your balance.")
    except ValueError:
        show_notification("Invalid input. Please enter a valid amount.")

# Update portfolio display
def update_portfolio_display():
    cash_label.config(text=f"Cash Balance: ₹{portfolio['cash']:.2f}")
    stock_label.config(
        text="\n".join([f"{symbol}: {shares} shares" for symbol, shares in portfolio["stocks"].items()])
    )

# Display stock data and graphs
def display_stocks():
    row = 0
    for symbol, name in stock_symbols.items():
        price = get_stock_price(symbol)
        graph_data = stock_graph_data.get(symbol)

        if price and graph_data is not None:
            # Stock name and price
            stock_label = tk.Label(scrollable_frame, text=f"{name} ({symbol}) - ₹{price:.2f}",
                                   font=("Arial", 14, "bold"), bg="black", fg="white")
            stock_label.grid(row=row, column=0, sticky="w", padx=10, pady=5)

            # Stock graph
            fig, ax = plt.subplots(figsize=(5, 3), dpi=100)
            ax.plot(graph_data.index, graph_data.values, label=symbol, color="blue")
            ax.set_title(f"{symbol} Price History", fontsize=12, color="white")
            ax.set_xlabel("Date", fontsize=10, color="white")
            ax.set_ylabel("Price", fontsize=10, color="white")
            ax.legend()
            ax.grid(True, linestyle='--', color='gray')
            ax.tick_params(axis='both', colors='white')
            fig.tight_layout()

            # Embed the figure into the Tkinter canvas
            canvas = FigureCanvasTkAgg(fig, master=scrollable_frame)
            canvas_widget = canvas.get_tk_widget()

            # Check if canvas already exists, if yes, remove it before adding
            if hasattr(canvas_widget, 'grid_info') and canvas_widget.grid_info():
                canvas_widget.grid_forget()

            canvas_widget.grid(row=row, column=1, padx=10, pady=10)

            # Input for shares to buy/sell
            shares_entry = tk.Entry(scrollable_frame, width=8, font=("Arial", 12))
            shares_entry.grid(row=row, column=2, padx=5, pady=5)

            # Buy button
            buy_button = tk.Button(scrollable_frame, text="Buy", command=lambda s=symbol, e=shares_entry: buy_stock(s, e),
                                   bg="#4CAF50", fg="white", font=("Arial", 12), relief="solid", width=10)
            buy_button.grid(row=row, column=3, padx=5, pady=5)

            # Sell button
            sell_button = tk.Button(scrollable_frame, text="Sell", command=lambda s=symbol, e=shares_entry: sell_stock(s, e),
                                    bg="#F44336", fg="white", font=("Arial", 12), relief="solid", width=10)
            sell_button.grid(row=row, column=4, padx=5, pady=5)

        row += 1

# Scrollable frame setup
def create_scrollable_frame():
    canvas = tk.Canvas(root, bg="black")
    scrollbar = ttk.Scrollbar(root, orient="vertical", command=canvas.yview)
    scrollable_frame = tk.Frame(canvas, bg="black")

    scrollable_frame.bind(
        "<Configure>",
        lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
    )
    canvas.create_window((0, 0), window=scrollable_frame, anchor="nw")
    canvas.configure(yscrollcommand=scrollbar.set)

    canvas.pack(side="left", fill="both", expand=True)
    scrollbar.pack(side="right", fill="y")
    return scrollable_frame

# GUI Setup
root = tk.Tk()
root.title("Stock Trading Dashboard")
root.geometry("1400x800")  # Increase the window size for better visibility
root.configure(bg="black")

# Fixed Portfolio section
portfolio_frame = tk.Frame(root, bg="black")
portfolio_frame.pack(fill=tk.X, padx=20, pady=20)

cash_label = tk.Label(portfolio_frame, text=f"Cash Balance: ₹{portfolio['cash']:.2f}", font=("Arial", 16, "bold"), bg="black", fg="white")
cash_label.pack(side="left", padx=20)

stock_label = tk.Label(portfolio_frame, text="Portfolio:", font=("Arial", 16, "bold"), bg="black", fg="white")
stock_label.pack(side="left", padx=20)

add_funds_entry = tk.Entry(portfolio_frame, width=12, font=("Arial", 12))
add_funds_entry.pack(side="left", padx=10)
add_funds_button = tk.Button(portfolio_frame, text="Add Funds", command=add_funds,
                             bg="#2196F3", fg="white", font=("Arial", 12), relief="solid", width=15)
add_funds_button.pack(side="left", padx=10)

# Create the scrollable frame for stock data and graphs (below the portfolio)
scrollable_frame = create_scrollable_frame()

# Initialize stock graph data
stock_graph_data = {symbol: get_stock_graph(symbol) for symbol in stock_symbols.keys()}

# Display stocks
display_stocks()

# Create notification area at the top-right corner
notification_frame = tk.Frame(root, bg="green", height=30)
notification_label = tk.Label(notification_frame, text="", fg="white", font=("Arial", 12), bg="green")
notification_label.pack(pady=5, padx=10)

# Run GUI
root.mainloop()
