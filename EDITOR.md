#!/bin/bash
# generate_rsa_keys_mobile.sh - Generowanie kluczy dla środowiska mobilnego
# Wersja: 1.1 | Data: 2024-12-29
# Klucz prywatny wyświetlany w terminalu do skopiowania

set -e

echo "=== 🔐 GENEROWANIE KLUCZY RSA (ŚRODOWISKO MOBILNE) ==="
echo "UWAGA: Klucz prywatny zostanie WYŚWIETLONY w terminalu"
echo "Data: $(date)"
echo ""

# =================== KONFIGURACJA ===================
KEYS_DIR="/home/ubuntu/.secure_vault"
PUBLIC_KEY="$KEYS_DIR/backup_public.pem"

# =================== WALIDACJA ===================
echo "1. 🧪 PRZYGOTOWANIE..."
if [ ! -d "$KEYS_DIR" ]; then
    echo "   ❌ BŁĄD: Brak katalogu .secure_vault"
    echo "   Uruchom najpierw secure_vault_setup.sh"
    exit 1
fi

if [ -f "$PUBLIC_KEY" ]; then
    echo "   ⚠️  Klucz publiczny już istnieje!"
    echo "   Czy nadpisać? (T/N)"
    read -r response
    if [[ ! "$response" =~ ^[TtYy]$ ]]; then
        echo "   ❌ Anulowano"
        exit 0
    fi
    rm -f "$PUBLIC_KEY"
fi

# =================== GENEROWANIE ===================
echo -e "\n2. 🔧 GENEROWANIE KLUCZA PRYWATNEGO (4096-bit)..."
echo "   To może zająć 30-60 sekund..."
echo "   Rozpoczynam: $(date)"

PRIVATE_KEY_CONTENT=$(openssl genpkey \
    -algorithm RSA \
    -pkeyopt rsa_keygen_bits:4096 \
    -pkeyopt rsa_keygen_pubexp:65537 2>/dev/null)

if [ -z "$PRIVATE_KEY_CONTENT" ]; then
    echo "   ❌ BŁĄD: Nie udało się wygenerować klucza"
    exit 1
fi

echo "   ✅ Klucz prywatny wygenerowany pomyślnie"

# =================== ZAPIS KLUCZA PUBLICZNEGO ===================
echo -e "\n3. 📤 TWORZENIE KLUCZA PUBLICZNEGO..."
echo "$PRIVATE_KEY_CONTENT" | openssl pkey -pubout -out "$PUBLIC_KEY" 2>/dev/null

if [ ! -s "$PUBLIC_KEY" ]; then
    echo "   ❌ BŁĄD: Nie udało się utworzyć klucza publicznego"
    exit 1
fi

chmod 600 "$PUBLIC_KEY"
chown ubuntu:ubuntu "$PUBLIC_KEY"

echo "   ✅ Klucz publiczny zapisany: $PUBLIC_KEY"

# =================== WYŚWIETLENIE KLUCZA PRYWATNEGO ===================
echo -e "\n4. 🚨 ==================================================="
echo "   🔥 KLUCZ PRYWATNY - SKOPIUJ CAŁOŚĆ PONIŻEJ 🔥"
echo "   ==================================================="
echo ""
echo "$PRIVATE_KEY_CONTENT"
echo ""
echo "   ==================================================="
echo "   ✅ Koniec klucza prywatnego"
echo "   ==================================================="

# =================== INSTRUKCJE KOPIOWANIA ===================
echo -e "\n5. 📋 INSTRUKCJE KOPIOWANIA W TERMINUSIE:"
cat << 'EOF'

📥 **JAK SKOPIOWAĆ W TERMINUSIE:**

1. DOTKNIJ i PRZYTRZYMAJ w dowolnym miejscu klucza powyżej
2. Wybierz "SELECT ALL" (Zaznacz wszystko)
3. Wybierz "COPY" (Kopiuj)
4. Wklej do:
   • Notatnika na telefonie
   • Aplikacji do notatek
   • Menedżera haseł

💾 **ZALECANE NAZWY PLIKU:**
   • secure_vault_private_$(date +%Y%m%d).pem
   • ai_firma_secure_vault_key.pem

⚠️  **OSTRZEŻENIA:**
   • Klucz jest wyświetlony TYLKO RAZ
   • Nie zapisuj na serwerze
   • Zachowaj w 2 bezpiecznych miejscach
   • Bez tego klucza backupy są BEZUŻYTECZNE
EOF

# =================== TEST ===================
echo -e "\n6. 🧪 TEST SYGNALIZACYJNY..."
echo "   Testuję czy klucz publiczny działa..."
TEST_MSG="OK"
if echo "$TEST_MSG" | openssl pkeyutl -encrypt -pubin -inkey "$PUBLIC_KEY" -out /tmp/test_enc.bin 2>/dev/null; then
    echo "   ✅ Klucz publiczny działa"
else
    echo "   ⚠️  Test szyfrowania pominięty (może wymagać więcej danych)"
fi
rm -f /tmp/test_enc.bin 2>/dev/null

echo -e "\n=== ✅ GENEROWANIE ZAKOŃCZONE ==="
echo "Klucz publiczny: $PUBLIC_KEY"
echo "Klucz prywatny: SKOPIOWANY POWYŻEJ"
echo "Następny krok: backup_secrets.sh"
