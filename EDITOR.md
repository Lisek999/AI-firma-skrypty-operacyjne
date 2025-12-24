#!/bin/bash
# ============================================================================
# SKRYPT: Konfiguracja HTTP Basic Auth dla dashboardu (57.128.247.215)
# AUTOR: System Wojtek/CEO
# DATA: 2024-12-24
# MODEL: 2-plikowy (Gotowy do EDITOR.md)
# WERSJA: 1.5 FINAL (Basic Auth tylko dla frontendu, bez /api/)
# ============================================================================
# ZASADY:
# 1. Używa STAŁYCH poświadczeń: Agnostyk / Castorama13.
# 2. Zabezpiecza TYLKO frontend (location /). Backend (/api/) dostępny lokalnie.
# 3. Zawiera pełną logikę: backup, modyfikację, walidację, rollback.
# 4. Działa TYLKO z uprawnieniami sudo.
# 5. Dystrybucja: GitHub (EDITOR.md) -> getscript -> VPS.
# ============================================================================

# --- BEZPIECZEŃSTWO ---
set -e          # Zatrzymaj się przy pierwszym błędzie.
set -u          # Wykrywaj użycie niezdefiniowanych zmiennych.
set -o pipefail # Rozpoznawaj błędy w potokach (|).

# --- KONFIGURACJA (ZMIENNE STAŁE) ---
NGINX_SITE_CONFIG="/etc/nginx/sites-available/dashboard"  # Ścieżka do pliku konfiguracji
BACKUP_DIR="/etc/nginx/backup"                            # Katalog na kopie zapasowe
HTPASSWD_FILE="/etc/nginx/.htpasswd_dashboard"            # Plik z hasłami
SERVER_IP="57.128.247.215"                                # Docelowy adres IP

# --- STAŁE POŚWIADCZENIA (DECYZJA CEO: STAŁE) ---
AUTH_USER="Agnostyk"
AUTH_PASS="Castorama13"

# --- FUNKCJE POMOCNICZE ---
function sprawdz_narzędzia() {
    echo "[1] Sprawdzanie wymaganych narzędzi (nginx, htpasswd)..."
    
    # Sprawdź, czy nginx jest zainstalowany (poprzez sprawdzenie usługi)
    if ! systemctl is-active --quiet nginx 2>/dev/null && ! command -v nginx >/dev/null 2>&1; then
        echo "   [BŁĄD] Nie znaleziono usługi 'nginx' ani polecenia 'nginx'."
        echo "   [ROZWIĄZANIE] Zainstaluj: sudo apt update && sudo apt install -y nginx"
        exit 1
    else
        echo "   [OK] Nginx jest dostępny."
    fi
    
    # Sprawdź, czy htpasswd jest dostępny
    if ! command -v htpasswd >/dev/null 2>&1; then
        echo "   [BŁĄD] Nie znaleziono polecenia 'htpasswd'."
        echo "   [ROZWIĄZANIE] Zainstaluj pakiet: sudo apt install -y apache2-utils"
        exit 1
    else
        echo "   [OK] Narzędzie 'htpasswd' (apache2-utils) jest dostępne."
    fi
    echo ""
}

function utworz_kopie_zapasowa() {
    echo "[2] Tworzenie kopii zapasowej konfiguracji..."
    
    # 1. SPRAWDŹ, CZY PLIK KONFIGURACYJNY ISTNIEJE
    if [[ ! -f "$NGINX_SITE_CONFIG" ]]; then
        echo "   [BŁĄD] Nie znaleziono pliku konfiguracyjnego: $NGINX_SITE_CONFIG"
        echo "   [ROZWIĄZANIE] Upewnij się, że Nginx jest poprawnie skonfigurowany dla dashboardu."
        exit 1
    fi
    echo "   [OK] Znaleziono plik konfiguracyjny: $NGINX_SITE_CONFIG"
    
    # 2. UTWÓRZ KATALOG BACKUP, JEŚLI NIE ISTNIEJE
    if [[ ! -d "$BACKUP_DIR" ]]; then
        echo "   [INFO] Katalog backup nie istnieje. Tworzę: $BACKUP_DIR"
        mkdir -p "$BACKUP_DIR"
        if [[ $? -eq 0 ]]; then
            echo "   [OK] Katalog backup utworzony."
        else
            echo "   [BŁĄD] Nie udało się utworzyć katalogu backup."
            exit 1
        fi
    fi
    
    # 3. STWÓRZ KOPIĘ ZAPASOWĄ Z TIMESTAMPEM
    local timestamp=$(date +"%Y%m%d_%H%M%S")
    local backup_file="$BACKUP_DIR/$(basename "$NGINX_SITE_CONFIG")_${timestamp}.backup"
    
    cp "$NGINX_SITE_CONFIG" "$backup_file"
    
    if [[ $? -eq 0 ]]; then
        echo "   [OK] Utworzono kopię zapasową: $backup_file"
        # Zabezpiecz uprawnienia kopii
        chmod 600 "$backup_file"
    else
        echo "   [BŁĄD] Nie udało się utworzyć kopii zapasowej."
        exit 1
    fi
    echo ""
}

