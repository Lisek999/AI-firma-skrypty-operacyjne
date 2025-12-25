#!/bin/bash

# ============================================================================
# create_gold_image.sh - DEFINITYWNA WERSJA FINALNA (OSTATECZNA)
# Gold Image Creator - Faza 1 Stabilna
# 
# OSTATECZNE POPRAWKI:
# 1. awk zamiast sed - zmienia tylko pierwsze wystąpienie
# 2. Logi przed pobraniem hasha - czysty output
# 3. Pełna obsługa istniejącego taga
# ============================================================================

# --- KONFIGURACJA ---
REPO_ROOT="/home/ubuntu/ai_firma_dokumenty"
BACKUP_DIR="${REPO_ROOT}/gold_image_v1.0"
REPORT_FILE="${REPO_ROOT}/GOLD_IMAGE_v1.0.md"
TAG_NAME="v1.0-stable"
COMMIT_MSG="Gold Image - Faza 1 Stabilna"
UPDATE_MSG="Update report with final commit hash"
CURRENT_DATE=$(date '+%Y-%m-%d %H:%M:%S')

# --- TABLICA PLIKÓW (PRZYKŁADOWA - DO ZASTĄPIENIA RZECZYWISTĄ LISTĄ) ---
declare -a FILES_TO_BACKUP=(
    "/etc/nginx/nginx.conf"
    "/home/ubuntu/.bashrc"
    "/opt/moja_aplikacja/config.yaml"
    "/var/www/html/index.php"
)

# --- FUNKCJE POMOCNICZE ---
log_info() {
    echo "[INFO] ${CURRENT_DATE} - $1"
}

log_warning() {
    echo "[WARNING] ${CURRENT_DATE} - $1" >&2
}

log_error() {
    echo "[ERROR] ${CURRENT_DATE} - $1" >&2
    exit 1
}

# --- WALIDACJA WCZEŚNIEJSZA (pkt 3.3) ---
validate_git_status() {
    log_info "Sprawdzanie stanu repozytorium Git..."
    
    if ! git rev-parse --git-dir > /dev/null 2>&1; then
        log_error "Brak repozytorium Git w bieżącym katalogu!"
    fi
    
    local git_changes=$(git status --porcelain | grep -v '^?? gold_image_v1.0/')
    
    if [[ -n "${git_changes}" ]]; then
        echo "================================================"
        echo "BŁĄD: Repozytorium ma niezcommitowane zmiany!"
        echo "Zmiany wykryte:"
        echo "${git_changes}"
        echo ""
        echo "Proszę wykonaj:"
        echo "  git status               # zobacz zmiany"
        echo "  git add .                # dodaj wszystko"
        echo "  git commit -m 'message'  # zcommituj"
        echo "lub:"
        echo "  git stash                # schowaj zmiany tymczasowo"
        echo "================================================"
        exit 1
    fi
    
    log_info "Repozytorium Git jest czyste - można kontynuować."
}

# --- SPRAWDZENIE TAGA ---
check_existing_tag() {
    if git rev-parse "${TAG_NAME}" >/dev/null 2>&1; then
        return 0
    fi
    
    if git ls-remote --tags origin "refs/tags/${TAG_NAME}" | grep -q "${TAG_NAME}"; then
        return 0
    fi
    
    return 1
}

# --- PRZYGOTOWANIE KATALOGU (pkt 3.4) ---
prepare_backup_directory() {
    log_info "Przygotowywanie katalogu backup: ${BACKUP_DIR}"
    
    if [[ -d "${BACKUP_DIR}" ]]; then
        rm -rf "${BACKUP_DIR}"
        log_info "Usunięto stary katalog backup."
    fi
    
    mkdir -p "${BACKUP_DIR}"
    log_info "Utworzono nowy katalog backup."
}

# --- KOPIOWANIE PLIKÓW (pkt 3.4) ---
copy_files_to_backup() {
    local copied_count=0
    local missing_count=0
    
    log_info "Rozpoczynanie kopiowania plików..."
    echo "================================================"
    
    for source_file in "${FILES_TO_BACKUP[@]}"; do
        if [[ ! -f "${source_file}" ]] && [[ ! -d "${source_file}" ]]; then
            log_warning "❌ Plik nie istnieje: ${source_file}"
            ((missing_count++))
            continue
        fi
        
        local target_file="${BACKUP_DIR}${source_file}"
        local target_dir=$(dirname "${target_file}")
        
        mkdir -p "${target_dir}"
        
        if cp -r "${source_file}" "${target_file}" 2>/dev/null; then
            log_info "✅ Skopiowano: ${source_file}"
            ((copied_count++))
        else
            log_warning "⚠️  Błąd kopiowania: ${source_file}"
        fi
    done
    
    echo "================================================"
    echo "📊 PODSUMOWANIE KOPIOWANIA:"
    echo "   ✅ Skopiowano plików: ${copied_count}"
    echo "   ⚠️  Pominięto plików: ${missing_count}"
    echo "================================================"
}

