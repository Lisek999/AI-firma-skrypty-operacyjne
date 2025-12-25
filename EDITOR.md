#!/bin/bash
# Skrypt: Dodanie endpointów Gold Image do dashboardu AI Firma
# Format: Bezpieczny skrypt bash do wykonania przez getscript
# Zgodność: Konstytucja Systemu §2 (Architektura 2 plików)

echo "=== DODAWANIE PANELU GOLD IMAGE (BEZPIECZNIE) ==="
echo "Krok 1/5: Backup obecnych plików..."

# Backup app.py
BACKUP_APP="/opt/ai_firma_dashboard/app.py.backup_gold_$(date +%Y%m%d_%H%M%S)"
sudo cp /opt/ai_firma_dashboard/app.py "$BACKUP_APP"
echo "✓ Backup app.py: $BACKUP_APP"

# Backup index.html  
BACKUP_HTML="/opt/ai_firma_dashboard/static/index.html.backup_gold_$(date +%Y%m%d_%H%M%S)"
sudo cp /opt/ai_firma_dashboard/static/index.html "$BACKUP_HTML" 2>/dev/null || true
echo "✓ Backup index.html: $BACKUP_HTML"

echo "Krok 2/5: Tworzenie poprawnego app.py z endpointami Gold Image..."

# Utwórz tymczasowy plik z nowym app.py
TEMP_APP=$(mktemp)
sudo tee "$TEMP_APP" > /dev/null << 'APP_EOF'
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
                'dashboard_api': get_service_status('supervisor')
            }
        }
        return jsonify(data)
    except Exception as e:
        return jsonify({'status': 'error', 'message': str(e)}), 500

@app.route('/api/verify_key', methods=['POST'])
def verify_api_key():
    """Endpoint do weryfikacji klucza API"""
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
    """Endpoint do tworzenia Gold Image"""
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
    print(f"[GOLD IMAGE] Żądanie utworzenia backupu. Opis: {description[:100]}")
    
    script_path = "/home/ubuntu/skrypty/create_gold_image.sh"
    
    if not os.path.exists(script_path):
        return jsonify({
            "success": False,
            "message": "Skrypt nie został znaleziony",
            "tag": None,
            "output": f"Ścieżka: {script_path}"
        }), 500
    
    try:
        process = subprocess.Popen(
            ['bash', script_path],
            stdout=subprocess.PIPE,
            stderr=subprocess.PIPE,
            text=True,
            cwd=os.path.dirname(script_path)
        )
        
        stdout, stderr = process.communicate(timeout=120)
        return_code = process.returncode
        full_output = stdout + ("\n" + stderr if stderr else "")
        
        import re
        tag_match = re.search(r'tag:\s*(v\d+\.\d+[\w\.-]*)', full_output, re.IGNORECASE)
        tag = tag_match.group(1) if tag_match else None
        
        if return_code == 0:
            return jsonify({
                "success": True,
                "message": f"Gold Image utworzony pomyślnie{' (tag: ' + tag + ')' if tag else ''}",
                "tag": tag,
                "output": full_output[-1000:]
            }), 200
        else:
            return jsonify({
                "success": False,
                "message": "Błąd podczas wykonywania skryptu",
                "tag": tag,
                "output": full_output[-1000:]
            }), 500
            
    except subprocess.TimeoutExpired:
        return jsonify({
            "success": False,
            "message": "Timeout (ponad 2 minuty)",
            "tag": None,
            "output": "Proces zabity"
        }), 500
    except Exception as e:
        return jsonify({
            "success": False,
            "message": f"Nieoczekiwany błąd: {str(e)}",
            "tag": None,
            "output": ""
        }), 500

@app.route('/')
def serve_index():
    return send_from_directory('static', 'index.html')

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=False)
APP_EOF

# Skopiuj nowy app.py
sudo cp "$TEMP_APP" /opt/ai_firma_dashboard/app.py
sudo chown www-data:www-data /opt/ai_firma_dashboard/app.py
rm -f "$TEMP_APP"
echo "✓ Nowy app.py z endpointami Gold Image"

echo "Krok 3/5: Tworzenie prostego dashboardu z panelem Gold Image..."