function skonfiguruj_autoryzacje() {
    echo "[3] Konfiguracja HTTP Basic Auth (TYLKO dla frontendu)..."
    
    # --- CZĘŚĆ A: TWORZENIE PLIKU Z HASŁAMI (.htpasswd) ---
    echo "   [3.1] Tworzenie pliku z poświadczeniami: $HTPASSWD_FILE"
    # Użyj htpasswd w trybie batch (-b) z opcją -c (create) aby utworzyć lub nadpisać plik
    htpasswd -bc "$HTPASSWD_FILE" "$AUTH_USER" "$AUTH_PASS" 2>/dev/null
    
    if [[ $? -eq 0 ]]; then
        echo "   [OK] Plik $HTPASSWD_FILE utworzony z użytkownikiem: $AUTH_USER"
        # Zabezpiecz plik - tylko do odczytu dla root
        chmod 600 "$HTPASSWD_FILE"
    else
        echo "   [BŁĄD] Nie udało się utworzyć pliku z hasłami."
        exit 1
    fi
    
    # --- CZĘŚĆ B: MODYFIKACJA KONFIGURACJI NGINX ---
    echo "   [3.2] Modyfikacja konfiguracji Nginx ($NGINX_SITE_CONFIG)..."
    
    # 1. Upewnij się, że Basic Auth jest w 'location /' (frontend)
    if ! grep -q 'auth_basic.*location /' "$NGINX_SITE_CONFIG"; then
        echo "   [INFO] Dodaję Basic Auth do 'location /' (frontend)..."
        sed -i '/location \/.*{/,/}/ {
            /}/i\
    auth_basic "Restricted Access";\
    auth_basic_user_file '"$HTPASSWD_FILE"';
        }' "$NGINX_SITE_CONFIG"
    else
        echo "   [OK] Basic Auth już obecny w 'location /'."
    fi
    
    # 2. USUŃ Basic Auth z 'location /api/' (backend) - jeśli istnieje
    if grep -q 'auth_basic.*location /api/' "$NGINX_SITE_CONFIG"; then
        echo "   [INFO] Usuwam Basic Auth z 'location /api/' (backend)..."
        # Usuń linie zawierające 'auth_basic' wewnątrz bloku 'location /api/'
        sed -i '/location \/api\/.*{/,/}/ {
            /auth_basic/d
        }' "$NGINX_SITE_CONFIG"
        echo "   [OK] Basic Auth usunięty z 'location /api/'."
    else
        echo "   [OK] Basic Auth nieobecny w 'location /api/' (tak jak trzeba)."
    fi
    
    # Ostateczna weryfikacja
    echo "   [INFO] Stan końcowy:"
    echo "     - Frontend (location /): $(grep -q 'auth_basic.*location /' "$NGINX_SITE_CONFIG" && echo 'ZABEZPIECZONY' || echo 'NIEZABEZPIECZONY')"
    echo "     - Backend (location /api/): $(grep -q 'auth_basic.*location /api/' "$NGINX_SITE_CONFIG" && echo 'ZABEZPIECZONY (UWAGA!)' || echo 'OTWARTY (tylko lokalnie)')"
    echo ""
}

