#!/bin/bash
# Naprawa skryptu backupu - usunięcie interakcji dla cron
BACKUP_SCRIPT="/home/ubuntu/skrypty/git_daily_dashboard_backup.sh"

echo "=== Naprawa skryptu backupu dla cron ==="

echo "[1] Obecny problematyczny fragment (linie 35-45):"
sed -n '35,45p' "$BACKUP_SCRIPT"

echo "[2] Tworzę kopię bezpieczeństwa..."
cp "$BACKUP_SCRIPT" "$BACKUP_SCRIPT.backup_$(date +%Y%m%d_%H%M%S)"

echo "[3] Usuwam interaktywny read i zastępuję automatycznym TAK..."
# Usuń linie 38-43 i zastąp prostym kontynuowaniem
sed -i '38,43d' "$BACKUP_SCRIPT"
sed -i '37a\echo "✅  Kontynuowanie backupu (tryb automatyczny dla cron)..."' "$BACKUP_SCRIPT"

echo "[4] Nowy fragment (linie 35-40):"
sed -n '35,40p' "$BACKUP_SCRIPT"

echo "[5] Test uruchomienia (powinno działać bez pytania):"
echo "   Uruchamiam: bash $BACKUP_SCRIPT"
bash "$BACKUP_SCRIPT" 2>&1 | tail -10

echo "[6] Sprawdzam czy log powstał:"
sleep 3
if [ -f "/var/log/dashboard_backup.log" ]; then
    echo "   ✅ Log utworzony!"
    ls -la /var/log/dashboard_backup.log
    echo "   Ostatnie 3 linie:"
    tail -3 /var/log/dashboard_backup.log
    
    echo "[7] Test endpointu (będzie pokazywał daty z logów):"
    echo "   curl -s http://57.128.247.215:5000/api/backup/status | python3 -c \"import sys, json; print(json.load(sys.stdin)['message'])\""
else
    echo "   ❌ Log NIE utworzony - skrypt może mieć inne problemy"
    echo "   Sprawdzam błędy z ostatniego uruchomienia..."
    bash "$BACKUP_SCRIPT" 2>&1 | grep -i "error\|fail\|błąd" | head -5
fi

echo "[8] To samo dla innych skryptów backupu:"
echo "   /home/ubuntu/skrypty/git_daily_skrypty_backup.sh"
echo "   /home/ubuntu/skrypty/git_weekly_full_backup.sh"
echo "   (wymagają tej samej naprawy)"

echo "=== Gotowe ==="
