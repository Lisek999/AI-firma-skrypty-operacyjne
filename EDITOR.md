#!/bin/bash
# SKRYPT 1: Aktualizacja backendu (app.py) o endpointy Gold Image i Backup
# Autor: Wojtek (AI)
# Data: $(date)
# Cel: Dodanie 5 endpointów API do działającej aplikacji Flask

BACKUP_DIR="/opt/ai_firma_dashboard/backups"
mkdir -p "$BACKUP_DIR"
BACKUP_FILE="$BACKUP_DIR/app.py.backup_before_gold_image_$(date +%Y%m%d_%H%M%S)"
cp /opt/ai_firma_dashboard/app.py "$BACKUP_FILE"
echo "Utworzono backup: $BACKUP_FILE"

# Definicja nowych endpointów do wstawienia
NEW_ENDPOINTS='@app.route(\x27/api/gold_image/manage\x27, methods=[\x27POST\x27])
def gold_image_manage():
    """Endpoint do zarządzania Gold Image (testowy)"""
    print(f"[GOLD_IMAGE] Żądanie zarządzania o {datetime.now()}")
    return jsonify({
        "status": "success",
        "message": "Endpoint Gold Image /manage jest aktywny (tryb testowy).",
        "timestamp": datetime.now().isoformat()
    }), 200

@app.route(\x27/api/backup/status\x27, methods=[\x27GET\x27])
def backup_status():
    """Endpoint sprawdzania statusu backupów"""
    print(f"[BACKUP] Żądanie statusu o {datetime.now()}")
    return jsonify({
        "status": "success",
        "data": {
            "daily": {"last_run": "2024-01-18T02:00:00", "status": "ok"},
            "weekly": {"last_run": "2024-01-14T00:00:00", "status": "ok"},
            "gold_image": {"last_created": "2024-01-10T12:00:00", "status": "ready"}
        },
        "message": "Statusy backupów (dane testowe)"
    }), 200

@app.route(\x27/api/backup/run_manual\x27, methods=[\x27POST\x27])
def backup_run_manual():
    """Endpoint uruchamiania ręcznego backupu"""
    print(f"[BACKUP] Żądanie ręcznego backupu o {datetime.now()}")
    return jsonify({
        "status": "success",
        "message": "Rozpoczęto ręczny backup (symulacja).",
        "job_id": "backup_manual_" + str(int(time.time())),
        "timestamp": datetime.now().isoformat()
    }), 202

@app.route(\x27/api/backup/restore\x27, methods=[\x27POST\x27])
def backup_restore():
    """Endpoint przywracania z backupu"""
    print(f"[BACKUP] Żądanie przywrócenia o {datetime.now()}")
    return jsonify({
        "status": "success",
        "message": "Rozpoczęto proces przywracania (symulacja).",
        "timestamp": datetime.now().isoformat()
    }), 202

@app.route(\x27/api/backup/configure\x27, methods=[\x27POST\x27])
def backup_configure():
    """Endpoint konfiguracji systemu backupów"""
    print(f"[BACKUP] Żądanie konfiguracji o {datetime.now()}")
    return jsonify({
        "status": "success",
        "message": "Konfiguracja backupów (symulacja).",
        "timestamp": datetime.now().isoformat()
    }), 200'

# Wstawienie nowych endpointów do app.py (przed ostatnimi definicjami @app.route)
# Znajdź linię z "@app.route('\x27/archive\x27')" i wstaw przed nią
TEMP_FILE=$(mktemp)
awk -v new_endpoints="$NEW_ENDPOINTS" '
    /@app.route.*\\x27\\/archive\\x27/ {
        print new_endpoints
        print ""
    }
    {print}
' /opt/ai_firma_dashboard/app.py > "$TEMP_FILE" && mv "$TEMP_FILE" /opt/ai_firma_dashboard/app.py

# Dodanie brakujących importów jeśli potrzebne (datetime, time)
if ! grep -q "from datetime import datetime" /opt/ai_firma_dashboard/app.py; then
    sed -i '\x27/from flask import/ s/$/, datetime/\x27 /opt/ai_firma_dashboard/app.py
    sed -i '\x27/from flask import/ a import time\x27 /opt/ai_firma_dashboard/app.py
fi

echo "Zaktualizowano app.py o nowe endpointy API."
echo "Restartowanie gunicorn..."
sudo systemctl restart gunicorn 2>/dev/null || sudo systemctl restart ai_firma_dashboard 2>/dev/null || echo "Restart ręczny wymagany."
