#!/bin/bash
# OSTATECZNA naprawa uprawnień dla create_gold_image.sh
# Problem: www-data należy do grupy ubuntu, ale grupa ma tylko --x (wykonanie)
# Rozwiązanie: chmod g+r (dodaj czytanie dla grupy)

SCRIPT="/home/ubuntu/skrypty/create_gold_image.sh"

echo "=== OSTATECZNA NAPRAWA UPRAWNIEŃ (grupa) ==="
echo "Obecnie: -rwx--xr-x (715) - grupa 'ubuntu' ma tylko --x (wykonanie)"
echo "www-data należy do grupy 'ubuntu', ale potrzebuje 'r' do czytania przez bash"

echo "Wykonuję: chmod g+r $SCRIPT"
chmod g+r "$SCRIPT"

echo "Nowe uprawnienia:"
ls -la "$SCRIPT"
echo "Numerycznie: $(stat -c "%a" "$SCRIPT")"

echo "Test:"
sudo -u www-data bash -c "if head -1 '$SCRIPT' >/dev/null 2>&1; then echo '✅ www-data MOŻE czytać'; else echo '❌ www-data NIE MOŻE czytać'; fi"

echo "=== GOTOWE ==="
echo "Teraz przetestuj endpoint: curl -X POST http://127.0.0.1:5000/api/gold_image/manage"
