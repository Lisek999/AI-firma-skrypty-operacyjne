#!/bin/bash
# SKRYPT: Naprawa funkcji initGoldPanel - usuwa błędy JavaScript
FILE="/opt/ai_firma_dashboard/static/index.html"
BACKUP="${FILE}.fix_js_$(date +%Y%m%d_%H%M%S)"

echo "1. Tworzenie backupu..."
cp "$FILE" "$BACKUP"

echo "2. Szukam funkcji initGoldPanel..."
START_LINE=$(grep -n "function initGoldPanel()" "$FILE" | cut -d: -f1)

if [ -z "$START_LINE" ]; then
    echo "❌ Nie znaleziono funkcji initGoldPanel()"
    exit 1
fi

echo "3. Funkcja znaleziona w linii $START_LINE"

echo "4. Zastępuję starą funkcję nową..."
# Usuwamy 20 linii od START_LINE (wystarczająco na całą funkcję)
sed -i "${START_LINE},+20d" "$FILE"

# Wstawiamy nową, bezpieczną funkcję
sed -i "${START_LINE}i\\
        function initGoldPanel() {\\
            // Funkcja dezaktywowana - panel Gold Image przeniesiony\\
            // do /static/backup_management.html\\
            console.log('Gold Image: panel disabled, use backup management page');\\
        }" "$FILE"

echo "5. Usuwam wywołanie initGoldPanel() z DOMContentLoaded..."
sed -i "s/initGoldPanel();//g" "$FILE"

echo "6. Sprawdzam czy są jeszcze odwołania do nieistniejących elementów..."
if grep -q "getElementById.*goldStatus\|getElementById.*authSection\|getElementById.*createGoldBtn" "$FILE"; then
    echo "   ⚠️  Są jeszcze odwołania - mogą powodować błędy"
    grep -n "getElementById.*goldStatus\|getElementById.*authSection\|getElementById.*createGoldBtn" "$FILE"
else
    echo "   ✓ Brak niebezpiecznych odwołań"
fi

echo "7. Restart Gunicorn..."
sudo pkill gunicorn
cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon
sleep 3

echo "✓ Naprawa JavaScript zakończona. Sprawdź dashboard."
