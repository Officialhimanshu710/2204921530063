# 2204921530063
Campus hiring evaluation-frontend
from flask import Flask, jsonify
import requests
from collections import deque
import threading
import time

app = Flask(__name__)
WINDOW_SIZE = 10
number_window = deque(maxlen=WINDOW_SIZE)
window_lock = threading.Lock()
API_ENDPOINTS = {
    'p': 'http://20.244.56.144/evaluation-service/primes',
    'f': 'http://20.244.56.144/evaluation-service/fibo',
    'e': 'http://20.244.56.144/evaluation-service/even',
    'r': 'http://20.244.56.144/evaluation-service/rand'
}

@app.route('/numbers/<numberid>', methods=['GET'])
def get_numbers(numberid):
    if numberid not in API_ENDPOINTS:
        return jsonify({"error": "Invalid numberid"}), 400

   previous_window = []
    with window_lock:
        previous_window = list(number_window)

  api_url = API_ENDPOINTS[numberid]
    fetched_numbers = []

   try:
        start_time = time.time()
        response = requests.get(api_url, timeout=0.5)
        end_time = time.time()
        response.raise_for_status() 
        data = response.json()
        fetched_numbers = data.get('numbers', [])
        if (end_time - start_time) > 0.5:
            print(f"Warning: Response from {api_url} took longer than 500ms")
    except requests.exceptions.RequestException as e:
        return jsonify({"error": f"Error fetching from external API: {e}"}), 500

   unique_new_numbers = [num for num in fetched_numbers if num not in number_window]

   with window_lock:
        for num in unique_new_numbers:
            number_window.append(num)
        current_window = list(number_window)
        average = sum(current_window) / len(current_window) if current_window else 0

   return jsonify({
        "windowPrevState": previous_window,
        "windowCurrState": current_window,
        "numbers": fetched_numbers,
        "avg": round(average, 2)
    })
    
