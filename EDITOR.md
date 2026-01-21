#!/bin/bash
# ============================================================================
# SKRYPT: naprawa_uprawnien_final.sh
# CEL: Dodanie prawa czytania dla www-data do create_gold_image.sh
# DATA: 2026-01-21
# ============================================================================

SCRIPT="/home/ubuntu/skrypty/create_gold_image.sh"

echo "=== FINALNA NAPRAWA UPRAWNIEŃ ==="

echo "[1] Obecny stan:"
ls -la "$SCRIPT"
echo "   Numerycznie: $(stat -c "%a" "$SCRIPT")"

echo "[2] Problem:"
echo "   -rwx--x--x = właściciel: rwx, grupa: --x, inni: --x"
echo "   www-data (inni) potrzebuje 'r' (czytania) do interpretacji przez bash"

echo "[3] Wykonanie: chmod o+r $SCRIPT"
chmod o+r "$SCRIPT"

echo "[4] Nowy stan:"
ls -la "$SCRIPT"
echo "   Numerycznie: $(stat -c "%a" "$SCRIPT")"
echo "   -rwx--xr-x = właściciel: rwx, grupa: --x, inni: r-x (dodano czytanie)"

echo "[5] Test uprawnień dla www-data:"
if sudo -u www-data test -r "$SCRIPT"; then
    echo "   ✅ www-data MOŻE czytać plik"
else
    echo "   ❌ www-data NIE MOŻE czytać pliku"
fi

if sudo -u www-data test -x "$SCRIPT"; then
    echo "   ✅ www-data MOŻE wykonać plik"
else
    echo "   ❌ www-data NIE MOŻE wykonać pliku"
fi

echo "[6] Test endpointu (po restarcie gunicorn):"
echo "   sudo pkill -f 'gunicorn.*app:app'"
echo "   cd /opt/ai_firma_dashboard && sudo -u www-data /opt/ai_firma_dashboard/venv/bin/gunicorn --workers 2 --bind 127.0.0.1:5000 app:app --daemon"
echo "   sleep 3 && curl -X POST http://127.0.0.1:5000/api/gold_image/manage"

echo "=== GOTOWE ==="
