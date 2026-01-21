#!/bin/bash
# Test ręcznego backupu i weryfikacja logów
echo "=== Test ręcznego backupu dashboardu ==="

echo "[1] Sprawdzam czy skrypt backupu istnieje:"
BACKUP_SCRIPT="/home/ubuntu/skrypty/git_daily_dashboard_backup.sh"
if [ -f "$BACKUP_SCRIPT" ]; then
    echo "   ✅ $BACKUP_SCRIPT istnieje"
    ls -la "$BACKUP_SCRIPT"
else
    echo "   ❌ Skrypt nie istnieje!"
    exit 1
fi

echo "[2] Sprawdzam obecny stan logów:"
ls -la /var/log/*backup.log 2>/dev/null || echo "   Brak logów backup"

echo "[3] Uruchamiam ręczny backup jako www-data (symulacja cron):"
echo "   Wykonuję: sudo -u www-data bash $BACKUP_SCRIPT"
sudo -u www-data bash "$BACKUP_SCRIPT"

echo "[4] Sprawdzam czy powstał log:"
sleep 2
if [ -f "/var/log/dashboard_backup.log" ]; then
    echo "   ✅ Log został utworzony!"
    ls -la /var/log/dashboard_backup.log
    echo "   Ostatnie 5 linii logu:"
    tail -5 /var/log/dashboard_backup.log
else
    echo "   ❌ Log NIE został utworzony"
    echo "   Sprawdzam czy cron tworzy log w inny sposób..."
    
    # Sprawdźmy czy cron zapisuje gdzie indziej
    echo "   Test ręczny bez sudo:"
    bash "$BACKUP_SCRIPT" 2>&1 | tail -3
    
    echo "   Sprawdzam katalog /home/ubuntu/logs:"
    ls -la /home/ubuntu/logs/ 2>/dev/null || echo "   Brak katalogu logs"
fi

echo "[5] Test endpointu backup_status():"
echo "   curl -s http://127.0.0.1:5000/api/backup/status | python3 -m json.tool | grep -A5 daily_dashboard"

echo "[6] Jeśli log powstał - restart gunicorn:"
echo "   sudo pkill -f 'gunicorn.*app:app'"
echo "   cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon"

echo "=== Gotowe ==="

