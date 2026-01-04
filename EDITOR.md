#!/bin/bash
# configure_secure_vault_cron.sh - Konfiguracja cron dla Secure Vault
# Wersja: 1.0 | Data: 2024-12-29

set -e

echo "=== 🔧 KONFIGURACJA CRON DLA SECURE VAULT ==="
echo "Data: $(date)"
echo ""

# =================== KONFIGURACJA ===================
CRON_ENTRY="# Secure Vault - backup tajemnic (3:30 codziennie)"
CRON_COMMAND="30 3 * * * /home/ubuntu/ai_firma_backups/secure_vault/backup_secrets.sh >> /home/ubuntu/ai_firma_backups/secure_vault/backup_secrets_cron.log 2>&1"
BACKUP_SCRIPT="/home/ubuntu/ai_firma_backups/secure_vault/backup_secrets.sh"
CRON_LOG="/home/ubuntu/ai_firma_backups/secure_vault/backup_secrets_cron.log"

# =================== WALIDACJA ===================
echo "1. 🧪 WALIDACJA PRZED KONFIGURACJĄ..."
echo "   Sprawdzam skrypt backup: $BACKUP_SCRIPT"
if [ ! -f "$BACKUP_SCRIPT" ]; then
    echo "   ❌ BŁĄD: Brak skryptu backup!"
    exit 1
fi

if [ ! -x "$BACKUP_SCRIPT" ]; then
    echo "   ⚠️  Skrypt nie jest wykonywalny, naprawiam..."
    chmod +x "$BACKUP_SCRIPT"
fi

echo "   ✅ Skrypt backup jest gotowy"

# =================== KONFIGURACJA CRON ===================
echo -e "\n2. ⏰ KONFIGURACJA CRON..."
echo "   Obecny cron:"
sudo -u ubuntu crontab -l | grep -i "backup" | head -5 || echo "   (brak wpisów backup)"

echo -e "\n   Dodaję nowy wpis:"
echo "   $CRON_ENTRY"
echo "   $CRON_COMMAND"

# Usuń stare wpisy dla backup_secrets.sh i dodaj nowy
(sudo -u ubuntu crontab -l 2>/dev/null | grep -v "backup_secrets.sh"; echo "$CRON_ENTRY"; echo "$CRON_COMMAND") | sudo -u ubuntu crontab -

echo "   ✅ Cron skonfigurowany"

# =================== TWORZENIE PLIKU LOGÓW ===================
echo -e "\n3. 📝 PRZYGOTOWANIE LOGÓW..."
if [ ! -f "$CRON_LOG" ]; then
    echo "   Tworzę plik logów: $CRON_LOG"
    touch "$CRON_LOG"
    chmod 600 "$CRON_LOG"
    chown ubuntu:ubuntu "$CRON_LOG"
else
    echo "   Plik logów już istnieje"
    chmod 600 "$CRON_LOG" 2>/dev/null || true
fi

echo "   Uprawnienia logów: $(stat -c %A "$CRON_LOG")"

# =================== TEST CRON ===================
echo -e "\n4. 🧪 TEST KONFIGURACJI..."
echo "   Testuję czy skrypt uruchomi się z cron (symulacja)..."
cd /home/ubuntu/ai_firma_backups/secure_vault/

# Test szybkiego wykonania
echo "   Rozpoczynam test: $(date)"
TEST_OUTPUT=$(./backup_secrets.sh 2>&1 | tail -5)
echo "   Zakończono test: $(date)"

if echo "$TEST_OUTPUT" | grep -q "BACKUP ZAKOŃCZONY POMYŚLNIE"; then
    echo "   ✅ Test wykonania zakończony sukcesem"
else
    echo "   ⚠️  Test wykonany, ale bez końcowego komunikatu"
fi

# =================== AKTUALIZACJA STATUSU ===================
echo -e "\n5. 📊 AKTUALIZACJA STATUSU SYSTEMU..."
STATUS_FILE="/var/log/backup_status.json"

if [ -f "$STATUS_FILE" ]; then
    echo "   Aktualizuję backup_status.json..."
    
    # Tworzymy JSON z informacją o cron
    CRON_JSON=$(cat << EOF
{
  "secure_vault_cron": {
    "configured": true,
    "time": "3:30 daily",
    "configured_at": "$(date -Iseconds)",
    "log_file": "$CRON_LOG",
    "script": "$BACKUP_SCRIPT"
  }
}
EOF
    )
    
    # Aktualizujemy plik statusu
    if command -v jq >/dev/null 2>&1; then
        echo "$CRON_JSON" | jq '.' > /tmp/cron_status.json
        sudo jq --argfile cron /tmp/cron_status.json '. + $cron' "$STATUS_FILE" > /tmp/updated_status.json 2>/dev/null
        if [ $? -eq 0 ]; then
            sudo cp /tmp/updated_status.json "$STATUS_FILE"
            sudo chmod 644 "$STATUS_FILE"
            echo "   ✅ Status zaktualizowany (z użyciem jq)"
        else
            # Alternatywna metoda
            sudo cp "$STATUS_FILE" "${STATUS_FILE}.backup"
            echo "$CRON_JSON" | sudo tee -a "$STATUS_FILE" > /dev/null
            echo "   ✅ Status zaktualizowany (metoda alternatywna)"
        fi
        rm -f /tmp/cron_status.json /tmp/updated_status.json
    else
        echo "   ⚠️  jq nie jest dostępne, pomijam aktualizację statusu"
    fi
else
    echo "   ℹ️  Plik statusu nie istnieje, tworzę..."
    echo '{"secure_vault_cron": {"configured": true, "time": "3:30 daily"}}' | sudo tee "$STATUS_FILE" > /dev/null
    sudo chmod 644 "$STATUS_FILE"
fi

# =================== PODSUMOWANIE ===================
echo -e "\n6. 📋 PODSUMOWANIE KONFIGURACJI:"
echo "   -----------------------------------------"
echo "   ✅ Cron skonfigurowany: 30 3 * * *"
echo "   ✅ Skrypt: $BACKUP_SCRIPT"
echo "   ✅ Logi: $CRON_LOG"
echo "   ✅ Status: /var/log/backup_status.json"
echo "   -----------------------------------------"

echo -e "\n7. ⏰ HARMONOGRAM BACKUPÓW AI FIRMA:"
cat << 'EOF'

⏰ HARMONOGRAM BACKUPÓW AI FIRMA:
─────────────────────────────────
03:00 - Backup dashboardu
03:15 - Backup skryptów  
03:30 - Secure Vault (tajemnice) ✅
04:00 - Backup tygodniowy (niedziela)

📅 Codziennie: 3:00, 3:15, 3:30
📅 Tygodniowo: Niedziela 4:00

📁 Logi Secure Vault:
• backup_secrets.log - logi ze skryptu
• backup_secrets_cron.log - logi z cron

🔧 Ręczne uruchomienie:
cd /home/ubuntu/ai_firma_backups/secure_vault/
./backup_secrets.sh
EOF

echo -e "\n=== ✅ KONFIGURACJA CRON ZAKOŃCZONA ==="
echo "Secure Vault będzie uruchamiany automatycznie codziennie o 3:30"
echo "Następny krok: Aktualizacja DISASTER_RECOVERY_PLAN.md"
