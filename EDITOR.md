#!/bin/bash
# SKRYPT 4: ZAMIANA panelu Gold Image na komunikat o przeniesieniu
FILE="/opt/ai_firma_dashboard/static/index.html"
BACKUP="${FILE}.before_replace_$(date +%Y%m%d_%H%M%S)"

echo "Tworzenie backupu..."
cp "$FILE" "$BACKUP"

echo "Zamiana panelu Gold Image na komunikat..."
# Zamień CAŁY panel (linie 86-102) na nową treść
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
                    <a href="/backup_management" style="\
                        background: #3498db;\
                        color: white;\
                        padding: 10px 20px;\
                        border-radius: 6px;\
                        text-decoration: none;\
                        display: inline-block;\
                    ">Przejdź do panelu Backup</a>\
                </p>\
            </div>\
        </div>' "$FILE"

echo "Zrobione. Panel został ZASTĄPIONY, nie usunięty."
