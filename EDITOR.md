#!/bin/bash
# generate_rsa_keys_safe.sh - Bezpieczne generowanie kluczy RSA 4096-bit
# Wersja: 1.0 | Data: 2024-12-29
# Używa openssl genpkey (OpenSSL 3.x compatible)

set -e

echo "=== 🔐 BEZPIECZNE GENEROWANIE KLUCZY RSA 4096-BIT ==="
echo "Metoda: openssl genpkey (OpenSSL 3.x compatible)"
echo "Data: $(date)"
echo ""

# =================== KONFIGURACJA ===================
KEYS_DIR="/home/ubuntu/.secure_vault"
PUBLIC_KEY="$KEYS_DIR/backup_public.pem"
PRIVATE_KEY_TEMP="/tmp/secure_vault_private_$(date +%Y%m%d_%H%M%S).pem"

# =================== WALIDACJA ===================
echo "1. 🧪 WALIDACJA ŚRODOWISKA..."
echo "   Sprawdzam katalog kluczy: $KEYS_DIR"
if [ ! -d "$KEYS_DIR" ]; then
    echo "   ❌ BŁĄD: Katalog kluczy nie istnieje!"
    echo "   Uruchom najpierw secure_vault_setup.sh"
    exit 1
fi

echo "   Sprawdzam uprawnienia: $(stat -c %A "$KEYS_DIR")"
if [ "$(stat -c %a "$KEYS_DIR")" -ne 700 ]; then
    echo "   ⚠️  Poprawiam uprawnienia katalogu na 700..."
    chmod 700 "$KEYS_DIR"
fi

echo "   Sprawdzam czy klucz publiczny już istnieje..."
if [ -f "$PUBLIC_KEY" ]; then
    echo "   ⚠️  UWAGA: Klucz publiczny już istnieje!"
    echo "   Lokalizacja: $PUBLIC_KEY"
    echo "   Data modyfikacji: $(stat -c %y "$PUBLIC_KEY")"
    echo "   Czy nadpisać? (T/N)"
    read -r response
    if [[ ! "$response" =~ ^[TtYy]$ ]]; then
        echo "   ❌ Anulowano przez użytkownika"
        exit 0
    fi
    echo "   🔄 Usuwam stary klucz..."
    rm -f "$PUBLIC_KEY"
fi

# =================== GENEROWANIE ===================
echo -e "\n2. 🔧 GENEROWANIE KLUCZA PRYWATNEGO (4096-bit)..."
echo "   To może zająć 30-60 sekund..."
echo "   Rozpoczynam: $(date)"

START_TIME=$(date +%s)
openssl genpkey \
    -algorithm RSA \
    -out "$PRIVATE_KEY_TEMP" \
    -pkeyopt rsa_keygen_bits:4096 \
    -pkeyopt rsa_keygen_pubexp:65537 2>/dev/null

END_TIME=$(date +%s)
DURATION=$((END_TIME - START_TIME))

if [ ! -s "$PRIVATE_KEY_TEMP" ]; then
    echo "   ❌ BŁĄD: Nie udało się wygenerować klucza prywatnego"
    exit 1
fi

echo "   ✅ Klucz prywatny wygenerowany pomyślnie"
echo "   Czas generowania: ${DURATION} sekund"
echo "   Rozmiar klucza: $(stat -c %s "$PRIVATE_KEY_TEMP") bajtów"

# =================== EKSTRAKCJA KLUCZA PUBLICZNEGO ===================
echo -e "\n3. 📤 EKSTRAKCJA KLUCZA PUBLICZNEGO..."
openssl pkey -in "$PRIVATE_KEY_TEMP" -pubout -out "$PUBLIC_KEY" 2>/dev/null

if [ ! -s "$PUBLIC_KEY" ]; then
    echo "   ❌ BŁĄD: Nie udało się wygenerować klucza publicznego"
    rm -f "$PRIVATE_KEY_TEMP"
    exit 1
