#!/bin/bash
# SKRYPT 3: BEZPIECZNE UKRYCIE panelu Gold Image przez komentarz
FILE="/opt/ai_firma_dashboard/static/index.html"
BACKUP="${FILE}.before_comment_$(date +%Y%m%d_%H%M%S)"

echo "1. Tworzenie backupu: $BACKUP"
cp "$FILE" "$BACKUP"

echo "2. Dodawanie komentarza OTWIERAJĄCEGO przed panelem..."
# Dodaj komentarz otwierający PRZED linią 86
sed -i '86i\        <!-- PANEL GOLD IMAGE WYŁĄCZONY - PRZENIESIONY DO OSOBNEJ ZAKŁADKI -->' "$FILE"
sed -i '87i\        <!--' "$FILE"

echo "3. Dodawanie komentarza ZAMYKAJĄCEGO po panelu..."
# Znajdź linię z </div> zamykającym panel (około linia 102)
# Dodaj komentarz zamykający PO tej linii
sed -i '103i\        -->' "$FILE"

echo "4. Weryfikacja..."
echo "   Podgląd linii 84-108:"
sed -n '84,108p' "$FILE"

echo "✓ Panel Gold Image został wyłączony (zakomentowany)."
echo "✓ HTML pozostaje nienaruszony strukturalnie."
echo "✓ Możesz łatwo przywrócić usuwając komentarze."
