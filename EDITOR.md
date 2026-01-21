#!/bin/bash
# Naprawa błędnego fi w skrypcie backupu
SCRIPT="/home/ubuntu/skrypty/git_daily_dashboard_backup.sh"

echo "=== Naprawa błędnego fi ==="

echo "[1] Usuwam osierocone fi (linia 39)..."
sed -i '39d' "$SCRIPT"

echo "[2] Sprawdzam poprawiony fragment (linie 35-45):"
sed -n '35,45p' "$SCRIPT"

echo "[3] Test składni:"
bash -n "$SCRIPT" && echo "✅ Składnia OK" || {
    echo "❌ Błąd składni:"
    bash -n "$SCRIPT" 2>&1
}

echo "[4] Test uruchomienia:"
bash "$SCRIPT" 2>&1 | tail -5

echo "[5] Sprawdzam log:"
sleep 2
if [ -f "/var/log/dashboard_backup.log" ]; then
    echo "✅ Log utworzony!"
    tail -3 /var/log/dashboard_backup.log
else
    echo "❌ Log nie utworzony"
fi

echo "=== Gotowe ==="
