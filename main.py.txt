from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from selenium.webdriver.common.by import By
import time
import requests

TELEGRAM_TOKEN = '8073820431:AAF6m55PBiI1haJPEqnHgVA1L7LDlmtDxVo'
CHAT_ID = '7307787310'
URL = 'https://www.mediaexpert.pl/smartfony-i-zegarki/smartfony/smartfon-apple-iphone-16-pro-5g-black-titanium-128gb'
CENA_PROGOWA = 5299

def get_price():
    options = Options()
    options.add_argument('--headless')
    options.add_argument('--disable-gpu')
    options.add_argument('--no-sandbox')
    options.add_argument('--disable-dev-shm-usage')

    driver = webdriver.Chrome(options=options)
    driver.get(URL)
    time.sleep(5)

    try:
        price_el = driver.find_element(By.CLASS_NAME, 'offer-price__number')
        price_text = price_el.text.strip().replace(' ', '').replace('zł', '').replace(',', '.')
        driver.quit()
        return float(price_text)
    except:
        driver.quit()
        return None

def send_telegram_message(msg):
    url = f'https://api.telegram.org/bot{TELEGRAM_TOKEN}/sendMessage'
    payload = {'chat_id': CHAT_ID, 'text': msg}
    requests.post(url, data=payload)

def check_price():
    cena = get_price()
    if cena:
        print(f"💰 Aktualna cena: {cena} zł")
        if cena < CENA_PROGOWA:
            send_telegram_message(f"📉 Cena spadła do {cena} zł!\n👉 {URL}")
    else:
        send_telegram_message("❌ Nie udało się odczytać ceny.")

check_price()
