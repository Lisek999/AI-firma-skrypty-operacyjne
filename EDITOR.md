#!/bin/bash
# SKRYPT: PODGLĄD BACKUPU GRUDNIOWEGO W PRZEGLĄDARCE
# Data: 2026-01-23
# Autor: Wojtek (asystent CEO)

echo "=== PODGLĄD BACKUPU GRUDNIOWEGO ==="

# 1. BACKUP - LINIA OBOWIĄZKOWA (§2)
TIMESTAMP=$(date +%s)
echo "1. Tworzę backup konfiguracji portów..."
sudo cp /etc/supervisor/conf.d/dashboard-api.conf /etc/supervisor/conf.d/dashboard-api.conf.backup_${TIMESTAMP} 2>/dev/null || echo "Backup konfiguracji pominięty (brak pliku)"

# 2. Przejdź do katalogu backupu
cd /home/ubuntu/old_dashboard_backup_20260118_153656/dashboard_backup

# 3. Sprawdź, czy są zależności Pythona
if [ ! -d "venv" ]; then
    echo "2. Tworzę środowisko wirtualne Pythona..."
    python3 -m venv venv
    source venv/bin/activate
    pip install flask psutil
else
    echo "2. Używam istniejącego venv..."
    source venv/bin/activate
fi

# 4. Uruchom Flask na porcie 5002
echo "3. Uruchamiam podgląd backupu na http://$(curl -s ifconfig.me):5002"
echo "   Naciśnij Ctrl+C aby zatrzymać"
echo "=========================================="

python3 -c "
from flask import Flask, render_template
import os
app = Flask(__name__)

@app.route('/')
def index():
    try:
        return render_template('index.html')
    except:
        return '<h1>Strona główna backupu</h1><p>Szablon index.html może nie istnieć.</p><p><a href=\"/archive\">Archiwum</a></p>'

@app.route('/archive')
def archive():
    try:
        return render_template('archive.html')
    except:
        return '<h1>Archiwum (backup)</h1><p>Szablon archive.html może nie istnieć.</p>'

@app.route('/api/health')
def health():
    return {'status': 'backup-preview', 'port': 5002}

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5002, debug=False)
"
