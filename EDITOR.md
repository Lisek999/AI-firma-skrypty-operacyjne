#!/bin/bash
# MINIMALNY skrypt aktualizacji DISASTER_RECOVERY_PLAN.md
set -e
PLIK="/opt/ai_firma_dashboard/DISASTER_RECOVERY_PLAN.md"
KOPIA="$PLIK.backup_$(date +%Y%m%d_%H%M%S)"
cp "$PLIK" "$KOPIA"
cat >> "$PLIK" << 'EOF'

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
3. Rozpakuj: tar xzf backup.tar.gz
4. Przywróć pliki na nowy serwer

Krytyczne informacje

· Fingerprint klucza: 63:69:60:8b:36:33:b7:24:6b:d4:81:1f:db:8d:9c:ae
· Bez klucza prywatnego = utrata danych
· Testuj odszyfrowanie PRZED awarią
  EOF
  mkdir -p /home/ubuntu/ai_firma_dokumenty
  cp "$PLIK" /home/ubuntu/ai_firma_dokumenty/
  echo "SUKCES: Plan odzyskiwania zaktualizowany. Backup: $KOPIA"

```

**Po wklejeniu i ewentualnej ręcznej korekcie:** Uruchom go przez `getscript update_plan`.   
