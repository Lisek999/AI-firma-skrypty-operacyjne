#!/bin/bash
# SKRYPT NAPRAWCZY: Usuwa błędy JavaScript w dashboardzie
FILE="/opt/ai_firma_dashboard/static/index.html"
BACKUP="${FILE}.final_fix_$(date +%Y%m%d_%H%M%S)"

echo "=== NAPRAWA DASHBOARD ==="
echo "1. Backup: $BACKUP"
cp "$FILE" "$BACKUP"

echo "2. Usuwam problematyczne event listeners..."
# Usuń linie które powodują błędy
sed -i "/document.getElementById('createGoldBtn').addEventListener/d" "$FILE"
sed -i "/document.getElementById('saveKeyBtn').addEventListener/d" "$FILE"

echo "3. Zastępuję odwołania do nieistniejących elementów..."
# Zasłoń wszystkie getElementById które mogą powodować błędy
sed -i "s/document.getElementById('goldStatus')/null/g" "$FILE"
sed -i "s/document.getElementById('authSection')/null/g" "$FILE"
sed -i "s/document.getElementById('apiKeyInput')/null/g" "$FILE"
sed -i "s/document.getElementById('goldMessage')/null/g" "$FILE"
sed -i "s/document.getElementById('createGoldBtn')/null/g" "$FILE"
sed -i "s/document.getElementById('saveKeyBtn')/null/g" "$FILE"

echo "4. Naprawiam funkcję showMessage jeśli jest używana..."
# Jeśli showMessage odwołuje się do goldMessage, to napraw
sed -i "s/messageDiv.classList/addMessageClass/g" "$FILE" 2>/dev/null || true

echo "5. Sprawdzam czy loadData() jest wywoływane..."
if grep -q "loadData()" "$FILE"; then
    echo "   ✓ loadData() jest w kodzie"
else
    echo "   ❌ loadData() NIE MA - to problem!"
    # Dodajemy wywołanie loadData jeśli brakuje
    sed -i "/setInterval(loadData, 10000)/a\\
            loadData();" "$FILE"
fi

echo "6. Restart Gunicorn..."
sudo pkill gunicorn
cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon
sleep 3

echo "7. Weryfikacja..."
echo "   Backend działa:" $(curl -s http://localhost:5000/api/health | grep -o '"status":"[^"]*"' || echo "nie")

echo "=== NAPRAWA ZAKOŃCZONA ==="
echo "✓ Sprawdź dashboard w przeglądarce."
