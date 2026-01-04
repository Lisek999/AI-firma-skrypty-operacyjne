#!/bin/bash
# backup_secrets.sh - Backup i szyfrowanie plików wrażliwych Secure Vault
# Wersja: 1.0 | Data: 2024-12-29
# Szyfrowanie asymetryczne RSA 4096-bit

set -e

# =================== KONFIGURACJA ===================
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"
BACKUP_ROOT="/home/ubuntu/ai_firma_backups"
SECURE_VAULT_DIR="$BACKUP_ROOT/secure_vault"
BACKUPS_DIR="$SECURE_VAULT_DIR/backups"
PUBLIC_KEY="/home/ubuntu/.secure_vault/backup_public.pem"
STATUS_FILE="/var/log/backup_status.json"
LOG_FILE="$SECURE_VAULT_DIR/backup_secrets.log"

# Lista plików do backupu (można rozszerzyć)
SOURCE_FILES=(
    "/etc/nginx/.htpasswd_dashboard"
    "/opt/ai_firma_dashboard/.env"  # Może nie istnieć
)

# =================== FUNKCJE POMOCNICZE ===================
log_message() {
    echo "[$(date '+%Y-%m-%d %H:%M:%S')] $1" | tee -a "$LOG_FILE"
}

update_status() {
    local status_data="$1"
    local temp_file="/tmp/backup_status_$$.json"
    
    if [ -f "$STATUS_FILE" ]; then
        # Aktualizuj istniejący plik
        jq --argjson new "$status_data" '.secure_vault = $new' "$STATUS_FILE" > "$temp_file" 2>/dev/null
    else
        # Utwórz nowy plik
        echo "{\"secure_vault\": $status_data}" > "$temp_file"
    fi
    
    # Zapisz z zachowaniem uprawnień
    sudo cp "$temp_file" "$STATUS_FILE" 2>/dev/null || cp "$temp_file" "$STATUS_FILE"
    sudo chmod 644 "$STATUS_FILE" 2>/dev/null || chmod 644 "$STATUS_FILE"
    rm -f "$temp_file"
}

# =================== WALIDACJA ===================
log_message "=== 🛡️ URUCHOMIENIE BACKUP SECURE VAULT ==="

# Sprawdź klucz publiczny
if [ ! -f "$PUBLIC_KEY" ]; then
    log_message "❌ BŁĄD: Brak klucza publicznego: $PUBLIC_KEY"
    exit 1
fi

# Sprawdź katalog backupów
if [ ! -d "$BACKUPS_DIR" ]; then
    log_message "❌ BŁĄD: Brak katalogu backupów: $BACKUPS_DIR"
    exit 1
fi

# =================== PRZYGOTOWANIE ===================
TIMESTAMP=$(date '+%Y%m%d_%H%M%S')
BACKUP_NAME="secrets_${TIMESTAMP}"
TEMP_DIR="/tmp/secure_vault_backup_$$"
TEMP_ARCHIVE="$TEMP_DIR/${BACKUP_NAME}.tar.gz"
TEMP_ENCRYPTED="$TEMP_DIR/${BACKUP_NAME}.tar.gz.enc"
FINAL_FILE="$BACKUPS_DIR/${BACKUP_NAME}.tar.gz.enc"

mkdir -p "$TEMP_DIR"
log_message "📦 Przygotowanie backupu: $BACKUP_NAME"

# =================== ZBIERANIE PLIKÓW ===================
log_message "🔍 Zbieranie plików źródłowych..."

EXISTING_FILES=()
MISSING_FILES=()

for file in "${SOURCE_FILES[@]}"; do
    if [ -f "$file" ] && [ -r "$file" ]; then
        EXISTING_FILES+=("$file")
        log_message "   ✅ $file (dostępny)"
    else
        MISSING_FILES+=("$file")
        log_message "   ⚠️  $file (brak lub brak dostępu)"
    fi
done

