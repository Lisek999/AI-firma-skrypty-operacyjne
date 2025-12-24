## deploy_dashboard.sh
Kompletna instalacja Status Dashboard AI Firma.
```bash
#!/bin/bash
# deploy_dashboard.sh - Instalacja panelu statusu (v1.0) - PRODUKCJA
set -e
LOG_FILE="/tmp/dashboard_deploy_$(date +%Y%m%d_%H%M%S).log"
exec > >(tee -a "$LOG_FILE") 2>&1
echo "Rozpoczynam wdrażanie Status Dashboard v1.0..."
echo "Pełny log: $LOG_FILE"
echo ""

# 1. INSTALACJA ZALEŻNOŚCI
echo "[1/6] Instalacja wymaganych pakietów..."
sudo apt update
sudo apt install -y python3-pip python3-venv gunicorn nginx supervisor
echo "✅ Zależności systemowe zainstalowane."
echo ""
echo "[1a/6] Tworzenie virtual environment Python..."
sudo mkdir -p /opt/ai_firma_dashboard
cd /opt/ai_firma_dashboard
sudo python3 -m venv venv
sudo ./venv/bin/pip install flask psutil gunicorn
echo "✅ Virtual environment i pakiety Python gotowe."
echo ""

# 2. TWORZENIE STRUKTURY KATALOGÓW
echo "[2/6] Tworzenie struktury katalogów..."
sudo mkdir -p /opt/ai_firma_dashboard/static
sudo chown -R www-data:www-data /opt/ai_firma_dashboard
cd /opt/ai_firma_dashboard
echo "✅ Katalogi utworzone."
echo ""

# 3. ZAPISYWANIE PLIKÓW ŹRÓDŁOWYCH (PRZEZ GETSCRIPT)
echo "[3/6] Pobieranie i zapisywanie plików źródłowych..."
getscript SOURCE_dashboard_api.py > app.py
getscript SOURCE_dashboard_frontend.html > static/index.html
getscript SOURCE_nginx_dashboard.conf > /tmp/dashboard.nginx.conf
getscript SOURCE_supervisor_dashboard.conf > /tmp/dashboard.supervisor.conf
echo "✅ Pliki źródłowe pobrane."
echo ""

# 4. KONFIGURACJA NGINX (Z WALIDACJĄ I BACKUPEM)
echo "[4/6] Konfiguracja Nginx..."
if [ -f /etc/nginx/sites-available/dashboard ]; then
    sudo cp /etc/nginx/sites-available/dashboard /etc/nginx/sites-available/dashboard.backup.$(date +%s)
fi
sudo cp /tmp/dashboard.nginx.conf /etc/nginx/sites-available/dashboard
sudo ln -sf /etc/nginx/sites-available/dashboard /etc/nginx/sites-enabled/
echo "✅ Plik konfiguracyjny Nginx zapisany. Walidacja..."
if sudo nginx -t; then
    sudo systemctl reload nginx
    echo "✅ Konfiguracja Nginx poprawna, usługa przeładowana."
else
    echo "❌ BŁĄD: Konfiguracja Nginx nieprawidłowa. Przywracam backup..."
    if [ -f /etc/nginx/sites-available/dashboard.backup.* ]; then
        sudo cp /etc/nginx/sites-available/dashboard.backup.* /etc/nginx/sites-available/dashboard
    fi
    sudo nginx -t
    exit 1
fi
echo ""

# 5. KONFIGURACJA SUPERVISORA
echo "[5/6] Konfiguracja Supervisora..."
sudo cp /tmp/dashboard.supervisor.conf /etc/supervisor/conf.d/dashboard-api.conf
# POPRAWKA: użyj gunicorn z virtual environment
sudo sed -i 's|command=/usr/bin/gunicorn|command=/opt/ai_firma_dashboard/venv/bin/gunicorn|' /etc/supervisor/conf.d/dashboard-api.conf
sudo supervisorctl reread
sudo supervisorctl update
sleep 2
if sudo supervisorctl status dashboard-api | grep -q "RUNNING"; then
    echo "✅ Usługa dashboard-api uruchomiona w Supervisorze."
else
    echo "❌ Usługa nie działa. Sprawdź logi: supervisorctl tail dashboard-api"
    exit 1
fi
echo ""

# 6. TEST KOŃCOWY
echo "[6/6] Testowanie wdrożenia..."
sleep 2
if curl -s http://localhost/api/health | grep -q "online"; then
    echo "✅ Backend API odpowiada poprawnie."
    echo "✅ Frontend powinien być dostępny pod adresem IP serwera na porcie 80."
    echo ""
    echo "======================================================================"
    echo "🎉 DASHBOARD ZOSTAŁ POMYŚLNIE WDROŻONY!"
    echo "======================================================================"
    echo "Otwórz w przeglądarce: http://$(curl -s ifconfig.me)"
    echo "Logi instalacji: $LOG_FILE"
    echo "Logi aplikacji: /var/log/dashboard-api.*.log"
    echo "======================================================================"
else
    echo "❌ Backend API nie odpowiada. Sprawdź logi."
    exit 1
fi
```

## update_dashboard.sh
Bezpieczna aktualizacja kodu dashboardu.
```bash
#!/bin/bash
cd /opt/ai_firma_dashboard || exit 1
sudo git fetch origin
sudo git reset --hard origin/main
sudo supervisorctl restart dashboard-api
echo "Dashboard zaktualizowany."
```

