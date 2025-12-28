#!/bin/bash
# SKRYPT: configure_backup_cron.sh
# CEL: Konfiguracja automatycznych backupów przez cron
# DATA: 2024-12-29

echo "=== KONFIGURACJA AUTOMATYCZNYCH BACKUPÓW (CRON) ==="
echo ""

# 1. DIAGNOZA
echo "1. DIAGNOZA aktualnego cron..."
crontab -l 2>/dev/null || echo "   Brak wpisów cron"

# 2. ANALIZA
echo ""
echo "2. ANALIZA planowanych zadań..."
echo "   a) Backup dzienny: 0 3 * * *"
echo "      - Dashboard: git_daily_dashboard_backup.sh"
echo "      - Skrypty: git_daily_skrypty_backup.sh"
echo "   b) Backup tygodniowy: 0 4 * * 0 (niedziela 4:00)"
echo "      - Pełne archiwum: git_weekly_full_backup.sh"
echo "   c) Logowanie: /var/log/backup_status.json"

# 3. ZMIANA
echo ""
echo "3. ZMIANA - konfiguracja cron..."
read -p "Czy dodać zadania cron? (TAK/n): " -n 1 -r
echo ""
if [[ ! $REPLY =~ ^[Tt]$ ]] && [[ ! $REPLY =~ ^[Yy]$ ]]; then
    echo "❌ Anulowano"
    exit 0
fi

# Stwórz katalog na skrypty jeśli nie istnieje
SCRIPT_DIR="/home/ubuntu/backup_scripts"
mkdir -p "$SCRIPT_DIR"

# Pobierz skrypty z GitHub
echo "   a) Pobieranie skryptów backupu..."
curl -s "https://raw.githubusercontent.com/Lisek999/AI-firma-skrypty-operacyjne/main/EDITOR.md" -o "$SCRIPT_DIR/git_daily_dashboard_backup.sh"
curl -s "https://raw.githubusercontent.com/Lisek999/AI-firma-skrypty-operacyjne/main/EDITOR.md" -o "$SCRIPT_DIR/git_daily_skrypty_backup.sh"
curl -s "https://raw.githubusercontent.com/Lisek999/AI-firma-skrypty-operacyjne/main/EDITOR.md" -o "$SCRIPT_DIR/git_weekly_full_backup.sh"

# Ustaw uprawnienia
chmod +x "$SCRIPT_DIR"/*.sh

# Dodaj do crontab
echo "   b) Dodawanie zadań do crontab..."
(crontab -l 2>/dev/null; echo "# === AUTOMATYCZNE BACKUPY AI FIRMA ===") | crontab -
(crontab -l 2>/dev/null; echo "# Backup dzienny - dashboard (3:00)") | crontab -
(crontab -l 2>/dev/null; echo "0 3 * * * $SCRIPT_DIR/git_daily_dashboard_backup.sh >> /var/log/dashboard_backup.log 2>&1") | crontab -
(crontab -l 2>/dev/null; echo "# Backup dzienny - skrypty (3:15)") | crontab -
(crontab -l 2>/dev/null; echo "15 3 * * * $SCRIPT_DIR/git_daily_skrypty_backup.sh >> /var/log/skrypty_backup.log 2>&1") | crontab -
(crontab -l 2>/dev/null; echo "# Backup tygodniowy - pełny (niedziela 4:00)") | crontab -
(crontab -l 2>/dev/null; echo "0 4 * * 0 $SCRIPT_DIR/git_weekly_full_backup.sh >> /var/log/weekly_backup.log 2>&1") | crontab -

# Stwórz plik statusu
echo "   c) Tworzenie pliku statusu..."
echo '{"last_update": "'$(date -Iseconds)'", "status": "cron_configured"}' > /var/log/backup_status.json

# 4. WERYFIKACJA
echo ""
echo "4. WERYFIKACJA - konfiguracja cron..."
crontab -l
echo ""
echo "✅ Skrypy backupu: $SCRIPT_DIR/"
ls -la "$SCRIPT_DIR/"
echo ""
echo "✅ Plik statusu: /var/log/backup_status.json"
cat /var/log/backup_status.json

echo ""
echo "=== KONFIGURACJA CRON ZAKOŃCZONA ==="
echo "Backupy będą uruchamiane automatycznie:"
echo "- Dzienne: 3:00 (dashboard), 3:15 (skrypty)"
echo "- Tygodniowe: Niedziela 4:00 (pełne archiwum)"
echo ""
echo "Możesz ręcznie przetestować:"
echo "  $SCRIPT_DIR/git_daily_dashboard_backup.sh"
echo "  $SCRIPT_DIR/git_weekly_full_backup.sh"
