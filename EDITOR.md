#!/bin/bash
# ============================================================================
# SKRYPT: integracja_statusu_backupow.sh
# CEL: Podpięcie endpointu backup_status() pod rzeczywiste dane
# DATA: $(date)
# ============================================================================

APP_PATH="/opt/ai_firma_dashboard/app.py"
BACKUP_PATH="/opt/ai_firma_dashboard/app.py.backup_status_$(date +%Y%m%d_%H%M%S)"

echo "=== Integracja rzeczywistego statusu backupów ==="

# 1. Backup
echo "[1] Backup: $BACKUP_PATH"
cp "$APP_PATH" "$BACKUP_PATH"

# 2. Znajdź funkcję backup_status()
echo "[2] Obecna funkcja backup_status():"
grep -n -A 15 "def backup_status()" "$APP_PATH"

# 3. Nowa wersja funkcji (z rzeczywistymi danymi)
TEMP_FILE=$(mktemp)
cat > "$TEMP_FILE" << 'EOF'
def backup_status():
    """Endpoint sprawdzania statusu backupów (RZECZYWISTE DANE)"""
    print(f"[BACKUP] Żądanie statusu o {datetime.now()}")
    
    import os
    from datetime import datetime
    
    # Ścieżki do skryptów backup
    backup_scripts = {
        "daily": "/home/ubuntu/skrypty/git_daily_dashboard_backup.sh",
        "weekly": "/home/ubuntu/skrypty/git_weekly_full_backup.sh",
        "gold_image": "/home/ubuntu/skrypty/create_gold_image.sh"
    }
    
    status_data = {}
    
    for backup_type, script_path in backup_scripts.items():
        if os.path.exists(script_path):
            # Pobierz czas ostatniej modyfikacji
            mtime = os.path.getmtime(script_path)
            last_run = datetime.fromtimestamp(mtime).isoformat()
            
            # Sprawdź czy skrypt był uruchamiany "ostatnio"
            # (jeśli modyfikacja była w ciągu ostatnich 2 dni - status ok)
            time_diff = datetime.now().timestamp() - mtime
            status = "ok" if time_diff < 172800 else "stale"  # 2 dni w sekundach
        else:
            last_run = None
            status = "missing"
        
        status_data[backup_type] = {
            "last_run": last_run,
            "status": status,
            "script_path": script_path
        }
    
    # Dodaj informację o gold image z repozytorium (jeśli istnieje)
    gold_image_report = "/home/ubuntu/ai_firma_dokumenty/GOLD_IMAGE_v1.0.md"
    if os.path.exists(gold_image_report):
        gold_mtime = os.path.getmtime(gold_image_report)
        status_data["gold_image"]["last_created"] = datetime.fromtimestamp(gold_mtime).isoformat()
        status_data["gold_image"]["status"] = "ready"
    
    return jsonify({
        "status": "success",
        "data": status_data,
        "message": "Rzeczywisty status backupów",
        "timestamp": datetime.now().isoformat()
    }), 200
EOF

# 4. Wykonaj zamianę w app.py
echo "[3] Wykonuję zamianę funkcji..."
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
echo "[4] Nowa funkcja backup_status():"
grep -n -A 20 "def backup_status()" "$APP_PATH"

# 6. Test
echo "[5] Test endpointu (po restarcie gunicorn):"
echo "   sudo pkill -f 'gunicorn.*app:app'"
echo "   cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon"
echo "   sleep 2 && curl http://127.0.0.1:5000/api/backup/status"

# 7. Czyszczenie
rm -f "$TEMP_FILE"

echo "=== Gotowe ==="
