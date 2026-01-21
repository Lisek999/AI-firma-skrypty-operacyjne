#!/bin/bash
# Naprawa uprawnień dla create_gold_image.sh
SCRIPT="/home/ubuntu/skrypty/create_gold_image.sh"

echo "=== Naprawa uprawnień ==="
echo "Obecne uprawnienia:"
ls -la "$SCRIPT"

echo "Dodaję prawo czytania dla innych (www-data)..."
chmod o+r "$SCRIPT"

echo "Nowe uprawnienia:"
ls -la "$SCRIPT"

# Test
sudo -u www-data test -r "$SCRIPT" && echo "✅ www-data MOŻE czytać" || echo "❌ www-data NIE MOŻE czytać"
sudo -u www-data test -x "$SCRIPT" && echo "✅ www-data MOŻE wykonać" || echo "❌ www-data NIE MOŻE wykonać"

echo "=== Gotowe ==="
