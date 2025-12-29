#!/bin/bash
# secure_vault_setup.sh - Inicjalizacja struktury Secure Vault
# Wersja: 1.0 | Data: 2024-12-29
# Autor: Wojtek (AI Programista) pod kierownictwem CEO Tomka

set -e  # Zatrzymaj przy pierwszym błędzie

echo "=== 🛡️ INICJALIZACJA SECURE VAULT - WARSTWA 3 ==="
echo "Data wykonania: $(date)"
echo "Użytkownik: $(whoami)"
echo ""

# =================== KONFIGURACJA ===================
BACKUP_ROOT="/home/ubuntu/ai_firma_backups"
SECURE_VAULT_DIR="$BACKUP_ROOT/secure_vault"
BACKUPS_DIR="$SECURE_VAULT_DIR/backups"
KEYS_DIR="/home/ubuntu/.secure_vault"

# =================== WALIDACJA ===================
echo "1. 🧪 WALIDACJA WEJŚCIA..."
echo "   Sprawdzam czy jestem użytkownikiem ubuntu..."
if [ "$(whoami)" != "ubuntu" ]; then
    echo "   ⚠️  Uwaga: Skrypt uruchomiony jako $(whoami), a nie ubuntu"
    echo "   Kontynuuję, ale uprawnienia mogą wymagać dostosowania"
fi

# =================== TWORZENIE STRUKTURY ===================
echo -e "\n2. 📁 TWORZENIE STRUKTURY KATALOGÓW..."

# 1. Główny katalog backupów
if [ ! -d "$BACKUP_ROOT" ]; then
    echo "   📂 Tworzenie: $BACKUP_ROOT"
    mkdir -p "$BACKUP_ROOT"
    chmod 755 "$BACKUP_ROOT"
    chown ubuntu:ubuntu "$BACKUP_ROOT"
    echo "   ✅ Utworzono główny katalog backupów"
else
    echo "   ℹ️  Katalog $BACKUP_ROOT już istnieje"
    # Upewnij się o uprawnieniach
    chmod 755 "$BACKUP_ROOT" 2>/dev/null || true
    chown ubuntu:ubuntu "$BACKUP_ROOT" 2>/dev/null || true
fi

# 2. Katalog Secure Vault (najważniejszy - chmod 700)
if [ ! -d "$SECURE_VAULT_DIR" ]; then
    echo "   🔐 Tworzenie: $SECURE_VAULT_DIR"
    mkdir -p "$SECURE_VAULT_DIR"
    chmod 700 "$SECURE_VAULT_DIR"  # TYLKO właściciel ma dostęp
    chown ubuntu:ubuntu "$SECURE_VAULT_DIR"
    echo "   ✅ Utworzono katalog Secure Vault (chmod 700)"
else
    echo "   ℹ️  Katalog $SECURE_VAULT_DIR już istnieje"
    # WYMUSZENIE bezpiecznych uprawnień
    chmod 700 "$SECURE_VAULT_DIR" 2>/dev/null || echo "   ⚠️  Nie mogę zmienić uprawnień (może wymagać sudo)"
    chown ubuntu:ubuntu "$SECURE_VAULT_DIR" 2>/dev/null || echo "   ⚠️  Nie mogę zmienić właściciela"
fi

# 3. Podkatalog na zaszyfrowane backupy
if [ ! -d "$BACKUPS_DIR" ]; then
    echo "   💾 Tworzenie: $BACKUPS_DIR"
    mkdir -p "$BACKUPS_DIR"
    chmod 700 "$BACKUPS_DIR"
    chown ubuntu:ubuntu "$BACKUPS_DIR"
    echo "   ✅ Utworzono katalog na zaszyfrowane backupy"
else
    echo "   ℹ️  Katalog $BACKUPS_DIR już istnieje"
    chmod 700 "$BACKUPS_DIR" 2>/dev/null || true
fi

# 4. Katalog na klucze (oddzielny, ukryty)
if [ ! -d "$KEYS_DIR" ]; then
    echo "   🔑 Tworzenie: $KEYS_DIR"
    mkdir -p "$KEYS_DIR"
    chmod 700 "$KEYS_DIR"
    chown ubuntu:ubuntu "$KEYS_DIR"
    echo "   ✅ Utworzono ukryty katalog na klucze"
else
    echo "   ℹ️  Katalog $KEYS_DIR już istnieje"
    chmod 700 "$KEYS_DIR" 2>/dev/null || true
fi

# =================== WERYFIKACJA ===================
echo -e "\n3. 🔍 WERYFIKACJA UTWORZONEJ STRUKTURY..."

echo "   Struktura katalogów:"
tree -a -L 3 "$BACKUP_ROOT" 2>/dev/null || {
    echo "   📊 Alternatywne wyświetlenie:"
    ls -la "$BACKUP_ROOT" 2>/dev/null || echo "   ❌ Nie mogę wyświetlić $BACKUP_ROOT"
    if [ -d "$SECURE_VAULT_DIR" ]; then
        ls -la "$SECURE_VAULT_DIR" 2>/dev/null || echo "   ❌ Nie mogę wyświetlić $SECURE_VAULT_DIR"
    fi
}

echo -e "\n4. 📋 PODSUMOWANIE UPRAWNIEŃ:"
echo "   Katalog                | Uprawnienia | Właściciel"
echo "   -----------------------|-------------|-----------"
for dir in "$BACKUP_ROOT" "$SECURE_VAULT_DIR" "$BACKUPS_DIR" "$KEYS_DIR"; do
    if [ -d "$dir" ]; then
        perms=$(stat -c "%A" "$dir" 2>/dev/null || echo "BŁĄD")
        owner=$(stat -c "%U:%G" "$dir" 2>/dev/null || echo "BŁĄD")
        echo "   $(basename "$dir") | $perms | $owner"
    fi
done

# =================== DOKUMENTACJA ===================
echo -e "\n5. 📝 DOKUMENTACJA STRUKTURY:"
cat << EOF

🗂️  STRUKTURA SECURE VAULT:
─────────────────────────────────────────
$BACKUP_ROOT/                    (755) - Główny katalog backupów
├── secure_vault/               (700) - Katalog Secure Vault (TYLKO właściciel)
│   └── backups/                (700) - Zaszyfrowane archiwa
└── ... (inne katalogi backupów mogą istnieć)

$KEYS_DIR/                      (700) - Ukryty katalog na klucze (w home ubuntu)

📋 PRZEZNACZENIE:
• secure_vault/backups/ - przechowuje zaszyfrowane pliki .tar.gz.enc
• .secure_vault/ - przechowuje klucz publiczny (backup_public.pem)
• Skrypt backup_secrets.sh będzie w secure_vault/

⚠️  UWAGI BEZPIECZEŃSTWA:
• chmod 700 oznacza, że tylko użytkownik 'ubuntu' ma dostęp
• Klucz prywatny NIGDY nie trafia na serwer
• Zaszyfrowane pliki mogą być odszyfrowane TYLKO z kluczem prywatnym CEO
EOF

echo -e "\n=== ✅ INICJALIZACJA ZAKOŃCZONA ==="
echo "Następny krok: Generowanie pary kluczy RSA"
echo "Czas wykonania: $(date)"
