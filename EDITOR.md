#!/bin/bash
# SKRYPT 4 (REALNY): Zamiana panelu Gold Image na komunikat o przeniesieniu
FILE="/opt/ai_firma_dashboard/static/index.html"
BACKUP="${FILE}.before_final_change_$(date +%Y%m%d_%H%M%S)"

echo "1. Tworzenie backupu: $BACKUP"
cp "$FILE" "$BACKUP"

echo "2. Zamiana całego panelu Gold Image (linie 86-102)..."
sed -i '86,102c\
        <!-- PANEL GOLD IMAGE PRZENIESIONY -->\
        <div class="card">\
            <h2>🔧 Zarządzanie Systemem</h2>\
            <div style="padding: 20px; text-align: center;">\
                <p style="color: #666; font-size: 16px;">\
                    Funkcje Gold Image i Backup zostały przeniesione<br>\
                    do dedykowanego panelu zarządzania.\
                </p>\
                <p style="margin-top: 15px;">\
                    <button onclick="location.href=\'/backup_management\'" style="\
                        background: #3498db;\
                        color: white;\
                        padding: 10px 20px;\
                        border-radius: 6px;\
                        border: none;\
                        cursor: pointer;\
                        font-size: 16px;\
                    ">Przejdź do panelu Backup</button>\
                </p>\
            </div>\
        </div>' "$FILE"

echo "3. Restart Gunicorn aby załadować zmiany..."
sudo pkill gunicorn
sleep 2
cd /opt/ai_firma_dashboard
sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon
sleep 2

echo "4. Weryfikacja..."
curl -s http://localhost:5000 | grep -A5 -B5 "Zarządzanie Systemem"

echo "✓ Zmiany zastosowane. Otwórz dashboard w przeglądarce."
