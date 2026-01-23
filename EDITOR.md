#!/bin/bash
echo "=== DIAGNOZA SYSTEMU HYBRYDOWEGO - CEO Tomek ==="
echo "Data wykonania: $(date '+%Y-%m-%d %H:%M:%S')"
echo ""

# 1. Status usługi
echo "1. STATUS USŁUGI DASHBOARD:"
sudo supervisorctl status dashboard 2>/dev/null || echo "  Błąd: Nie można sprawdzić statusu"
echo ""

# 2. Struktura katalogów
echo "2. STRUKTURA KATALOGÓW:"
echo "Katalog główny:"
ls -la /var/www/dashboard_v2/ 2>/dev/null | head -10
echo ""
echo "Katalog templates:"
ls -la /var/www/dashboard_v2/templates/ 2>/dev/null | head -10
echo ""

# 3. Plik app.py
echo "3. BACKEND (app.py):"
if [ -f "/var/www/dashboard_v2/app.py" ]; then
    echo "  - Rozmiar: $(wc -l < /var/www/dashboard_v2/app.py) linii"
    echo "  - Endpointy API:"
    grep -c "@app.route" /var/www/dashboard_v2/app.py 2>/dev/null || echo "    0"
    grep -n "@app.route" /var/www/dashboard_v2/app.py 2>/dev/null | head -5
else
    echo "  - BŁĄD: Plik app.py nie istnieje!"
fi
echo ""

# 4. Plik archive.html
echo "4. FRONTEND ARCHIWUM (archive.html):"
if [ -f "/var/www/dashboard_v2/templates/archive.html" ]; then
    echo "  - Rozmiar: $(wc -l < /var/www/dashboard_v2/templates/archive.html) linii"
    echo "  - Zawartość (pierwsze 15 linii):"
    head -15 /var/www/dashboard_v2/templates/archive.html 2>/dev/null | cat -n
else
    echo "  - UWAGA: Plik archive.html nie istnieje - trzeba będzie utworzyć"
fi
echo ""

# 5. Backup directory
echo "5. SYSTEM BACKUPÓW:"
if [ -d "/backups" ]; then
    echo "  - Katalog /backups istnieje"
    ls -la /backups/ | head -10
else
    echo "  - UWAGA: Katalog /backups nie istnieje"
fi
echo ""

# 6. Cron jobs
echo "6. ZADANIA CRON:"
crontab -l 2>/dev/null | grep -A5 -B5 "backup\|dashboard" || echo "  - Brak wpisów związanych z backupem/dashboardem"
echo ""

# 7. Python dependencies
echo "7. ZALEŻNOŚCI PYTHON:"
if python3 -c "import flask" 2>/dev/null; then
    echo "  - Flask: INSTALOWANY"
else
    echo "  - Flask: BRAK"
fi
echo ""

echo "=== KONIEC DIAGNOZY ==="
echo "Gotowy do analizy (Etap 2)."
