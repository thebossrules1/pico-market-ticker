# 📈 Live Market Sentiment Desk Indicator (Pico WH)

I wanted a physical desk gadget that blends my interests in tech and finance. This project uses a **Raspberry Pi Pico WH** to connect to home Wi-Fi, pull live crypto market data via a public API, and light up my desk **green** or **red** based on daily market performance. 

No more checking my phone every 10 minutes—now my desk just tells me how the market is doing! 

---

## 💡 How It Works
1. **The Handshake:** On boot, the Pico connects locally to my Wi-Fi using MicroPython.
2. **The Fetch:** Every 5 minutes, it sends an HTTP GET request to the public **Binance API** (`BTCUSDT`) to grab the latest 24-hour price change.
3. **The Logic:** It parses the JSON payload, looks at the percentage change variable, and updates the physical hardware state.
4. **The Output:** 
   * 🟢 **Green LED (Pin 14)** turns on if the market is up (`>= 0%`).
   * 🔴 **Red LED (Pin 15)** turns on if the market is down (`< 0%`).

---

## 🛠️ Tech Stack
* **Microcontroller:** Raspberry Pi Pico WH (RP2040 with built-in Wi-Fi)
* **Language:** MicroPython
* **Networking:** TCP/IP, Wi-Fi stack, HTTP REST requests (`urequests`)
* **Data Handling:** JSON parsing, real-time API integration
* **Hardware:** Breadboard, standard LEDs, 220Ω current-limiting resistors, jumper wires

---

## 🔌 Wiring Quick Guide
* **Green LED:** Connected to **GPIO Pin 14** via a 220Ω resistor to **GND**.
* **Red LED:** Connected to **GPIO Pin 15** via a 220Ω resistor to **GND**. 


## 🚀 Quick Setup
1. Flash MicroPython onto your Raspberry Pi Pico W/WH using Thonny.
2. Open `main.py` and drop in your Wi-Fi credentials (`WIFI_SSID` and `WIFI_PASSWORD`).
3. Save the script to the Pico as `main.py` so it runs automatically on boot!

## 💻 Code Snippet (`main.py`)
import network
import time
import urequests
from machine import Pin

# 1. Setup LED Pins (Pin 14 = Green, Pin 15 = Red)
green_led = Pin(14, Pin.OUT)
red_led = Pin(15, Pin.OUT)

# Startup test: flash both lights twice to show they work
for _ in range(2):
    green_led.on()
    red_led.on()
    time.sleep(0.3)
    green_led.off()
    red_led.off()
    time.sleep(0.3)

# 2. Connect to Wi-Fi
WIFI_SSID = "WIFI_SSID"
WIFI_PASSWORD = "WIFI_PASSWORD"

wlan = network.WLAN(network.STA_IF)
wlan.active(True)
wlan.connect(WIFI_SSID, WIFI_PASSWORD)

print("Connecting to Wi-Fi...", end="")
while not wlan.isconnected():
    print(".", end="")
    time.sleep(1)
print(" Connected!\n")

API_URL = "https://api.binance.com/api/v3/ticker/24hr?symbol=BTCUSDT"

# 3. Main Loop - Check market continuously every 5 minutes
while True:
    try:
        print("Fetching market data...")
        response = urequests.get(API_URL)
        data = response.json()
        response.close() # Always close the connection to free up memory
        
        btc_price = float(data["lastPrice"])
        btc_change = float(data["priceChangePercent"])
        
        print(f"--- Market Update ---")
        print(f"Bitcoin Price: ${btc_price:,.2f} USD")
        print(f"24h Change: {btc_change:.2f}%")
        
        if btc_change >= 0:
            print("Market Status: GREEN (Up for the day) 🟢\n")
            green_led.on()
            red_led.off()
        else:
            print("Market Status: RED (Down for the day) 🔴\n")
            green_led.off()
            red_led.on()

    except Exception as e:
        print("Failed to fetch data:", e)
        # Turn both on if an error occurs (like a brief Wi-Fi drop)
        green_led.on()
        red_led.on()
        
    # Wait 5 minutes (300 seconds) before checking again
    time.sleep(300)
