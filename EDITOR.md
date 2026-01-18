#!/bin/bash
# SKRYPT 1 BEZPIECZNY: Dodanie endpointów API do istniejącego app.py (append)
# Autor: Wojtek (AI)
# Cel: Bezpieczne dopisanie 5 nowych endpointów bez modyfikacji istniejącego kodu.

cd /opt/ai_firma_dashboard || exit 1

# 1. Backup
BACKUP_DIR="backups"
mkdir -p "$BACKUP_DIR"
BACKUP_FILE="$BACKUP_DIR/app.py.backup_before_endpoints_$(date +%Y%m%d_%H%M%S)"
cp app.py "$BACKUP_FILE"
echo "Utworzono backup: $BACKUP_FILE"

# 2. Sprawdzenie, czy potrzebne importy już istnieją
if ! grep -q "from datetime import datetime" app.py; then
    echo "Dodaję brakujący import: datetime"
    sed -i "/from flask import/ s/from flask import/from flask import/" app.py  # Placeholder, rzeczywista komenda poniżej
    # Dokładna komenda dodająca 'datetime' do importów
    sed -i "s/from flask import Flask, jsonify, render_template/from flask import Flask, jsonify, render_template\nfrom datetime import datetime/" app.py
fi

if ! grep -q "^import time$" app.py; then
    echo "Dodaję brakujący import: time"
    # Dodaj 'import time' po innych importach
    sed -i "/from datetime import datetime/a import time" app.py
fi

# 3. Dopisanie nowych endpointów na końcu pliku (przed ewentualnym if __name__)
cat >> app.py << 'EOF'

# ========== NOWE ENDPOINTY API - GOLD IMAGE & BACKUP (dodane $(date)) ==========
@app.route('/api/gold_image/manage', methods=['POST'])
def gold_image_manage():
    """Endpoint do zarządzania Gold Image (testowy)"""
    print(f"[GOLD_IMAGE] Żądanie zarządzania o {datetime.now()}")
    return jsonify({
        "status": "success",
        "message": "Endpoint Gold Image /manage jest aktywny (tryb testowy).",
        "timestamp": datetime.now().isoformat()
    }), 200

@app.route('/api/backup/status', methods=['GET'])
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

@app.route('/api/backup/run_manual', methods=['POST'])
def backup_run_manual():
    """Endpoint uruchamiania ręcznego backupu"""
    print(f"[BACKUP] Żądanie ręcznego backupu o {datetime.now()}")
    return jsonify({
        "status": "success",
        "message": "Rozpoczęto ręczny backup (symulacja).",
        "job_id": "backup_manual_" + str(int(time.time())),
        "timestamp": datetime.now().isoformat()
    }), 202

@app.route('/api/backup/restore', methods=['POST'])
def backup_restore():
    """Endpoint przywracania z backupu"""
    print(f"[BACKUP] Żądanie przywrócenia o {datetime.now()}")
    return jsonify({
        "status": "success",
        "message": "Rozpoczęto proces przywracania (symulacja).",
        "timestamp": datetime.now().isoformat()
    }), 202

@app.route('/api/backup/configure', methods=['POST'])
def backup_configure():
    """Endpoint konfiguracji systemu backupów"""
    print(f"[BACKUP] Żądanie konfiguracji o {datetime.now()}")
    return jsonify({
        "status": "success",
        "message": "Konfiguracja backupów (symulacja).",
        "timestamp": datetime.now().isoformat()
    }), 200
EOF

echo "Dopisano nowe endpointy do app.py."

# 4. Restart usługi (delikatny reload)
echo "Przeładowanie aplikacji (HUP)..."
pkill -HUP gunicorn 2>/dev/null && echo "Przeładowano gunicorn." || {
    echo "Nie udało się wysłać HUP. Możliwy ręczny restart."
    # sudo systemctl restart gunicorn 2>/dev/null || sudo systemctl restart ai_firma_dashboard 2>/dev/null
}
