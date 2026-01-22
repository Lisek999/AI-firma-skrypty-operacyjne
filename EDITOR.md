#!/bin/bash
# SKRYPT: INSTALACJA ZALEŻNOŚCI DASHBOARD_V2
# Data: 2026-01-23
# Autor: Wojtek (asystent CEO)

echo "=== INSTALACJA ZALEŻNOŚCI DASHBOARD_V2 ==="

# 1. Sprawdźmy czy venv istnieje
cd /var/www/dashboard_v2

if [ ! -d "venv" ]; then
    echo "1. Tworzę środowisko wirtualne Python..."
    python3 -m venv venv
fi

# 2. Aktywuj venv i zainstaluj zależności
echo "2. Instaluję zależności Python..."
source venv/bin/activate

# Sprawdź czy requirements.txt istnieje
if [ -f "requirements.txt" ]; then
    pip install -r requirements.txt
else
    # Zainstaluj podstawowe pakiety
    pip install flask psutil
    # Zapisz requirements.txt
    pip freeze > requirements.txt
fi

echo "3. Zależności zainstalowane pomyślnie"
echo "   Flask: $(python3 -c 'import flask; print(flask.__version__)')"
echo "   psutil: $(python3 -c 'import psutil; print(psutil.__version__)')"

# 3. Uruchom testowy serwer
echo "4. Uruchamiam test serwera..."
echo "   Otwórz: http://57.128.247.215:5001"
echo "   Naciśnij Ctrl+C aby zatrzymać"
echo "=========================================="

python3 app.py
