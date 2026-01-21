#!/bin/bash
# Naprawa błędu składni w backup_status()
APP_PATH="/opt/ai_firma_dashboard/app.py"

echo "=== Naprawa błędu składni ==="

echo "[1] Naprawiam linię 223 (rozbity string)..."
sed -i '223s/readable_message = "/readable_message = "\\n"/' "$APP_PATH"

echo "[2] Sprawdzam naprawioną linię:"
sed -n '222,225p' "$APP_PATH"

echo "[3] Test składni:"
cd /opt/ai_firma_dashboard && python3 -m py_compile app.py && echo "✅ OK" || {
    echo "❌ Wciąż błąd, pokażę więcej:"
    python3 -m py_compile app.py 2>&1 | head -10
}

echo "[4] Restart i test:"
echo "    sudo pkill -f 'gunicorn.*app:app'"
echo "    cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 0.0.0.0:5000 app:app --daemon"
echo "    sleep 2 && curl -s http://57.128.247.215:5000/api/backup/status | python3 -c \"import sys, json; print(json.load(sys.stdin)['message'])\""

echo "=== Gotowe ==="

