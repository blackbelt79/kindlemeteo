#!/usr/bin/env python3
"""Genera un'immagine meteo 758x1024 in scala di grigi per Kindle Paperwhite 2.

Uso:
    python3 kindle_meteo.py          # scarica i dati da Open-Meteo e crea weather.png
    python3 kindle_meteo.py --demo   # usa dati finti (per provare il layout)

Requisiti: Python 3 e Pillow  (pip install pillow)
"""
import json
import sys
import urllib.request
from datetime import datetime

from PIL import Image, ImageDraw, ImageFont

# ---- DA MODIFICARE -------------------------------------------------------
LAT = 45.1847          # latitudine di Pavia
LON = 9.1582           # longitudine di Pavia
CITTA = "Pavia"
# --------------------------------------------------------------------------

W, H = 758, 1024       # risoluzione del Kindle Paperwhite 2 in verticale
OUT = "weather.png"

URL = (
    "https://api.open-meteo.com/v1/forecast"
    f"?latitude={LAT}&longitude={LON}"
    "&current=temperature_2m,apparent_temperature,relative_humidity_2m,"
    "weather_code,wind_speed_10m"
    "&daily=weather_code,temperature_2m_max,temperature_2m_min,"
    "precipitation_probability_max"
    "&timezone=auto&forecast_days=5"
)

CODICI = {
    0: "Sereno", 1: "Quasi sereno", 2: "Parz. nuvoloso", 3: "Coperto",
    45: "Nebbia", 48: "Nebbia gelata",
    51: "Pioggerella", 53: "Pioggerella", 55: "Pioggerella fitta",
    56: "Pioggerella gelata", 57: "Pioggerella gelata",
    61: "Pioggia debole", 63: "Pioggia", 65: "Pioggia forte",
    66: "Pioggia gelata", 67: "Pioggia gelata",
    71: "Neve debole", 73: "Neve", 75: "Neve forte", 77: "Nevischio",
    80: "Rovesci", 81: "Rovesci", 82: "Rovesci forti",
    85: "Rovesci di neve", 86: "Rovesci di neve",
    95: "Temporale", 96: "Temporale e grandine", 99: "Temporale e grandine",
}
GIORNI = ["Lunedì", "Martedì", "Mercoledì", "Giovedì", "Venerdì", "Sabato", "Domenica"]


def font(size, bold=False):
    nomi = ["DejaVuSans-Bold.ttf", "DejaVuSans.ttf"] if bold else ["DejaVuSans.ttf"]
    for nome in nomi:
        try:
            return ImageFont.truetype(nome, size)
        except OSError:
            continue
    return ImageFont.load_default()


def descrizione(codice):
    return CODICI.get(codice, f"Codice {codice}")


def scarica():
    with urllib.request.urlopen(URL, timeout=20) as r:
        return json.load(r)


def dati_demo():
    return {
        "current": {
            "temperature_2m": 12.4, "apparent_temperature": 10.8,
            "relative_humidity_2m": 78, "weather_code": 3, "wind_speed_10m": 9.5,
        },
        "daily": {
            "time": ["2026-11-27", "2026-11-28", "2026-11-29", "2026-11-30", "2026-12-01"],
            "weather_code": [3, 61, 63, 2, 0],
            "temperature_2m_max": [13, 11, 10, 12, 14],
            "temperature_2m_min": [7, 6, 5, 4, 5],
            "precipitation_probability_max": [10, 60, 80, 20, 0],
        },
    }


def centra(draw, y, testo, f, riempimento=0):
    larghezza = draw.textlength(testo, font=f)
    draw.text(((W - larghezza) / 2, y), testo, font=f, fill=riempimento)


def disegna(dati):
    img = Image.new("L", (W, H), 255)  # scala di grigi, sfondo bianco
    d = ImageDraw.Draw(img)
    adesso = dati.get("_ora") or datetime.now()

    # Intestazione
    centra(d, 30, CITTA, font(44, True))
    centra(d, 88, f"{GIORNI[adesso.weekday()]} {adesso.day}/{adesso.month}", font(30))

    # Condizioni attuali
    c = dati["current"]
    centra(d, 150, f"{round(c['temperature_2m'])}°", font(210, True))
    centra(d, 400, descrizione(c["weather_code"]), font(46))
    dettagli = (
        f"Percepita {round(c['apparent_temperature'])}°   "
        f"Umidità {round(c['relative_humidity_2m'])}%   "
        f"Vento {round(c['wind_speed_10m'])} km/h"
    )
    centra(d, 470, dettagli, font(26))

    d.line([(40, 540), (W - 40, 540)], fill=0, width=3)

    # Previsioni dei prossimi giorni
    g = dati["daily"]
    y = 558
    for i in range(1, 5):
        data = datetime.strptime(g["time"][i], "%Y-%m-%d")
        d.text((50, y), GIORNI[data.weekday()], font=font(36, True), fill=0)
        d.text((50, y + 50), descrizione(g["weather_code"][i]), font=font(28), fill=60)
        massima = f"{round(g['temperature_2m_max'][i])}°"
        minima = f"{round(g['temperature_2m_min'][i])}°"
        d.text((470, y), massima, font=font(44, True), fill=0)
        d.text((570, y + 8), minima, font=font(36), fill=90)
        d.text((470, y + 56), f"Pioggia {g['precipitation_probability_max'][i]}%",
               font=font(24), fill=60)
        if i < 4:
            d.line([(40, y + 94), (W - 40, y + 94)], fill=190, width=2)
        y += 104

    # Piè di pagina
    centra(d, H - 38, f"Aggiornato alle {adesso:%H:%M}", font(22), 110)
    return img


def main():
    if "--demo" in sys.argv:
        dati = dati_demo()
    else:
        if LAT == 0.0 and LON == 0.0:
            sys.exit("Imposta LAT, LON e CITTA in cima allo script.")
        dati = scarica()
    img = disegna(dati)
    img.save(OUT, optimize=True)  # PNG a 8 bit, senza trasparenza
    print(f"Creato {OUT} ({W}x{H})")


if __name__ == "__main__":
    main()