function procedura_awaryjna() {
    echo "[!] Uruchomienie procedury awaryjnego wycofania..."
    
    # 1. Znajdź NAJNOWSZĄ kopię zapasową
    local latest_backup=$(ls -t "$BACKUP_DIR"/*.backup 2>/dev/null | head -1)
    
    if [[ -z "$latest_backup" ]]; then
        echo "   [BŁĄD KRYTYCZNY] Nie znaleziono żadnej kopii zapasowej w $BACKUP_DIR."
        echo "   Nie mogę przywrócić konfiguracji. Wymagana interwencja ręczna."
        exit 1
    fi
    
    echo "   [INFO] Przywracam konfigurację z: $latest_backup"
    echo "   [INFO] Do: $NGINX_SITE_CONFIG"
    
    # 2. Przywróć kopię
    cp "$latest_backup" "$NGINX_SITE_CONFIG"
    if [[ $? -ne 0 ]]; then
        echo "   [BŁĄD] Nie udało się przywrócić kopii zapasowej."
        exit 1
    fi
    
    # 3. Usuń plik z hasłami (jeśli istnieje)
    if [[ -f "$HTPASSWD_FILE" ]]; then
        rm -f "$HTPASSWD_FILE"
        echo "   [INFO] Usunięto plik z hasłami: $HTPASSWD_FILE"
    fi
    
    # 4. Przeładuj Nginx, aby wrócić do stanu początkowego
    echo "   [INFO] Przeładowanie Nginx (powrót do oryginalnej konfiguracji)..."
    systemctl reload nginx
    if [[ $? -eq 0 ]]; then
        echo "   [OK] Nginx przeładowany. Konfiguracja przywrócona."
    else
        echo "   [BŁĄD] Nie udało się przeładować Nginx po przywróceniu kopii."
        echo "   Sprawdź stan usługi ręcznie: systemctl status nginx"
    fi
    
    # 5. Zakończ skrypt z błędem
    echo "   [KONIEC] Skrypt zakończony z powodu błędu walidacji. Zmiany wycofane."
    exit 1
}

function waliduj_i_przeladuj() {
    echo "[4] Walidacja i przeładowanie Nginx..."
    
    # 1. Walidacja konfiguracji
    echo "   [4.1] Walidacja konfiguracji (nginx -t)..."
    nginx -t 2>&1
    local validation_result=$?
    
    if [[ $validation_result -eq 0 ]]; then
        echo "   [OK] Konfiguracja Nginx jest poprawna."
        
        # 2. Przeładowanie Nginx
        echo "   [4.2] Przeładowanie usługi Nginx (systemctl reload nginx)..."
        systemctl reload nginx
        if [[ $? -eq 0 ]]; then
            echo "   [OK] Nginx przeładowany pomyślnie."
            echo "   [SUKCES] Konfiguracja Basic Auth (tylko frontend) aktywowana dla $SERVER_IP"
        else
            echo "   [BŁĄD] Nie udało się przeładować Nginx (usługa może mieć problem)."
            echo "   [INFO] Wywołuję procedurę awaryjną..."
            procedura_awaryjna
        fi
    else
        echo "   [BŁĄD] Konfiguracja Nginx jest NIEPOPRAWNA (nginx -t zwrócił błąd)."
        echo "   [INFO] Wywołuję procedurę awaryjną..."
        procedura_awaryjna
    fi
    echo ""
}

# --- LOGIKA GŁÓWNA ---
function main() {
    echo "================================================"
    echo "Rozpoczynam konfigurację Basic Auth dla: $SERVER_IP"
    echo "UŻYCIE: TYLKO frontend (location /). Backend (/api/) otwarty lokalnie."
    echo "Użytkownik: $AUTH_USER"
    echo "Hasło: [ukryte]"
    echo "================================================"
    echo ""
    # KROK 1: Sprawdź narzędzia
    sprawdz_narzędzia
    # KROK 2: Utwórz kopię zapasową konfiguracji
    utworz_kopie_zapasowa
    # KROK 3: Skonfiguruj autoryzację (tylko frontend)
    skonfiguruj_autoryzacje
    # KROK 4: Walidacja i przeładowanie
    waliduj_i_przeladuj
    
    # --- INSTRUKCJA TESTOWA DLA CEO ---
    echo "================================================"
    echo "INSTRUKCJA TESTOWA:"
    echo "1. TEST POZYTYWNY (dostęp do frontendu przyznany):"
    echo "   curl -u 'Agnostyk:Castorama13' http://$SERVER_IP"
    echo ""
    echo "2. TEST NEGATYWNY (dostęp do frontendu zabroniony):"
    echo "   curl -v http://$SERVER_IP"
    echo "   (Oczekiwany kod: 401 Unauthorized)"
    echo ""
    echo "3. TEST BACKENDU (dostęp lokalny, bez Basic Auth):"
    echo "   curl -v http://$SERVER_IP/api/"
    echo "   (Powinno działać, może zwrócić 200 lub błąd aplikacji, ale NIE 401)"
    echo ""
    echo "4. TEST AWARYJNY (przywrócenie konfiguracji):"
    echo "   sudo $0 --rollback"
    echo "================================================"
}

# --- OPCJA AWARYJNEGO WYCOFANIA (dla ręcznego uruchomienia) ---
if [[ $# -gt 0 ]] && [[ "$1" == "--rollback" ]]; then
    echo "RĘCZNE WYWOŁANIE PROCEDURY AWARYJNEGO WYCOFANIA..."
    procedura_awaryjna
fi

# --- WYKONANIE ---
# Zabezpieczenie przed uruchomieniem bez sudo
if [[ $EUID -ne 0 ]]; then
   echo "BŁĄD: Skrypt wymaga uprawnień root (sudo). Uruchom: sudo $0" >&2
   exit 1
fi

main "$@"
