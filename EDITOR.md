#!/bin/bash
# ============================================================================
# SKRYPT PATCHUJĄCY app.py - Gold Image Fix
# AUTOR: Wojtek (AI Programista)
# DATA: 2024-12-26
# ============================================================================

echo "=== ROZPOCZĘCIE PATCHOWANIA app.py ==="

BACKUP_FILE="/opt/ai_firma_dashboard/app.py.backup.$(date +%Y%m%d_%H%M%S)"
APP_FILE="/opt/ai_firma_dashboard/app.py"

echo "1. Tworzenie backupu: $BACKUP_FILE"
sudo cp "$APP_FILE" "$BACKUP_FILE"

echo "2. Wyszukiwanie i wymiana ścieżki skryptu"
sudo sed -i 's|"/home/ubuntu/skrypty/create_gold_image.sh"|"/opt/ai_firma_skrypty/create_gold_image.sh"|g' "$APP_FILE"

echo "3. Wyszukiwanie i wymiana katalogu roboczego (cwd)"
sudo sed -i "s|cwd=os.path.dirname(script_path)|cwd='/tmp'|g" "$APP_FILE"

echo "4. Dodawanie logowania dla debugowania (przed wykonaniem skryptu)"
sudo sed -i '/print(f"\[GOLD IMAGE\] Start: {description\[:50\]}")/a\    print(f"[GOLD IMAGE] Ścieżka skryptu: {script_path}")' "$APP_FILE"

echo "5. Sprawdzanie poprawności zmian"
echo "--- Zmiana 1 (ścieżka):"
sudo grep -n "script_path = " "$APP_FILE"
echo "--- Zmiana 2 (cwd):"
sudo grep -n "cwd=" "$APP_FILE"
echo "--- Zmiana 3 (logowanie):"
sudo grep -n "\[GOLD IMAGE\] Ścieżka skryptu" "$APP_FILE"

echo "6. Testowanie składni Python"
if sudo python3 -m py_compile "$APP_FILE"; then
    echo "✓ Składnia poprawna"
else
    echo "✗ Błąd składni! Przywracam backup..."
    sudo cp "$BACKUP_FILE" "$APP_FILE"
    exit 1
fi

echo "7. Restart usługi"
sudo supervisorctl restart dashboard-api

echo "8. Sprawdzanie statusu"
sleep 3
sudo supervisorctl status dashboard-api

echo -e "\n=== PATCH ZAAKTUALIZOWANY ==="
echo "Backup: $BACKUP_FILE"
echo "Gotowe do testowania endpointu /api/gold_image/create"
