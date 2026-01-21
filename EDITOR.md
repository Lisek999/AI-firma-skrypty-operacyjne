#!/bin/bash
# ============================================================================
# SKRYPT: naprawa_duplikatu_cd.sh
# CEL: Usunięcie zduplikowanej sekcji cd z create_gold_image.sh
# DATA: $(date)
# ============================================================================

SCRIPT_PATH="/home/ubuntu/skrypty/create_gold_image.sh"
BACKUP_PATH="/home/ubuntu/skrypty/create_gold_image.sh.backup_dup_fix_$(date +%Y%m%d_%H%M%S)"

echo "=== Rozpoczynam naprawę duplikatu cd w skrypcie ==="

# 1. Tworzenie backupu
echo "[1] Tworzenie backupu: $BACKUP_PATH"
cp "$SCRIPT_PATH" "$BACKUP_PATH" || {
    echo "ERROR: Nie można utworzyć backupu!"
    exit 1
}

# 2. Pokazanie problematycznego fragmentu
echo "[2] Obecny problematyczny fragment (linie 285-310):"
sed -n '285,310p' "$SCRIPT_PATH"

# 3. Znalezienie dokładnych linii do usunięcia
echo "[3] Szukam zduplikowanych linii cd..."
# Szukamy wzorca: cd "$REPO_ROOT" z następną identyczną sekcją
START_LINE=$(grep -n '# Przejdź do katalogu repozytorium' "$SCRIPT_PATH" | tail -1 | cut -d: -f1)

if [ -z "$START_LINE" ] || [ "$START_LINE" -lt 290 ]; then
    echo "ERROR: Nie znaleziono zduplikowanej sekcji!"
    exit 1
fi

echo "    Zduplikowana sekcja zaczyna się w linii: $START_LINE"

# 4. Usunięcie zduplikowanych linii (293-298 wg wcześniejszego podglądu)
# Zakładając, że druga sekcja to linie START_LINE do START_LINE+5
END_LINE=$((START_LINE + 5))

echo "[4] Usuwanie zduplikowanych linii $START_LINE-$END_LINE..."
sed -i "${START_LINE},${END_LINE}d" "$SCRIPT_PATH"

# 5. Weryfikacja
echo "[5] Po naprawie (linie 285-305):"
sed -n '285,305p' "$SCRIPT_PATH"

# 6. Test składni
echo "[6] Test parsowania składni:"
if bash -n "$SCRIPT_PATH"; then
    echo "    ✅ Składnia poprawna"
else
    echo "    ❌ Błąd składni! Przywracam backup..."
    cp "$BACKUP_PATH" "$SCRIPT_PATH"
    exit 1
fi

echo "[7] Gotowe! Duplikat usunięty."
echo ""
echo "=== Następny krok ==="
echo "Teraz należy naprawić uprawnienia: chmod o+r $SCRIPT_PATH"
echo "Lub uruchomić skrypt naprawy uprawnień z EDITOR.md"
