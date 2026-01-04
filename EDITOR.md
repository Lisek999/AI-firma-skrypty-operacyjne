```bash
#!/bin/bash
# Skrypt: update_disaster_plan_main.sh
# Cel: Dopisanie rozdziału "Secure Vault" do DISASTER_RECOVERY_PLAN.md
# Autor: Wojtek (AI) | Data: 2024-12-29 | Protokół: DA-CZ-W

set -e

LOG_FILE="/var/log/update_disaster_plan.log"
BACKUP_FILE="/opt/ai_firma_dashboard/DISASTER_RECOVERY_PLAN.md.backup_$(date +%Y%m%d_%H%M%S)"
TARGET_FILE="/opt/ai_firma_dashboard/DISASTER_RECOVERY_PLAN.md"

echo "=== Rozpoczynam aktualizację planu odzyskiwania (Secure Vault) ===" | tee -a "$LOG_FILE"
echo "Czas: $(date)" | tee -a "$LOG_FILE"

# 1. Tworzenie kopii zapasowej pliku
if [ -f "$TARGET_FILE" ]; then
    echo "1. Tworzę kopię zapasową pliku: $BACKUP_FILE" | tee -a "$LOG_FILE"
    cp "$TARGET_FILE" "$BACKUP_FILE"
    echo "   Kopia utworzona pomyślnie." | tee -a "$LOG_FILE"
else
    echo "ERROR: Plik docelowy $TARGET_FILE nie istnieje!" | tee -a "$LOG_FILE"
    exit 1
fi

# 2. Dopisanie nowego rozdziału
echo "2. Dopisuję rozdział 'Secure Vault - Warstwa 3'..." | tee -a "$LOG_FILE"

cat >> "$TARGET_FILE" << 'EOF'

## Secure Vault - Warstwa 3: Ochrona kluczy i haseł

### Cel
Zaszyfrowany backup wrażliwych danych z odzyskaniem TYLKO przez CEO.

### Architektura
- Klucz publiczny: `/home/ubuntu/.secure_vault/backup_public.pem`
- Klucz prywatny: OFFLINE u CEO (NIGDY na serwerze)
- Backupy: `/home/ubuntu/ai_firma_backups/secure_vault/backups/secrets_*.tar.gz.enc`
- Automatyzacja: Codziennie 3:30

### Procedura odzyskiwania
1. **Pobierz backup** z serwera lub z kopii CEO
2. **Odszyfruj** (na komputerze CEO):
   ```bash
   openssl pkeyutl -decrypt -inkey private.pem -in backup.enc -out backup.tar.gz
```

1. Rozpakuj: tar xzf backup.tar.gz
2. Przywróć pliki na nowy serwer

Krytyczne informacje

· Fingerprint klucza: 63:69:60:8b:36:33:b7:24:6b:d4:81:1f:db:8d:9c:ae
· Bez klucza prywatnego = utrata danych
· Testuj odszyfrowanie PRZED awarią

EOF

echo "   Rozdział dopisany pomyślnie." | tee -a "$LOG_FILE"

3. Utworzenie kopii w katalogu dokumentacji

DOCS_DIR="/home/ubuntu/ai_firma_dokumenty"
echo "3. Tworzę kopię w katalogu dokumentacji: $DOCS_DIR/" | tee -a "$LOG_FILE"
mkdir -p "$DOCS_DIR"
cp "$TARGET_FILE" "$DOCS_DIR/DISASTER_RECOVERY_PLAN.md"
echo "   Kopia utworzona pomyślnie." | tee -a "$LOG_FILE"

4. Weryfikacja końcowa

echo "4. Weryfikacja: sprawdzam czy plik został zaktualizowany..." | tee -a "$LOG_FILE"
if tail -10 "$TARGET_FILE" | grep -q "Secure Vault - Warstwa 3"; then
echo "   SUKCES: Nowy rozdział znajduje się w pliku." | tee -a "$LOG_FILE"
echo "   Ścieżka do backupu: $BACKUP_FILE" | tee -a "$LOG_FILE"
echo "   Ścieżka do kopii w dokumentacji: $DOCS_DIR/DISASTER_RECOVERY_PLAN.md" | tee -a "$LOG_FILE"
else
echo "   ERROR: Nie znaleziono nowego rozdziału w pliku." | tee -a "$LOG_FILE"
echo "   Przywracam oryginalny plik z backupu..." | tee -a "$LOG_FILE"
cp "$BACKUP_FILE" "$TARGET_FILE"
exit 1
fi

echo "=== Aktualizacja zakończona pomyślnie ===" | tee -a "$LOG_FILE"

```
