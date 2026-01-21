#!/bin/bash
# Naprawa duplikatu importu datetime w backup_status()
APP_PATH="/opt/ai_firma_dashboard/app.py"

echo "=== Naprawa błędu importu datetime ==="

echo "[1] Obecny problem:"
echo "    Plik app.py już importuje 'from datetime import datetime' na początku"
echo "    Funkcja backup_status() też importuje - to powoduje konflikt"

echo "[2] Usuwam duplikat importu z funkcji backup_status()..."
# Usuń linię 'from datetime import datetime' z funkcji backup_status()
sed -i '/def backup_status/,/^def backup_run_manual/ { /from datetime import datetime/d }' "$APP_PATH"

echo "[3] Weryfikacja zmian (linie 180-210):"
sed -n '180,210p' "$APP_PATH"

echo "[4] Test składni Python:"
cd /opt/ai_firma_dashboard && python3 -m py_compile app.py && echo "✅ Składnia poprawna" || echo "❌ Błąd składni"

echo "[5] Instrukcja restartu:"
echo "    sudo pkill -f 'gunicorn.*app:app'"
echo "    cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon"
echo "    sleep 2 && curl http://127.0.0.1:5000/api/backup/status"

echo "=== Gotowe ==="
