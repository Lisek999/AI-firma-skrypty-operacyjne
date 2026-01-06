#!/bin/bash
# SKRYPT 2: Bezpieczne usunięcie panelu Gold Image (tylko linie 86-102)
FILE="/opt/ai_firma_dashboard/static/index.html"
BACKUP="${FILE}.pre_gold_removal_$(date +%Y%m%d_%H%M%S)"

echo "1. Tworzenie backupu: $BACKUP"
cp "$FILE" "$BACKUP"

echo "2. Usuwanie LINII 86-102 (panel Gold Image)..."
# Tworzymy tymczasowy plik bez tych linii
sed '86,102d' "$FILE" > "${FILE}.tmp"

echo "3. Zastępowanie oryginalnego pliku..."
mv "${FILE}.tmp" "$FILE"

echo "4. Weryfikacja - sprawdzanie czy 'Gold Image' nadal istnieje..."
if grep -q "Gold Image" "$FILE"; then
    echo "   UWAGA: Znaleziono jeszcze 'Gold Image' w pliku!"
    grep -n "Gold Image" "$FILE"
else
    echo "   ✓ Panel Gold Image został usunięty."
fi

echo "5. Liczba linii w pliku:"
wc -l "$FILE"

echo "6. Podgląd obszaru po usunięciu (linie 80-110):"
sed -n '80,110p' "$FILE"