if [ ${#EXISTING_FILES[@]} -eq 0 ]; then
    log_message "❌ BŁĄD: Brak plików do backupu!"
    rm -rf "$TEMP_DIR"
    exit 1
fi

# =================== TWORZENIE ARCHIWUM ===================
log_message "📁 Tworzenie archiwum..."

# Utwórz manifest plików
MANIFEST_FILE="$TEMP_DIR/manifest.txt"
{
    echo "Secure Vault Backup - $(date)"
    echo "Timestamp: $TIMESTAMP"
    echo "Files included:"
    printf '%s\n' "${EXISTING_FILES[@]}"
    echo ""
    echo "Files missing:"
    printf '%s\n' "${MISSING_FILES[@]}"
} > "$MANIFEST_FILE"

# Dodaj manifest do archiwum
tar czf "$TEMP_ARCHIVE" -C / "${EXISTING_FILES[@]}" -C "$TEMP_DIR" "manifest.txt" 2>/dev/null

if [ ! -s "$TEMP_ARCHIVE" ]; then
    log_message "❌ BŁĄD: Nie udało się utworzyć archiwum"
    rm -rf "$TEMP_DIR"
    exit 1
fi

ARCHIVE_SIZE=$(stat -c %s "$TEMP_ARCHIVE")
log_message "   ✅ Archiwum utworzone: $ARCHIVE_SIZE bajtów"

# =================== SZYFROWANIE ===================
log_message "🔐 Szyfrowanie kluczem publicznym..."

# Szyfruj za pomocą klucza publicznego
if openssl pkeyutl -encrypt -pubin -inkey "$PUBLIC_KEY" -in "$TEMP_ARCHIVE" -out "$TEMP_ENCRYPTED" 2>/dev/null; then
    ENCRYPTED_SIZE=$(stat -c %s "$TEMP_ENCRYPTED")
    log_message "   ✅ Zaszyfrowano: $ENCRYPTED_SIZE bajtów"
else
    log_message "❌ BŁĄD: Nie udało się zaszyfrować archiwum"
    rm -rf "$TEMP_DIR"
    exit 1
fi

# =================== ZAPIS BACKUPU ===================
log_message "💾 Zapis backupu..."

cp "$TEMP_ENCRYPTED" "$FINAL_FILE"
chmod 600 "$FINAL_FILE"
chown ubuntu:ubuntu "$FINAL_FILE"

# Oblicz hash dla weryfikacji
FILE_HASH=$(sha256sum "$FINAL_FILE" | awk '{print $1}')
log_message "   ✅ Backup zapisany: $FINAL_FILE"
log_message "   🔑 Hash SHA-256: $FILE_HASH"

# =================== AKTUALIZACJA STATUSU ===================
log_message "📝 Aktualizacja statusu..."

STATUS_JSON=$(cat << EOF
{
    "timestamp": "$(date -Iseconds)",
    "backup_name": "$BACKUP_NAME",
    "file": "$(basename "$FINAL_FILE")",
    "size_bytes": $ENCRYPTED_SIZE,
    "hash_sha256": "$FILE_HASH",
    "files_included": [$(printf '"%s",' "${EXISTING_FILES[@]}" | sed 's/,$//')],
    "files_missing": [$(printf '"%s",' "${MISSING_FILES[@]}" | sed 's/,$//')],
    "status": "success"
}
EOF
)

update_status "$STATUS_JSON"
log_message "   ✅ Status zaktualizowany"

# =================== ROTACJA STARYCH BACKUPÓW ===================
log_message "🗑️  Sprawdzanie rotacji backupów (>30 dni)..."

FIND_CMD="find \"$BACKUPS_DIR\" -name \"secrets_*.tar.gz.enc\" -mtime +30"
OLD_FILES=$(eval "$FIND_CMD")

if [ -n "$OLD_FILES" ]; then
    COUNT=$(echo "$OLD_FILES" | wc -l)
    log_message "   🔄 Usuwanie $COUNT starych backupów..."
    echo "$OLD_FILES" | xargs rm -f
    log_message "   ✅ Rotacja wykonana"
else
    log_message "   ✅ Brak starych backupów do usunięcia"
fi

# =================== SPRZĄTANIE ===================
rm -rf "$TEMP_DIR"
log_message "🧹 Posprzątano pliki tymczasowe"

# =================== PODSUMOWANIE ===================
BACKUP_COUNT=$(find "$BACKUPS_DIR" -name "secrets_*.tar.gz.enc" | wc -l)
TOTAL_SIZE=$(find "$BACKUPS_DIR" -name "secrets_*.tar.gz.enc" -exec stat -c %s {} \; | awk '{sum+=$1} END {print sum}')

log_message "=== ✅ BACKUP ZAKOŃCZONY POMYŚLNIE ==="
log_message "📊 Statystyki Secure Vault:"
log_message "   • Liczba backupów: $BACKUP_COUNT"
log_message "   • Łączny rozmiar: $TOTAL_SIZE bajtów"
log_message "   • Najnowszy backup: $(basename "$FINAL_FILE")"
log_message "   • Hash weryfikacyjny: $FILE_HASH"

echo "✅ Backup Secure Vault wykonany: $FINAL_FILE"
