### SOURCE_dashboard_api.py
```python
from flask import Flask, jsonify, send_from_directory
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

@app.route('/')
def serve_index():
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