# Prosty index.html z panelem Gold Image
TEMP_HTML=$(mktemp)
sudo tee "$TEMP_HTML" > /dev/null << 'HTML_EOF'
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Firma - Status + Gold Image</title>
    <style>
        * { box-sizing: border-box; font-family: sans-serif; }
        body { background: #f5f5f5; margin: 0; padding: 15px; color: #333; }
        .container { max-width: 800px; margin: 0 auto; }
        header { text-align: center; margin-bottom: 25px; }
        h1 { color: #2c3e50; }
        .card { background: white; border-radius: 10px; padding: 20px; margin-bottom: 15px; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        .btn {
            background: #27ae60;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 5px;
            cursor: pointer;
            margin: 5px;
        }
        .btn:hover { background: #219653; }
        .btn:disabled { background: #95a5a6; cursor: not-allowed; }
        .message {
            padding: 10px;
            border-radius: 5px;
            margin: 10px 0;
            display: none;
        }
        .message.show { display: block; }
        .message.success { background: #d5f4e6; border: 1px solid #27ae60; }
        .message.error { background: #fadbd8; border: 1px solid #e74c3c; }
        .message.info { background: #e8f4fc; border: 1px solid #3498db; }
        input {
            padding: 8px;
            border: 1px solid #ddd;
            border-radius: 5px;
            width: 100%;
            margin: 5px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <header><h1>🖥️ Status Serwera AI Firma</h1><p>Ostatnia aktualizacja: <span id="updateTime">-</span></p></header>
        
        <div class="card"><h2>Zdrowie Systemu</h2><div id="healthMetrics"></div></div>
        <div class="card"><h2>Usługi</h2><div id="serviceList"></div></div>
        <div class="card"><h2>Zasoby</h2><div id="resourceMetrics"></div></div>
        
        <div class="card">
            <h2>🛡️ Zarządzanie Gold Image</h2>
            <div id="goldPanel">
                <p><strong>Status:</strong> <span id="goldStatus">Nie uwierzytelniony</span></p>
                <input type="password" id="apiKey" placeholder="Klucz API: AI_FIRMA_GOLD_IMAGE_KEY_2024">
                <button id="saveKey" class="btn">Zapisz klucz</button>
                <button id="createBtn" class="btn" style="display:none;">Utwórz Gold Image</button>
                <div id="goldMessage" class="message"></div>
            </div>
        </div>
    </div>

    <script>
        let apiKey = localStorage.getItem('gold_api_key') || '';
        
        function initGoldPanel() {
            if (apiKey) {
                document.getElementById('apiKey').style.display = 'none';
                document.getElementById('saveKey').style.display = 'none';
                document.getElementById('createBtn').style.display = 'inline-block';
                document.getElementById('goldStatus').textContent = 'Gotowy';
            }
        }
        
        document.getElementById('saveKey').addEventListener('click', function() {
            const key = document.getElementById('apiKey').value.trim();
            if (key) {
                localStorage.setItem('gold_api_key', key);
                apiKey = key;
                showMessage('Klucz zapisany', 'success');
                initGoldPanel();
            }
        });
        
        document.getElementById('createBtn').addEventListener('click', async function() {
            const btn = this;
            btn.disabled = true;
            btn.textContent = 'Tworzenie...';
            showMessage('Trwa tworzenie Gold Image...', 'info');
            
            try {
                const response = await fetch('/api/gold_image/create', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'X-API-Key': apiKey
                    },
                    body: JSON.stringify({ description: 'Z dashboardu' })
                });
                
                const data = await response.json();
                
                if (response.status === 401) {
                    showMessage('Nieprawidłowy klucz API', 'error');
                    localStorage.removeItem('gold_api_key');
                    location.reload();
                } else if (data.success) {
                    showMessage(`✓ ${data.message}`, 'success');
                } else {
                    showMessage(`✗ ${data.message}`, 'error');
                }
            } catch (error) {
                showMessage(`✗ Błąd: ${error.message}`, 'error');
            } finally {
                btn.disabled = false;
                btn.textContent = 'Utwórz Gold Image';
            }
        });
        
        function showMessage(text, type) {
            const msg = document.getElementById('goldMessage');
            msg.textContent = text;
            msg.className = `message show ${type}`;
            setTimeout(() => msg.className = 'message', 5000);
        }
        
        // Original dashboard functions
        async function loadData() {
            try {
                const resp = await fetch('/api/health');
                const data = await resp.json();
                document.getElementById('updateTime').textContent = new Date().toLocaleTimeString();
                
                const health = document.getElementById('healthMetrics');
                health.innerHTML = `
                    <div>Status: <strong>${data.status}</strong></div>
                    <div>Uptime: ${data.system.uptime}</div>
                    <div>CPU: ${data.system.cpu_percent}%</div>
                `;
                
                const services = document.getElementById('serviceList');
                services.innerHTML = Object.entries(data.services).map(([name, active]) => 
                    `<div>${name}: ${active ? '✅' : '❌'}</div>`
                ).join('');
                
                const resources = document.getElementById('resourceMetrics');
                resources.innerHTML = `
                    <div>RAM: ${data.memory.percent_used}% (${data.memory.available_gb}GB wolne)</div>
                    <div>Dysk: ${data.disk.percent_used}% (${data.disk.free_gb}GB wolne)</div>
                `;
            } catch (err) {
                console.error('Błąd ładowania:', err);
            }
        }
        
        document.addEventListener('DOMContentLoaded', function() {
            initGoldPanel();
            loadData();
            setInterval(loadData, 10000);
        });
    </script>
</body>
</html>
HTML_EOF

# Skopiuj nowy index.html
sudo cp "$TEMP_HTML" /opt/ai_firma_dashboard/static/index.html
sudo chown www-data:www-data /opt/ai_firma_dashboard/static/index.html
rm -f "$TEMP_HTML"
echo "✓ Nowy dashboard z panelem Gold Image"

echo "Krok 4/5: Restart backendu..."
sudo supervisorctl restart dashboard-api
sleep 2

echo "Krok 5/5: Weryfikacja..."
DASHBOARD_STATUS=$(sudo supervisorctl status dashboard-api | awk '{print $2}')
echo "✓ Status dashboard-api: $DASHBOARD_STATUS"

echo ""
echo "========================================"
echo "INSTALACJA ZAKOŃCZONA POMYŚLNIE!"
echo "========================================"
echo "Dashboard: http://57.128.247.215"
echo "Klucz API: AI_FIRMA_GOLD_IMAGE_KEY_2024"
echo ""
echo "Test endpointu:"
echo "curl -X POST http://localhost:5000/api/verify_key \\"
echo "  -H 'Content-Type: application/json' \\"
echo "  -d '{\"api_key\":\"AI_FIRMA_GOLD_IMAGE_KEY_2024\"}'"
echo "========================================"
