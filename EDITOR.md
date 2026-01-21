#!/bin/bash
# Poprawka endpointu backup_status() - czytanie logów cron zamiast dat modyfikacji
APP_PATH="/opt/ai_firma_dashboard/app.py"

echo "=== Poprawka endpointu - rzeczywiste daty wykonania backupów ==="

echo "[1] Obecny problem:"
echo "    Endpoint pokazuje daty modyfikacji skryptów, a nie daty wykonania"

echo "[2] Logi cron które istnieją:"
ls -la /var/log/*backup.log 2>/dev/null

echo "[3] Tworzę poprawioną wersję funkcji backup_status()..."

# Tworzenie tymczasowego pliku z nową funkcją
TEMP_FILE=$(mktemp)
cat > "$TEMP_FILE" << 'EOF'
def backup_status():
    """Endpoint sprawdzania statusu backupów (RZECZYWISTE WYKONANIA)"""
    print(f"[BACKUP] Żądanie statusu o {datetime.now()}")
    import os
    import subprocess
    
    # Mapowanie logów cron do typów backupów
    backup_logs = {
        "daily_dashboard": "/var/log/dashboard_backup.log",
        "daily_skrypty": "/var/log/skrypty_backup.log", 
        "weekly_full": "/var/log/weekly_backup.log"
    }
    
    status_data = {}
    
    for backup_type, log_path in backup_logs.items():
        if os.path.exists(log_path):
            try:
                # Pobierz ostatnią linię z loga
                result = subprocess.run(
                    ["tail", "-1", log_path],
                    capture_output=True,
                    text=True,
                    timeout=5
                )
                
                if result.returncode == 0 and result.stdout.strip():
                    # Spróbuj wyciągnąć datę z loga
                    last_line = result.stdout.strip()
                    
                    # Pobierz czas modyfikacji pliku log (jako przybliżenie)
                    mtime = os.path.getmtime(log_path)
                    from datetime import datetime
                    last_run = datetime.fromtimestamp(mtime).isoformat()
                    
                    # Sprawdź czy log jest świeży (ostatnie 2 dni)
                    time_diff = datetime.now().timestamp() - mtime
                    status = "ok" if time_diff < 172800 else "stale"
                    
                    status_data[backup_type] = {
                        "last_run": last_run,
                        "status": status,
                        "log_path": log_path,
                        "last_log_line": last_line[:100]  # Pierwsze 100 znaków
                    }
                else:
                    status_data[backup_type] = {
                        "last_run": None,
                        "status": "empty_log",
                        "log_path": log_path,
                        "error": "Log file empty"
                    }
                    
            except Exception as e:
                status_data[backup_type] = {
                    "last_run": None,
                    "status": "error",
                    "log_path": log_path,
                    "error": str(e)
                }
        else:
            status_data[backup_type] = {
                "last_run": None,
                "status": "missing_log",
                "log_path": log_path
            }
    
    # Dodaj status Gold Image
    gold_script = "/home/ubuntu/skrypty/create_gold_image.sh"
    gold_report = "/home/ubuntu/ai_firma_dokumenty/GOLD_IMAGE_v1.0.md"
    
    if os.path.exists(gold_report):
        mtime = os.path.getmtime(gold_report)
        from datetime import datetime
        status_data["gold_image"] = {
            "last_created": datetime.fromtimestamp(mtime).isoformat(),
            "status": "ready",
            "report_path": gold_report
        }
    elif os.path.exists(gold_script):
        status_data["gold_image"] = {
            "last_created": None,
            "status": "script_available",
            "script_path": gold_script
        }
    else:
        status_data["gold_image"] = {
            "last_created": None,
            "status": "not_configured"
        }
    
    return jsonify({
        "status": "success",
        "data": status_data,
        "message": "Rzeczywisty status backupów z logów cron",
        "timestamp": datetime.now().isoformat()
    }), 200
EOF

# 4. Wykonaj zamianę w app.py
echo "[4] Wykonuję zamianę funkcji..."
python3 -c "
import re

with open('$APP_PATH', 'r') as f:
    content = f.read()

with open('$TEMP_FILE', 'r') as f:
    new_function = f.read()

# Zamień starą funkcję na nową
pattern = r'def backup_status\(\):.*?^\s*def '
new_content = re.sub(pattern, new_function + '\n\ndef ', content, flags=re.DOTALL | re.MULTILINE)

with open('$APP_PATH', 'w') as f:
    f.write(new_content)

print('Zamiana wykonana')
"

# 5. Weryfikacja
echo "[5] Nowa funkcja (pierwsze 30 linii):"
grep -n -A 30 "def backup_status()" "$APP_PATH" | head -35

# 6. Test składni
echo "[6] Test składni:"
cd /opt/ai_firma_dashboard && python3 -m py_compile app.py && echo "✅ Składnia poprawna" || echo "❌ Błąd składni"

# 7. Czyszczenie
rm -f "$TEMP_FILE"

echo "[7] Instrukcja testu:"
echo "    sudo pkill -f 'gunicorn.*app:app'"
echo "    cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon"
echo "    sleep 2 && curl http://127.0.0.1:5000/api/backup/status"

echo "=== Gotowe ==="
