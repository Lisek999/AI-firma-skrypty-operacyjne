#!/bin/bash
# SKRYPT: AKTUALIZACJA APP.PY I URUCHOMIENIE DASHBOARD_V2
# Data: 2026-01-23
# Autor: Wojtek (asystent CEO)

echo "=== AKTUALIZACJA APP.PY DASHBOARD_V2 ==="

# 1. BACKUP - LINIA OBOWIĄZKOWA (§2)
TIMESTAMP=$(date +%s)
echo "1. Tworzę backup app.py..."
cp /var/www/dashboard_v2/app.py /var/www/dashboard_v2/app.py.backup_${TIMESTAMP}
echo "   Backup utworzony: app.py.backup_${TIMESTAMP}"

# 2. Aktualizacja app.py z podstawowymi endpointami
echo "2. Aktualizuję app.py z nowymi endpointami..."

cat > /tmp/new_app.py << 'EOF'
from flask import Flask, jsonify, render_template, request, send_from_directory
import psutil
import subprocess
import os
import time
import json
from functools import wraps

app = Flask(__name__)

# ===== KONFIGURACJA =====
API_KEYS = ["AI_FIRMA_SECURE_KEY_2025", "TEST_KEY_123"]  # Tymczasowo - później z .env
PORT = 5001

# ===== BEZPIECZEŃSTWO =====
def require_api_key(f):
    @wraps(f)
    def decorated(*args, **kwargs):
        api_key = request.headers.get('X-API-Key')
        if api_key and api_key in API_KEYS:
            return f(*args, **kwargs)
        return jsonify({"error": "Nieprawidłowy lub brakujący klucz API"}), 401
    return decorated

# ===== POMOCNICZE FUNKCJE =====
def get_service_status(service_name):
    try:
        result = subprocess.run(['systemctl', 'is-active', service_name],
                                capture_output=True, text=True, timeout=2)
        return result.stdout.strip() == 'active'
    except:
        return False

def get_system_status():
    try:
        mem = psutil.virtual_memory()
        disk = psutil.disk_usage('/')
        return {
            'status': 'online',
            'timestamp': time.time(),
            'system': {
                'uptime': subprocess.getoutput('uptime -p'),
                'load_avg': [round(x, 2) for x in os.getloadavg()],
                'cpu_percent': round(psutil.cpu_percent(interval=0.5), 1),
            },
            'memory': {
                'total_gb': round(mem.total / (1024**3), 1),
                'available_gb': round(mem.available / (1024**3), 1),
                'percent_used': round(mem.percent, 1)
            },
            'disk': {
                'total_gb': round(disk.total / (1024**3), 1),
                'free_gb': round(disk.free / (1024**3), 1),
                'percent_used': round(disk.percent, 1)
            },
            'services': {
                'nginx': get_service_status('nginx'),
                'supervisor': get_service_status('supervisor'),
                'dashboard_api': True  # To my właśnie działamy
            }
        }
    except Exception as e:
        return {'status': 'error', 'message': str(e)}

# ===== ENDPOINTY WIDOKÓW =====
@app.route('/')
def index():
    status = get_system_status()
    return render_template('index.html', 
                         status='online' if status['status'] == 'online' else 'error',
                         status_message='Systemy sprawne' if status['status'] == 'online' else 'Błąd systemu')

@app.route('/archive')
def archive():
    return render_template('archive.html',
                         status='online',
                         status_message='Moduł Archiwum')

@app.route('/pulse')
def pulse():
    return render_template('pulse.html',
                         status='online',
                         status_message='Puls Systemu')

@app.route('/terminal')
def terminal():
    return render_template('terminal.html',
                         status='online',
                         status_message='Terminal')

@app.route('/explorer')
def explorer():
    return render_template('explorer.html',
                         status='online',
                         status_message='Eksplorator')

@app.route('/config')
def config():
    return render_template('config.html',
                         status='online',
                         status_message='Konfiguracja')

# ===== ENDPOINTY API =====
@app.route('/api/health')
def api_health():
    return jsonify(get_system_status())

@app.route('/api/backup/status')
@require_api_key
def backup_status():
    """Status backupów aplikacji - czyta z istniejącego systemu"""
    try:
        # Próba odczytu z logu backupów aplikacji
        log_path = "/var/log/backup_status.json"
        if os.path.exists(log_path):
            with open(log_path, 'r') as f:
                data = json.load(f)
            return jsonify(data)
        else:
            return jsonify({
                "status": "info",
                "message": "System backupów aplikacji nie zgłasza statusu",
                "last_check": time.time()
            })
    except Exception as e:
        return jsonify({"error": str(e)}), 500

@app.route('/api/dashboard/backup/list')
@require_api_key
def dashboard_backup_list():
    """Lista backupów kodu dashboardu"""
    backup_dir = "/backups/dashboard_v2"
    backups = []
    if os.path.exists(backup_dir):
        for f in sorted(os.listdir(backup_dir)):
            if f.startswith("dashboard_backup_") and f.endswith(".tar.gz"):
                path = os.path.join(backup_dir, f)
                stat = os.stat(path)
                backups.append({
                    "name": f,
                    "size_mb": round(stat.st_size / (1024*1024), 2),
                    "modified": stat.st_mtime,
                    "path": path
                })
    return jsonify({"backups": backups})

# ===== STATIC FILES =====
@app.route('/static/<path:filename>')
def static_files(filename):
    return send_from_directory('static', filename)

# ===== START =====
if __name__ == '__main__':
    print(f"Uruchamianie dashboard v2 na porcie {PORT}")
    app.run(host='0.0.0.0', port=PORT, debug=False)
EOF

# 3. Zastępujemy stary app.py nowym
cp /tmp/new_app.py /var/www/dashboard_v2/app.py
echo "   App.py zaktualizowany z endpointami dla 6 modułów"

# 4. Uruchamiamy tymczasowy serwer do testu
echo "3. Uruchamiam dashboard_v2 na porcie 5001..."
echo "   Otwórz w przeglądarce: http://57.128.247.215:5001"
echo "   Naciśnij Ctrl+C aby zatrzymać"
echo "=========================================="

cd /var/www/dashboard_v2
python3 app.py
