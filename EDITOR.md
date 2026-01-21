#!/bin/bash
# Naprawa nasłuchiwania gunicorn - zmiana z 127.0.0.1 na 0.0.0.0
echo "=== Naprawa dostępu do aplikacji ==="

echo "[1] Zabijam obecne procesy gunicorn..."
sudo pkill -f "gunicorn.*app:app"

echo "[2] Sprawdzam czy port 5000 jest wolny..."
sudo netstat -tlnp | grep :5000 || echo "Port 5000 wolny"

echo "[3] Uruchamiam gunicorn na 0.0.0.0:5000 (dostępny z zewnątrz)..."
cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 0.0.0.0:5000 app:app --daemon

echo "[4] Sprawdzam nasłuchiwanie..."
sleep 1
sudo netstat -tlnp | grep :5000

echo "[5] Test lokalny:"
curl -s http://127.0.0.1:5000/api/health | head -1

echo "[6] Test zewnętrzny (jeśli masz dostęp do innego serwera):"
echo "    curl -s http://57.128.247.215:5000/api/health"
echo "    curl -s http://57.128.247.215:5000/archive | grep -o '<title>.*</title>'"

echo "[7] WAŻNE: Jeśli działa na porcie 5000, możesz potrzebować:"
echo "    - Firewall: sudo ufw allow 5000"
echo "    - Proxy nginx (zalecane dla produkcji)"

echo "=== Gotowe ==="
