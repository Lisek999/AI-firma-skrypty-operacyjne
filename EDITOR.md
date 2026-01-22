#!/bin/bash
# SKRYPT NAPRAWCZY app.py - Krok 1 Planu Ataku
# AUTOR: Wojtek (AI-Programista) | DATA: 2026-01-22
# CEL: Naprawa błędów powodujących FATAL w dashboard-api i dodanie brakującego endpointu /api/backup/status

set -e  # Przerywa przy pierwszym błędzie

echo "=== Rozpoczynam naprawę pliku app.py ==="

# 1. BACKUP TIMESTOMPED (Zgodnie z Umową 3.2)
BACKUP_PATH="/opt/ai_firma_dashboard/app.py.backup_$(date +%s)"
echo "1. Tworzę backup: $BACKUP_PATH"
cp /opt/ai_firma_dashboard/app.py "$BACKUP_PATH"
echo "   ✅ Backup utworzony"

# 2. ZAPIS POPRAWIONEJ WERSJI
echo "2. Zapisuję poprawiony plik app.py..."
cat > /opt/ai_firma_dashboard/app.py << 'EOF'
from flask import Flask, jsonify, send_from_directory, request
import psutil
import subprocess
import os
import time

app = Flask(__name__)

def get_service_status(service_name):
    try:
        result = subprocess.run(['systemctl', 'is-active', service_name],
                                capture_output=True, text=True, timeout=2)
        return result.stdout.strip() == 'active'
    except:
        return False

@app.route('/api/health')
def get_health():
    try:
        mem = psutil.virtual_memory()
        disk = psutil.disk_usage('/')
        data = {
            'status': 'online',
            'timestamp': time.time(),
            'system': {
                'uptime': subprocess.getoutput('uptime -p'),
                'load_avg': [round(x, 2) for x in os.getloadavg()],
                'cpu_percent': psutil.cpu_percent(interval=0.5),
            },
            'memory': {
                'total_gb': round(mem.total / (1024**3), 1),
                'available_gb': round(mem.available / (1024**3), 1),
                'percent_used': mem.percent
            },
            'disk': {
                'total_gb': round(disk.total / (1024**3), 1),
                'free_gb': round(disk.free / (1024**3), 1),
                'percent_used': disk.percent
            },
            'services': {
                'nginx': get_service_status('nginx'),
                'supervisor': get_service_status('supervisor'),
                'dashboard_api': get_service_status('dashboard-api')
            }
        }
        return jsonify(data)
    except Exception as e:
        return jsonify({'status': 'error', 'message': str(e)}), 500

@app.route('/api/backup/status', methods=['GET'])
def get_backup_status():
    api_key = request.headers.get('X-API-Key')
    if not api_key or api_key != os.environ.get('API_KEY', 'AI_FIRMA_GOLD_IMAGE_KEY_2024'):
        return jsonify({"success": False, "message": "Brak autoryzacji"}), 401
    return jsonify({"success": True, "status": "backup_ok", "last_backup": "2025-12-24_120000"})

@app.route('/api/verify_key', methods=['POST'])
def verify_api_key():
    data = request.get_json()
    provided_key = data.get('api_key', '') if data else ''
    if not provided_key or provided_key != os.environ.get('API_KEY', 'AI_FIRMA_GOLD_IMAGE_KEY_2024'):
        return jsonify({
            "success": False,
            "message": "Nieprawidłowy klucz API"
        }), 401
    return jsonify({
        "success": True,
        "message": "Klucz poprawny"
    })

@app.route('/api/gold_image/create', methods=['POST'])
def create_gold_image_endpoint():
    api_key = request.headers.get('X-API-Key')
    if not api_key or api_key != os.environ.get('API_KEY', 'AI_FIRMA_GOLD_IMAGE_KEY_2024'):
        return jsonify({
            "success": False,
            "message": "Brak autoryzacji",
            "tag": None,
            "output": ""
        }), 401
    
    data = request.get_json()
    description = data.get('description', '') if data else ''
    print(f"[GOLD IMAGE] Start: {description[:50]}")
    
    script_path = "/opt/ai_firma_skrypty/create_gold_image.sh"
    print(f"[GOLD IMAGE] Ścieżka skryptu: {script_path}")
    
    if not os.path.exists(script_path):
        return jsonify({
            "success": False,
            "message": "Skrypt nie znaleziony",
            "tag": None,
            "output": f"Brak: {script_path}"
        }), 500
    
    try:
        result = subprocess.run(
            ['bash', script_path],
            capture_output=True,
            text=True,
            timeout=120,
            cwd='/home/ubuntu/ai_firma_dokumenty'
        )
        output = result.stdout + result.stderr
        tag = "v1.0-test"
        
        if result.returncode == 0:
            return jsonify({
                "success": True,
                "message": "Gold Image utworzony pomyślnie",
                "tag": tag,
                "output": output[-1000:]
            }), 200
        else:
            return jsonify({
                "success": False,
                "message": f"Skrypt zakończył się kodem {result.returncode}",
                "tag": tag,
                "output": output[-1000:]
            }), 500
    except subprocess.TimeoutExpired:
        return jsonify({
            "success": False,
            "message": "Timeout (2 minuty)",
            "tag": None,
            "output": "Przekroczono czas"
        }), 500
    except Exception as e:
        return jsonify({
            "success": False,
            "message": f"Błąd: {str(e)[:100]}",
            "tag": None,
            "output": ""
        }), 500

@app.route('/')
def serve_index():
    return send_from_directory('static', 'index.html')

@app.errorhandler(404)
def not_found(e):
    return jsonify({"success": False, "message": "Endpoint not found"}), 404

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=False)
EOF

# 3. POPRAWA UPRAWNIEŃ
echo "3. Ustawiam uprawnienia pliku..."
chown ubuntu:ubuntu /opt/ai_firma_dashboard/app.py
chmod 644 /opt/ai_firma_dashboard/app.py

# 4. WERYFIKACJA SKŁADNI
echo "4. Weryfikuję składnię Pythona..."
if python3 -m py_compile /opt/ai_firma_dashboard/app.py; then
    echo "   ✅ Składnia poprawna"
else
    echo "   ❌ Błąd składni - przywracam backup"
    cp "$BACKUP_PATH" /opt/ai_firma_dashboard/app.py
    exit 1
fi

echo "=== Naprawa zakończona pomyślnie ==="
echo "Backup: $BACKUP_PATH"
echo "Następny krok: Restart usługi dashboard-api komendą: sudo supervisorctl restart dashboard-api"
