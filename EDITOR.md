#!/bin/bash
# Naprawa błędu składni w backup_status()
APP_PATH="/opt/ai_firma_dashboard/app.py"

echo "=== Naprawa błędu składni w backup_status() ==="

echo "[1] Znajduję i naprawiam rozbity string..."
# Znajdź linię z błędem i napraw
sed -i 's/readable_message = "\n".join(message_lines)/readable_message = "\\n".join(message_lines)/' "$APP_PATH"

echo "[2] Test składni:"
if cd /opt/ai_firma_dashboard && python3 -m py_compile app.py; then
    echo "✅ Składnia poprawna"
    
    echo "[3] Restart gunicorn:"
    sudo pkill -f "gunicorn.*app:app"
    cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 0.0.0.0:5000 app:app --daemon
    
    echo "[4] Test endpointu:"
    sleep 2
    curl -s http://57.128.247.215:5000/api/backup/status | python3 -c "
import sys, json
data = json.load(sys.stdin)
print('Message:')
print(data.get('message', 'Brak message'))
print()
print('Dane:')
print('Daily:', data.get('data', {}).get('daily', {}).get('last_run', 'brak'))
print('Weekly:', data.get('data', {}).get('weekly', {}).get('last_run', 'brak'))
print('Gold Image:', data.get('data', {}).get('gold_image', {}).get('last_created', 'brak'))
" 2>/dev/null || echo "Błąd parsowania"
    
else
    echo "❌ Błąd składni - pokazuję:"
    python3 -m py_compile app.py 2>&1
fi

echo "=== Gotowe ==="
