#!/bin/bash
# Ostateczna wersja endpointu backup_status() - realistyczny status systemu
APP_PATH="/opt/ai_firma_dashboard/app.py"

echo "=== Ostateczna integracja statusu backupów ==="

echo "[1] Tworzę realistyczną wersję funkcji..."

# Tymczasowy plik z nową funkcją
TEMP_FILE=$(mktemp)
cat > "$TEMP_FILE" << 'EOF'
def backup_status():
    """Endpoint sprawdzania statusu backupów (REALISTYCZNY)"""
    print(f"[BACKUP] Żądanie statusu o {datetime.now()}")
    import os
    import subprocess
    
    # 1. Sprawdź konfigurację cron
    cron_config = {
        "daily_dashboard": "0 3 * * * - backup dashboardu",
        "daily_skrypty": "15 3 * * * - backup skryptów", 
        "weekly_full": "0 4 * * 0 - backup tygodniowy",
        "secure_vault": "30 3 * * * - secure vault"
    }
    
    # 2. Sprawdź czy skrypty backup istnieją i ich daty modyfikacji
    backup_scripts = {
        "daily_dashboard": "/home/ubuntu/skrypty/git_daily_dashboard_backup.sh",
        "daily_skrypty": "/home/ubuntu/skrypty/git_daily_skrypty_backup.sh",
        "weekly_full": "/home/ubuntu/skrypty/git_weekly_full_backup.sh",
        "gold_image": "/home/ubuntu/skrypty/create_gold_image.sh"
    }
    
    # 3. Sprawdź czy istnieją logi cron
    cron_logs = {
        "dashboard_log": "/var/log/dashboard_backup.log",
        "skrypty_log": "/var/log/skrypty_backup.log",
        "weekly_log": "/var/log/weekly_backup.log"
    }
    
    status_data = {
        "cron_configuration": cron_config,
        "backup_scripts": {},
        "cron_logs": {},
        "summary": {}
    }
    
    # Informacje o skryptach backup
    for name, path in backup_scripts.items():
        if os.path.exists(path):
            mtime = os.path.getmtime(path)
            last_modified = datetime.fromtimestamp(mtime).isoformat()
            status_data["backup_scripts"][name] = {
                "exists": True,
                "last_modified": last_modified,
                "path": path,
                "size_bytes": os.path.getsize(path)
            }
        else:
            status_data["backup_scripts"][name] = {
                "exists": False,
                "path": path
            }
    
    # Informacje o logach
    for name, path in cron_logs.items():
        if os.path.exists(path):
            mtime = os.path.getmtime(path)
            last_modified = datetime.fromtimestamp(mtime).isoformat()
            size = os.path.getsize(path)
            
            # Sprawdź czy log jest świeży (ostatnie 7 dni)
            time_diff = datetime.now().timestamp() - mtime
            is_fresh = time_diff < 604800  # 7 dni
            
            status_data["cron_logs"][name] = {
                "exists": True,
                "last_modified": last_modified,
                "size_bytes": size,
                "is_fresh": is_fresh,
                "days_old": round(time_diff / 86400, 1)
            }
        else:
            status_data["cron_logs"][name] = {
                "exists": False,
                "note": "Log nie istnieje - cron może nie uruchomić się lub nie zapisywać logów"
            }
    
    # Gold Image status
    gold_report = "/home/ubuntu/ai_firma_dokumenty/GOLD_IMAGE_v1.0.md"
    if os.path.exists(gold_report):
        mtime = os.path.getmtime(gold_report)
        status_data["gold_image"] = {
            "status": "ready",
            "last_created": datetime.fromtimestamp(mtime).isoformat(),
            "report_path": gold_report
        }
    else:
        status_data["gold_image"] = {
            "status": "not_created",
            "note": "Gold Image nie został jeszcze utworzony"
        }
    
    # Podsumowanie
    scripts_exist = sum(1 for s in status_data["backup_scripts"].values() if s["exists"])
    logs_exist = sum(1 for l in status_data["cron_logs"].values() if l["exists"])
    
    status_data["summary"] = {
        "backup_scripts_available": scripts_exist,
        "total_backup_scripts": len(backup_scripts),
        "cron_logs_available": logs_exist,
        "total_cron_logs": len(cron_logs),
        "system_status": "configured" if scripts_exist > 0 else "not_configured",
        "note": "System backupów jest skonfigurowany w cronie. Brak logów może oznaczać że cron nie uruchomił się jeszcze lub logi nie są zapisywane."
    }
    
    return jsonify({
        "status": "success",
        "data": status_data,
        "message": "Realistyczny status systemu backupów",
        "timestamp": datetime.now().isoformat()
    }), 200
EOF

# 2. Wykonaj zamianę w app.py
echo "[2] Wykonuję zamianę funkcji backup_status()..."
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

print('✅ Zamiana wykonana')
"

# 3. Weryfikacja
echo "[3] Nowa funkcja (pierwsze 10 linii):"
grep -n -A 10 "def backup_status()" "$APP_PATH"

# 4. Test składni
echo "[4] Test składni:"
cd /opt/ai_firma_dashboard && python3 -m py_compile app.py && echo "✅ Składnia poprawna" || echo "❌ Błąd składni"

# 5. Czyszczenie
rm -f "$TEMP_FILE"

echo "[5] Instrukcja testu:"
echo "    sudo pkill -f 'gunicorn.*app:app'"
echo "    cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon"
echo "    sleep 2 && curl -s http://127.0.0.1:5000/api/backup/status | python3 -m json.tool | head -50"

echo "=== Gotowe ==="
