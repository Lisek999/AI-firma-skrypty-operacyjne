#!/bin/bash
# ============================================================================
# DEPLOY_GOLD.SH - System przywracania konfiguracji z Gold Image
# Wersja: 1.1 (poprawiona - dodano check_sudo)
# Autor: AI Firma - System Operacyjny
# Data: 2024-12-25
# ============================================================================
# ZASADY:
# 1. ZAWSZE tworzy pre-backup przed deployem
# 2. TYLKO 3 pliki konfiguracyjne (początkowo)
# 3. Wymaga sudo
# ============================================================================

set -e  # Zatrzymaj przy pierwszym błędzie

echo "=========================================="
echo "    GOLD IMAGE DEPLOY - SYSTEM RESTORE"
echo "=========================================="
echo "Data: $(date)"
echo ""

# ================= KONFIGURACJA =================
readonly GOLD_DIR="/home/ubuntu/ai_firma_backups/gold_image"
readonly TIMESTAMP=$(date +%Y%m%d_%H%M%S)
readonly BACKUP_BASE="/home/ubuntu/ai_firma_backups/tmp"
readonly BACKUP_DIR="${BACKUP_BASE}/_backup_before_gold_${TIMESTAMP}"
readonly STATUS_FILE="/var/log/backup_status.json"

# ================= FUNKCJE POMOCNICZE =================
log_message() {
    local level="$1"
    local message="$2"
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] [$level] $message"
}

check_sudo() {
    if [ "$EUID" -ne 0 ]; then 
        log_message "ERROR" "Skrypt wymaga uprawnień root (sudo)"
        exit 1
    fi
}

# ================= WALIDACJA =================
check_sudo  # WYMAGANE UPRAWNIENIA ROOT - POPRAWKA 1.1
log_message "INFO" "Rozpoczynam walidację..."

# 1. Czy Gold Image istnieje?
if [ ! -d "$GOLD_DIR" ]; then
    log_message "ERROR" "Katalog Gold Image nie istnieje: $GOLD_DIR"
    exit 1
fi

# 2. Lista plików do deploy
FILES_TO_DEPLOY=(
    "etc/nginx/sites-available/dashboard"
    "etc/nginx/.htpasswd_dashboard" 
    "etc/supervisor/conf.d/dashboard-api.conf"
)

log_message "INFO" "Znaleziono $(find "$GOLD_DIR" -type f | wc -l) plików w Gold Image"
log_message "INFO" "Będę deployować ${#FILES_TO_DEPLOY[@]} kluczowych plików"

# ================= TWORZENIE PRE-BACKUP =================
log_message "INFO" "Tworzę pre-backup w: $BACKUP_DIR"
mkdir -p "$BACKUP_DIR"

BACKUP_COUNT=0
DEPLOY_COUNT=0

# ================= PETLA DEPLOY =================
for REL_PATH in "${FILES_TO_DEPLOY[@]}"; do
    SRC_FILE="$GOLD_DIR/$REL_PATH"
    DEST_FILE="/$REL_PATH"
    
    log_message "PROCESS" "Plik: $DEST_FILE"
    
    # 1. Czy plik źródłowy istnieje w Gold Image?
    if [ ! -f "$SRC_FILE" ]; then
        log_message "WARNING" "Brak pliku w Gold Image: $SRC_FILE - pomijam"
        continue
    fi
    
    # 2. Backup istniejącego pliku (jeśli istnieje)
    if [ -f "$DEST_FILE" ]; then
        BACKUP_PATH="$BACKUP_DIR/$REL_PATH"
        mkdir -p "$(dirname "$BACKUP_PATH")"
        cp -v "$DEST_FILE" "$BACKUP_PATH"
        BACKUP_COUNT=$((BACKUP_COUNT + 1))
        log_message "BACKUP" "Utworzono backup: $BACKUP_PATH"
    else
        log_message "INFO" "Plik docelowy nie istnieje - pierwszy deploy"
    fi
    
    # 3. Deploy nowego pliku
    mkdir -p "$(dirname "$DEST_FILE")"
    cp -v "$SRC_FILE" "$DEST_FILE"
    DEPLOY_COUNT=$((DEPLOY_COUNT + 1))
    log_message "DEPLOY" "Skopiowano: $SRC_FILE -> $DEST_FILE"
done

# ================= RESTART USŁUG =================
log_message "INFO" "Restartuję usługi..."

# 1. Nginx
if nginx -t 2>/dev/null; then
    systemctl reload nginx
    log_message "SERVICE" "Nginx przeładowany"
else
    log_message "ERROR" "Błąd konfiguracji Nginx - sprawdź: nginx -t"
    exit 1
fi

# 2. Supervisor
supervisorctl reread
supervisorctl update
supervisorctl restart all
log_message "SERVICE" "Supervisor zrestartowany"

# ================= LOGOWANIE STATUSU =================
log_message "INFO" "Zapisuję status operacji..."

STATUS_JSON=$(cat <<EOF
{
  "timestamp": "$(date -Iseconds)",
  "operation": "gold_image_deploy",
  "backup_dir": "$BACKUP_DIR",
  "files_backed_up": $BACKUP_COUNT,
  "files_deployed": $DEPLOY_COUNT,
  "success": true,
  "system": "Gold Image v1.1"
}
EOF
)

# Zapis do pliku statusu
mkdir -p "$(dirname "$STATUS_FILE")"
if [ -f "$STATUS_FILE" ]; then
    # Dodaj do istniejącego arraya
    TEMP_FILE=$(mktemp)
    jq --argjson new "$STATUS_JSON" '.operations += [$new]' "$STATUS_FILE" > "$TEMP_FILE" 2>/dev/null || echo "$STATUS_JSON" > "$TEMP_FILE"
    mv "$TEMP_FILE" "$STATUS_FILE"
else
    echo '{"operations": []}' | jq --argjson new "$STATUS_JSON" '.operations += [$new]' > "$STATUS_FILE" 2>/dev/null || echo "$STATUS_JSON" > "$STATUS_FILE"
fi

# ================= PODSUMOWANIE =================
echo ""
echo "=========================================="
echo "          DEPLOY ZAKOŃCZONY"
echo "=========================================="
echo "• Data:            $(date)"
echo "• Pre-backup:      $BACKUP_DIR"
echo "• Pliki backupowane: $BACKUP_COUNT"
echo "• Pliki deployed:    $DEPLOY_COUNT"
echo "• Status:          ZAPISANO w $STATUS_FILE"
echo ""
echo "WAŻNE: Zapisano pre-backup w:"
echo "       $BACKUP_DIR"
echo "       Użyj go do ręcznego rollbacku jeśli potrzeba."
echo "=========================================="