# --- TWORZENIE RAPORTU (pkt 3.5) ---
create_initial_report() {
    log_info "Tworzenie wstępnego raportu..."
    
    cat > "${REPORT_FILE}" << EOF
# GOLD IMAGE - v1.0-stable
## Raport wykonania zrzutu systemowego

**Data utworzenia:** ${CURRENT_DATE}
**Tag:** ${TAG_NAME}
**Commit hash:** [PENDING - zostanie uzupełniony po commicie]

---

## Lista skopiowanych plików:

EOF
    
    for source_file in "${FILES_TO_BACKUP[@]}"; do
        local target_file="${BACKUP_DIR}${source_file}"
        if [[ -e "${target_file}" ]]; then
            echo "- ${source_file}" >> "${REPORT_FILE}"
        fi
    done
    
    cat >> "${REPORT_FILE}" << EOF

---

## Instrukcje przywracania:

Aby przywrócić pojedynczy plik:
\`\`\`bash
git checkout ${TAG_NAME} -- gold_image_v1.0/ścieżka/do/pliku
\`\`\`

Przykład dla nginx.conf:
\`\`\`bash
git checkout ${TAG_NAME} -- gold_image_v1.0/etc/nginx/nginx.conf
\`\`\`

---

**Uwaga:** Ten raport zostanie zaktualizowany o finalny hash commita po tagowaniu.
EOF
    
    log_info "Utworzono wstępny raport: ${REPORT_FILE}"
}

# --- OPERACJE GIT (pkt 3.6) - WYPERFEKCJONOWANA ---
perform_git_operations() {
    log_info "Rozpoczynanie operacji Git..."
    
    cd "${REPO_ROOT}" > /dev/null 2>&1
    
    # 1. Dodanie plików
    git add .
    
    # 2. Commit
    git commit -m "${COMMIT_MSG}"
    
    # 3. Pobranie hasha (BEZ LOGÓW MIĘDZY)
    local commit_hash=$(git rev-parse HEAD)
    
    # 4. Log info o commicie (PO pobraniu hasha)
    log_info "Utworzono commit: ${commit_hash}"
    
    # 5. Obsługa taga
    if check_existing_tag; then
        log_warning "Tag ${TAG_NAME} już istnieje - pomijam tworzenie taga."
    else
        git tag "${TAG_NAME}"
        log_info "Utworzono lokalny tag: ${TAG_NAME}"
        git push origin "${TAG_NAME}"
        log_info "Wypchnięto tag do zdalnego repozytorium."
    fi
    
    # 6. Zwrócenie CZYSTEGO hasha
    echo "${commit_hash}"
}

# --- AKTUALIZACJA RAPORTU (pkt 3.8) - WYPERFEKCJONOWANA ---
update_report_with_final_hash() {
    local final_hash="$1"
    
    log_info "Aktualizowanie raportu o finalny hash..."
    
    # Użycie awk zamiast sed - zmienia tylko PIERWSZE wystąpienie
    awk -v hash="${final_hash}" '
    /\*\*Commit hash:\*\* \[PENDING/ && !found {
        sub(/\[PENDING - zostanie uzupełniony po commicie\]/, hash)
        found=1
    }
    {print}
    ' "${REPORT_FILE}" > "${REPORT_FILE}.tmp" && mv "${REPORT_FILE}.tmp" "${REPORT_FILE}"
    
    # Dodanie linku do GitHub
    cat >> "${REPORT_FILE}" << EOF

---

## Link do repozytorium:

\`\`\`
https://github.com/Lisek999/ai-firma-vps/releases/tag/${TAG_NAME}
\`\`\`

**Finalny hash commita:** \`${final_hash}\`
EOF
    
    # Commit i push
    git add "${REPORT_FILE}"
    git commit -m "${UPDATE_MSG}"
    git push origin main
    
    log_info "Zaktualizowano raport i wypchnięto zmiany."
}

# --- PODSUMOWANIE (pkt 3.9) ---
print_summary() {
    local final_hash="$1"
    local copied_count=0
    
    for source_file in "${FILES_TO_BACKUP[@]}"; do
        local target_file="${BACKUP_DIR}${source_file}"
        if [[ -e "${target_file}" ]]; then
            ((copied_count++))
        fi
    done
    
    echo ""
    echo "================================================"
    echo "🎉 GOLD IMAGE v1.0-stable UTWORZONY POMYŚLNIE!"
    echo "================================================"
    echo ""
    echo "📊 PODSUMOWANIE:"
    echo "   • Skopiowanych plików: ${copied_count}"
    echo "   • Tag: ${TAG_NAME}"
    echo "   • Hash commita: ${final_hash}"
    echo "   • Katalog backup: ${BACKUP_DIR}"
    echo "   • Raport: ${REPORT_FILE}"
    echo ""
    echo "🔗 LINK DO ZDALNEGO REPOZYTORIUM:"
    echo "   https://github.com/Lisek999/ai-firma-vps/releases/tag/${TAG_NAME}"
    echo ""
    echo "✅ WERYFIKACJA:"
    echo "   1. git tag -l | grep v1.0-stable"
    echo "   2. Sprawdź tag na GitHubie"
    echo "   3. cat GOLD_IMAGE_v1.0.md"
    echo "================================================"
}

# --- GŁÓWNA FUNKCJA WYKONAWCZA ---
main() {
    log_info "Rozpoczynanie tworzenia Gold Image v1.0-stable..."
    log_info "Repozytorium: ${REPO_ROOT}"
    
    validate_git_status
    prepare_backup_directory
    copy_files_to_backup
    create_initial_report
    final_hash=$(perform_git_operations)
    update_report_with_final_hash "${final_hash}"
    print_summary "${final_hash}"
    
    log_info "Gold Image creation completed successfully!"
}

# --- URUCHOMIENIE ---
if [[ "${BASH_SOURCE[0]}" == "${0}" ]]; then
    main
fi
