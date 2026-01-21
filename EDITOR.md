#!/bin/bash
# Naprawa duplikatów importu datetime
APP_PATH="/opt/ai_firma_dashboard/app.py"

echo "=== Naprawa duplikatów importu datetime ==="

# Usuń wszystkie 'from datetime import datetime' z funkcji backup_status()
sed -i '/def backup_status/,/^def backup_run_manual/ { /from datetime import datetime/d }' "$APP_PATH"

echo "Sprawdzam czy zostały usunięte:"
grep -n "from datetime import datetime" "$APP_PATH"

echo "Test składni:"
cd /opt/ai_firma_dashboard && python3 -m py_compile app.py && echo "✅ OK" || echo "❌ Błąd"

echo "Restart: sudo pkill -f 'gunicorn.*app:app' && cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon"

