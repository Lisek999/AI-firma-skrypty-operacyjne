#!/bin/bash
# ============================================================================
# SKRYPT: naprawa_create_gold_image.sh
# CEL: Minimalna naprawa - dodanie cd do REPO_ROOT w skrypcie Gold Image
# DATA: $(date)
# ============================================================================

SCRIPT_PATH="/home/ubuntu/skrypty/create_gold_image.sh"
BACKUP_PATH="/home/ubuntu/skrypty/create_gold_image.sh.backup_$(date +%Y%m%d_%H%M%S)"

echo "=== Rozpoczynam naprawę skryptu create_gold_image.sh ==="

# 1. Tworzenie backupu
echo "[1] Tworzenie backupu: $BACKUP_PATH"
cp "$SCRIPT_PATH" "$BACKUP_PATH" || {
    echo "ERROR: Nie można utworzyć backupu!"
    exit 1
}

# 2. Diagnoza obecnego stanu - gdzie jest wywołanie validate_git_status?
echo "[2] Analiza obecnej struktury skryptu..."
echo "    Miejsce wywołania validate_git_status:"
grep -n "validate_git_status" "$SCRIPT_PATH"

echo "    Sekcja główna (linie po funkcjach pomocniczych):"
grep -n "log_info.*Rozpoczynanie" "$SCRIPT_PATH"

# 3. Znajdź linię rozpoczęcia głównej sekcji
START_LINE=$(grep -n "log_info.*Rozpoczynanie tworzenia Gold Image" "$SCRIPT_PATH" | cut -d: -f1)

if [ -z "$START_LINE" ]; then
    echo "ERROR: Nie znaleziono linii rozpoczęcia głównej sekcji!"
    exit 1
fi

echo "[3] Główna sekcja zaczyna się w linii: $START_LINE"

# 4. Wstawienie 'cd "$REPO_ROOT"' bezpośrednio po rozpoczęciu głównej sekcji
echo "[4] Wstawianie 'cd \"\$REPO_ROOT\"' w linii $((START_LINE + 1))..."

# Tworzenie tymczasowego pliku z poprawką
TEMP_FILE=$(mktemp)

# Kopiowanie skryptu z dodaną komendą cd
awk -v line="$START_LINE" '
{
    print $0
    if (NR == line) {
        print "    # Przejdź do katalogu repozytorium"
        print "    cd \"$REPO_ROOT\" || {"
        print "        log_error \"Nie można przejść do katalogu: $REPO_ROOT\""
        print "        exit 1"
        print "    }"
        print "    log_info \"Pracuję w katalogu: $(pwd)\""
    }
}' "$SCRIPT_PATH" > "$TEMP_FILE"

# 5. Zastąpienie oryginalnego pliku
mv "$TEMP_FILE" "$SCRIPT_PATH"
chmod +x "$SCRIPT_PATH"

# 6. Weryfikacja zmian
echo "[5] Weryfikacja zmian:"
echo "    Zmienione linie wokół $START_LINE:"
sed -n "$((START_LINE-2)),$((START_LINE+10))p" "$SCRIPT_PATH"

# 7. Test parsowania (bez wykonania)
echo "[6] Test parsowania składni:"
if bash -n "$SCRIPT_PATH"; then
    echo "    ✅ Składnia poprawna"
else
    echo "    ❌ Błąd składni!"
    echo "    Przywracanie backupu..."
    cp "$BACKUP_PATH" "$SCRIPT_PATH"
    exit 1
fi

echo "[7] Gotowe! Skrypt został naprawiony."
echo ""
echo "=== Podsumowanie ==="
echo "1. Backup: $BACKUP_PATH"
echo "2. Dodano: 'cd \"\$REPO_ROOT\"' po linii $START_LINE"
echo "3. Test składni: ✅"
echo ""
echo "Aby przetestować:"
echo "curl -X POST http://127.0.0.1:5000/api/gold_image/manage"