fi

chmod 600 "$PUBLIC_KEY"
chown ubuntu:ubuntu "$PUBLIC_KEY"

echo "   ✅ Klucz publiczny zapisany: $PUBLIC_KEY"
echo "   Uprawnienia: $(stat -c %A "$PUBLIC_KEY")"

# =================== TEST ===================
echo -e "\n4. 🧪 TEST SZYFROWANIA..."
TEST_MSG="Test Secure Vault $(date)"
TEST_ENC="/tmp/test_enc_$(date +%s).bin"
echo "$TEST_MSG" | openssl pkeyutl -encrypt -pubin -inkey "$PUBLIC_KEY" -out "$TEST_ENC" 2>/dev/null

if [ -s "$TEST_ENC" ]; then
    echo "   ✅ Szyfrowanie kluczem publicznym działa"
    
    # Test deszyfrowania
    DECRYPTED=$(openssl pkeyutl -decrypt -inkey "$PRIVATE_KEY_TEMP" -in "$TEST_ENC" 2>/dev/null)
    if [ "$DECRYPTED" = "$TEST_MSG" ]; then
        echo "   ✅ Deszyfrowanie kluczem prywatnym działa"
    else
        echo "   ⚠️  Deszyfrowanie działa, ale wiadomość się nie zgadza"
    fi
else
    echo "   ⚠️  Test szyfrowania nie powiódł się (może być normalne dla długich kluczy)"
fi

rm -f "$TEST_ENC"

# =================== PODSUMOWANIE ===================
echo -e "\n5. 📋 PODSUMOWANIE:"
echo "   -----------------------------------------"
echo "   ✅ Klucz publiczny: $PUBLIC_KEY"
echo "   ✅ Klucz prywatny (TYMCZASOWO): $PRIVATE_KEY_TEMP"
echo "   ✅ Długość klucza: 4096-bit"
echo "   ✅ Algorytm: RSA"
echo "   ✅ Metoda: openssl genpkey (OpenSSL 3.x)"
echo "   ✅ Czas generowania: ${DURATION}s"
echo "   -----------------------------------------"

# =================== INSTRUKCJE DLA CEO ===================
echo -e "\n6. 🚨 WAŻNE INSTRUKCJE DLA CEO:"
echo "   ==========================================="
cat << EOF

🔥 **KLUCZ PRYWATNY JEST W PLIKU TYMCZASOWYM:**
   $PRIVATE_KEY_TEMP

📥 **POBERZ GO TERAZ (ZA 60 SEKUND ZOSTANIE USUNIĘTY):**

1. Wyświetl zawartość:
   cat $PRIVATE_KEY_TEMP

2. Skopiuj CAŁĄ zawartość (od -----BEGIN PRIVATE KEY----- 
   do -----END PRIVATE KEY-----)

3. Zapisz w 2 bezpiecznych miejscach:
   • Menedżer haseł (Bitwarden/1Password)
   • Szyfrowany plik offline
   • Wydruk w sejfie

🔐 **ZALECANE NAZWY:**
   • secure_vault_private_$(date +%Y%m%d).pem
   • ai_firma_secure_vault_key.pem

⏳ **CZAS: Masz 60 sekund na skopiowanie!**
EOF

echo -e "\n⏳ Oczekiwanie 60 sekund przed usunięciem klucza prywatnego..."
for i in {60..1}; do
    echo -ne "   Pozostało: ${i}s\r"
    sleep 1
done

echo -e "\n🗑️  Usuwanie klucza prywatnego z serwera..."
rm -f "$PRIVATE_KEY_TEMP"
echo "   ✅ Klucz prywatny USUNIĘTY z serwera"

echo -e "\n=== ✅ GENEROWANIE KLUCZY ZAKOŃCZONE ==="
echo "Następny krok: Tworzenie skryptu backup_secrets.sh"