## PLIKI ŹRÓDŁOWE DASHBOARD

### SOURCE_dashboard_api.py
```python
from flask import Flask, jsonify, send_from_directory
import psutil
import subprocess
import os
import time

app = Flask(__name__)

def get_service_status(service_name):
    """Bezpiecznie sprawdza status usługi systemd."""
    try:
        result = subprocess.run(['systemctl', 'is-active', service_name],
                                capture_output=True, text=True, timeout=2)
        return result.stdout.strip() == 'active'
    except:
        return False

@app.route('/api/health')
def get_health():
    """Zwraca ONLY bezpieczne, ogólne metryki systemowe."""
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

@app.route('/')
def serve_index():
    """Serwuje frontend."""
    return send_from_directory('static', 'index.html')

if __name__ == '__main__':
    app.run(host='127.0.0.1', port=5000, debug=False)
```

### SOURCE_dashboard_frontend.html
```html
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI Firma - Status Serwera</title>
    <style>
        * { box-sizing: border-box; font-family: 'Segoe UI', sans-serif; }
        body { background: #f5f5f5; margin: 0; padding: 15px; color: #333; }
        .container { max-width: 800px; margin: 0 auto; }
        header { text-align: center; margin-bottom: 25px; }
        h1 { color: #2c3e50; }
        .card { background: white; border-radius: 10px; padding: 20px; margin-bottom: 15px; box-shadow: 0 3px 10px rgba(0,0,0,0.08); }
        .metric { display: flex; justify-content: space-between; margin: 10px 0; }
        .metric .value { font-weight: bold; }
        .status-online { color: #27ae60; }
        .status-offline { color: #e74c3c; }
        .service-list { display: grid; grid-template-columns: repeat(auto-fill, minmax(180px, 1fr)); gap: 10px; }
        .service { padding: 10px; border-radius: 6px; background: #ecf0f1; text-align: center; }
        @media (max-width: 600px) { .service-list { grid-template-columns: 1fr; } }
    </style>
</head>
<body>
    <div class="container">
        <header><h1>🖥️ Status Serwera AI Firma</h1><p>Ostatnia aktualizacja: <span id="updateTime">-</span></p></header>
        <div class="card"><h2>Zdrowie Systemu</h2><div id="healthMetrics"></div></div>
        <div class="card"><h2>Usługi</h2><div class="service-list" id="serviceList"></div></div>
        <div class="card"><h2>Zasoby</h2><div id="resourceMetrics"></div></div>
    </div>
    <script>
        async function loadData() {
            try {
                const resp = await fetch('/api/health');
                const data = await resp.json();
                document.getElementById('updateTime').textContent = new Date().toLocaleTimeString();
                renderHealth(data);
                renderServices(data.services);
                renderResources(data.memory, data.disk, data.system);
            } catch (err) {
                document.getElementById('healthMetrics').innerHTML = `<p style="color:red;">Błąd ładowania: ${err}</p>`;
            }
        }
        function renderHealth(data) {
            const el = document.getElementById('healthMetrics');
            el.innerHTML = `
                <div class="metric"><span>Status:</span><span class="value status-online">${data.status.toUpperCase()}</span></div>
                <div class="metric"><span>Uptime:</span><span class="value">${data.system.uptime}</span></div>
                <div class="metric"><span>Load Avg:</span><span class="value">${data.system.load_avg.join(', ')}</span></div>
                <div class="metric"><span>CPU:</span><span class="value">${data.system.cpu_percent}%</span></div>
            `;
        }
        function renderServices(services) {
            const el = document.getElementById('serviceList');
            el.innerHTML = Object.entries(services).map(([name, isUp]) => `
                <div class="service">
                    <strong>${name}</strong><br>
                    <span class="${isUp ? 'status-online' : 'status-offline'}">${isUp ? '✅ Aktywna' : '❌ Nieaktywna'}</span>
                </div>
            `).join('');
        }
        function renderResources(mem, disk, sys) {
            const el = document.getElementById('resourceMetrics');
            el.innerHTML = `
                <div class="metric"><span>Pamięć RAM:</span><span class="value">${mem.percent_used}% (${mem.available_gb} GB wolne)</span></div>
                <div class="metric"><span>Dysk (/) :</span><span class="value">${disk.percent_used}% (${disk.free_gb} GB wolne)</span></div>
            `;
        }
        loadData();
        setInterval(loadData, 10000);
    </script>
</body>
</html>
```

### SOURCE_nginx_dashboard.conf
```nginx
# /etc/nginx/sites-available/dashboard
server {
    listen 80;
    server_name _;
    root /opt/ai_firma_dashboard/static;
    index index.html;

    location /api/ {
        access_log off;
        proxy_pass http://127.0.0.1:5000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

    location / {
        try_files $uri $uri/ =404;
    }
}
```

### SOURCE_supervisor_dashboard.conf
```ini
; /etc/supervisor/conf.d/dashboard-api.conf
[program:dashboard-api]
command=/usr/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app
directory=/opt/ai_firma_dashboard
user=www-data
autostart=true
autorestart=true
stopasgroup=true
killasgroup=true
stderr_logfile=/var/log/dashboard-api.err.log
stdout_logfile=/var/log/dashboard-api.out.log
environment=PYTHONUNBUFFERED="1"
```
