#!/bin/bash
# ============================================================================
# SKRYPT: naprawa_uprawnien_gold_image.sh
# CEL: Naprawa uprawnień - dodanie czytania dla www-data
# DATA: $(date)
# ============================================================================

SCRIPT_PATH="/home/ubuntu/skrypty/create_gold_image.sh"

echo "=== Rozpoczynam naprawę uprawnień skryptu ==="

# 1. Diagnoza obecnych uprawnień
echo "[1] Obecne uprawnienia:"
ls -la "$SCRIPT_PATH"

CURRENT_PERMS=$(stat -c "%a" "$SCRIPT_PATH")
echo "    Uprawnienia numerycznie: $CURRENT_PERMS"

# 2. Analiza problemu
echo "[2] Analiza:"
echo "    Obecnie: -rwx--x--x (właściciel: rwx, grupa: --x, inni: --x)"
echo "    Problem: www-data (innym) brakuje 'r' (czytania) do interpretacji przez bash"
echo "    Rozwiązanie: chmod o+r (dodaj czytanie dla innych)"

# 3. Wykonanie naprawy
echo "[3] Wykonanie naprawy: chmod o+r $SCRIPT_PATH"
chmod o+r "$SCRIPT_PATH"

# 4. Weryfikacja zmian
echo "[4] Nowe uprawnienia:"
ls -la "$SCRIPT_PATH"

NEW_PERMS=$(stat -c "%a" "$SCRIPT_PATH")
echo "    Nowe uprawnienia numerycznie: $NEW_PERMS"

# 5. Test uprawnień jako www-data (symulacja)
echo "[5] Test uprawnień (symulacja):"
sudo -u www-data test -r "$SCRIPT_PATH" && echo "    ✅ www-data MOŻE czytać plik" || echo "    ❌ www-data NIE MOŻE czytać pliku"
sudo -u www-data test -x "$SCRIPT_PATH" && echo "    ✅ www-data MOŻE wykonać plik" || echo "    ❌ www-data NIE MOŻE wykonać pliku"

echo "[6] Gotowe! Uprawnienia naprawione."
echo ""
echo "=== Podsumowanie ==="
echo "Zmieniono: -rwx--x--x -> -rwx--xr-x"
echo "Dodano: czytanie (r) dla innych (w tym www-data)"
echo ""
echo "Aby przetestować:"
echo "curl -X POST http://127.0.0.1:5000/api/gold_image/manage"
