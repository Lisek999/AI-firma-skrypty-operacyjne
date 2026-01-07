#!/bin/bash
# SKRYPT: Zamiana wyłączonego panelu Gold Image na kafelek Backup Management
FILE="/opt/ai_firma_dashboard/static/index.html"
BACKUP="${FILE}.before_backup_tile_$(date +%Y%m%d_%H%M%S)"

echo "1. Tworzenie backupu..."
cp "$FILE" "$BACKUP"

echo "2. Znajdowanie i zamiana całego panelu Gold Image..."

# Usuń stare linie (86-102) i wstaw nowy kafelek
sed -i '86,102c\
        <!-- KAFELEK BACKUP MANAGEMENT (w miejsce starego Gold Image) -->\
        <div class="card">\
            <h2>💾 Zarządzanie Backupami</h2>\
            <div style="padding: 20px; text-align: center;">\
                <p style="color: #666; margin-bottom: 15px;">\
                    Bezpieczne zarządzanie backupami i Gold Image\
                    w dedykowanym, testowym środowisku.\
                </p>\
                <a href="/static/backup_management.html" style="\
                    background: #9b59b6;\
                    color: white;\
                    padding: 12px 24px;\
                    border-radius: 6px;\
                    text-decoration: none;\
                    display: inline-block;\
                    font-weight: bold;\
                    font-size: 16px;\
                ">🔧 Otwórz panel Backup</a>\
                <p style="font-size: 12px; color: #888; margin-top: 10px;">\
                    ⚡ Testy w izolowanym środowisku /tmp/ | 📊 Pełne logowanie\
                </p>\
            </div>\
        </div>' "$FILE"

echo "3. Sprawdzenie zmian..."
echo "   Szukam 'Zarządzanie Backupami':"
grep -n "Zarządzanie Backupami" "$FILE"

echo "4. Restart Gunicorn..."
sudo pkill gunicorn
cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon
sleep 2

echo "✓ Stary panel Gold Image zastąpiony kafelkiem Backup Management."
echo "✓ Sprawdź główny dashboard."
