#!/bin/bash
# Naprawa crontab - zmiana z EDITOR.md na dedykowane skrypty backupu
echo "=== Naprawa crontab - właściwe skrypty backupu ==="

echo "[1] Obecny crontab (problematyczny):"
crontab -l | head -10

echo "[2] Tworzę poprawiony crontab..."

# Tworzenie tymczasowego pliku z poprawionym crontab
TEMP_CRON=$(mktemp)

cat > "$TEMP_CRON" << 'EOF'
# === AUTOMATYCZNE BACKUPY AI FIRMA ===
# Backup dzienny - dashboard (3:00)
0 3 * * * /usr/bin/bash /home/ubuntu/skrypty/git_daily_dashboard_backup.sh >> /var/log/dashboard_backup.log 2>&1
# Backup dzienny - skrypty (3:15)
15 3 * * * /usr/bin/bash /home/ubuntu/skrypty/git_daily_skrypty_backup.sh >> /var/log/skrypty_backup.log 2>&1
# Backup tygodniowy - pełny (niedziela 4:00)
0 4 * * 0 /usr/bin/bash /home/ubuntu/skrypty/git_weekly_full_backup.sh >> /var/log/weekly_backup.log 2>&1
# Secure Vault - backup tajemnic (3:30 codziennie)
30 3 * * * /home/ubuntu/ai_firma_backups/secure_vault/backup_secrets.sh >> /home/ubuntu/ai_firma_backups/secure_vault/backup_secrets_cron.log 2>&1
EOF

echo "[3] Nowy crontab:"
cat "$TEMP_CRON"

echo "[4] Zastępuję crontab..."
crontab "$TEMP_CRON"

echo "[5] Weryfikacja:"
crontab -l

echo "[6] Test ręcznego uruchomienia backupu dashboardu (tworzy log):"
echo "   Wykonuję: bash /home/ubuntu/skrypty/git_daily_dashboard_backup.sh"
bash /home/ubuntu/skrypty/git_daily_dashboard_backup.sh 2>&1 | tail -5

echo "[7] Sprawdzam czy log powstał:"
sleep 2
if [ -f "/var/log/dashboard_backup.log" ]; then
    echo "   ✅ Log utworzony!"
    ls -la /var/log/dashboard_backup.log
    echo "   Ostatnie 3 linie:"
    tail -3 /var/log/dashboard_backup.log
else
    echo "   ❌ Log NIE utworzony"
fi

# Czyszczenie
rm -f "$TEMP_CRON"

echo "[8] Endpoint backup_status() będzie teraz pokazywał daty z logów!"
echo "    Restart gunicorn:"
echo "    sudo pkill -f 'gunicorn.*app:app'"
echo "    cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 0.0.0.0:5000 app:app --daemon"

echo "=== Gotowe ==="
