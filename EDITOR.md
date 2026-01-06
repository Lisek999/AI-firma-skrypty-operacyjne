#!/bin/bash
# Skrypt usuwa panel Gold Image z dashboardu
FILE="/opt/ai_firma_dashboard/static/index.html"
BACKUP="${FILE}.backup_$(date +%Y%m%d_%H%M%S)"

echo "Tworzenie backupu: $BACKUP"
cp "$FILE" "$BACKUP"

echo "Usuwanie panelu Gold Image..."
# Usuń sekcję od <!-- NOWY PANEL... do </div> zamykającego panel
sed -i '/<!-- NOWY PANEL: Zarządzanie Gold Image -->/,/^\s*<\/div>\s*$/d' "$FILE"

# Usuń również CSS związany z gold-panel (opcjonalnie, ale zostawmy na razie)
echo "Sprawdzenie czy panel został usunięty..."
grep -n "Gold Image" "$FILE" | head -5

echo "Zrobione. Panel Gold Image usunięty z dashboardu."
