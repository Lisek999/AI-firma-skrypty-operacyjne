#!/bin/bash
# Endpoint backup_status() z czytelnym podsumowaniem w message
APP_PATH="/opt/ai_firma_dashboard/app.py"

echo "=== Endpoint z czytelnym podsumowaniem ==="

echo "[1] Tworzę wersję z czytelnym message..."

TEMP_FILE=$(mktemp)
cat > "$TEMP_FILE" << 'EOF'
def backup_status():
    """Endpoint sprawdzania statusu backupów z czytelnym podsumowaniem"""
    print(f"[BACKUP] Żądanie statusu o {datetime.now()}")
    import os
    
    # Pobierz rzeczywiste daty modyfikacji skryptów
    backup_info = {}
    
    scripts = {
        "daily": "/home/ubuntu/skrypty/git_daily_dashboard_backup.sh",
        "weekly": "/home/ubuntu/skrypty/git_weekly_full_backup.sh",
        "gold_image": "/home/ubuntu/skrypty/create_gold_image.sh"
    }
    
    for name, path in scripts.items():
        if os.path.exists(path):
            mtime = os.path.getmtime(path)
            last_run = datetime.fromtimestamp(mtime).strftime("%Y-%m-%d %H:%M")
            backup_info[name] = last_run
        else:
            backup_info[name] = "brak skryptu"
    
    # Status Gold Image
    gold_report = "/home/ubuntu/ai_firma_dokumenty/GOLD_IMAGE_v1.0.md"
    if os.path.exists(gold_report):
        mtime = os.path.getmtime(gold_report)
        backup_info["gold_image_created"] = datetime.fromtimestamp(mtime).strftime("%Y-%m-%d %H:%M")
        gold_status = "gotowy"
    else:
        backup_info["gold_image_created"] = "nie utworzony"
        gold_status = "niegotowy"
    
    # Stwórz czytelne message
    message_lines = [
        "Status backupów:",
        f"• Daily: {backup_info.get('daily', 'brak')}",
        f"• Weekly: {backup_info.get('weekly', 'brak')}",
        f"• Gold Image: {gold_status} ({backup_info.get('gold_image_created', 'brak')})",
        f"• Skrypt Gold Image: {backup_info.get('gold_image', 'brak')}"
    ]
    
    readable_message = "\n".join(message_lines)
    
    return jsonify({
        "status": "success",
        "data": {
            "daily": {"last_run": backup_info.get("daily"), "status": "ok" if backup_info.get("daily") != "brak skryptu" else "error"},
            "weekly": {"last_run": backup_info.get("weekly"), "status": "ok" if backup_info.get("weekly") != "brak skryptu" else "error"},
            "gold_image": {"last_created": backup_info.get("gold_image_created"), "status": gold_status}
        },
        "message": readable_message,
        "timestamp": datetime.now().isoformat()
    }), 200
EOF

# 2. Wykonaj zamianę
echo "[2] Wykonuję zamianę..."
python3 -c "
import re

with open('$APP_PATH', 'r') as f:
    content = f.read()

with open('$TEMP_FILE', 'r') as f:
    new_function = f.read()

pattern = r'def backup_status\(\):.*?^\s*def '
new_content = re.sub(pattern, new_function + '\n\ndef ', content, flags=re.DOTALL | re.MULTILINE)

with open('$APP_PATH', 'w') as f:
    f.write(new_content)

print('✅ Zamiana wykonana')
"

# 3. Weryfikacja
echo "[3] Nowa funkcja (pierwsze 15 linii):"
grep -n -A 15 "def backup_status()" "$APP_PATH"

# 4. Test składni
echo "[4] Test składni:"
cd /opt/ai_firma_dashboard && python3 -m py_compile app.py && echo "✅ OK" || echo "❌ Błąd"

rm -f "$TEMP_FILE"

echo "[5] Test:"
echo "    sudo pkill -f 'gunicorn.*app:app'"
echo "    cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 0.0.0.0:5000 app:app --daemon"
echo "    sleep 2 && curl -s http://57.128.247.215:5000/api/backup/status | python3 -m json.tool | grep -A10 message"

echo "=== Gotowe ==="
