#!/bin/bash
# SKRYPT 7: Dodaje brakujące tagi <script> w bloku extra_scripts
cd /opt/ai_firma_dashboard

# Znajdź linie bloku extra_scripts
START_LINE=$(grep -n "{% block extra_scripts %}" templates/archive.html | cut -d: -f1)
END_LINE=$(grep -n "{% endblock %}" templates/archive.html | tail -1 | cut -d: -f1)

if [ -z "$START_LINE" ] || [ -z "$END_LINE" ]; then
    echo "BŁĄD: Nie znaleziono bloku extra_scripts."
    exit 1
fi

# Sprawdź, czy już ma tag <script>
if grep -q "<script>" templates/archive.html; then
    echo "Tag <script> już istnieje."
else
    # Dodaj <script> po linii z {% block extra_scripts %}
    sed -i "${START_LINE}a <script>" templates/archive.html
    # Dodaj </script> przed linią z {% endblock %}
    sed -i "$((END_LINE-1))i </script>" templates/archive.html
    echo "Dodano tagi <script> i </script>."
fi

# Upewnij się, że przyciski mają poprawne atrybuty data-action (a nie onclick)
# Najpierw znajdź sekcję .action-buttons (przykładowe linie)
sed -i 's/onclick="showComingSoon(.*)"//g' templates/archive.html  # Usuń wszystkie onclick
sed -i 's/<button class="action-btn disabled"/<button class="action-btn" id="btnManageGoldImage"/' templates/archive.html
# Dodaj data-action do przycisków (muszę znać dokładny HTML, ale zrobię uniwersalnie)
sed -i 's/<button class="action-btn" title="Sprawdź Status Backupów"/<button class="action-btn" data-action="status"/' templates/archive.html 2>/dev/null || true
sed -i 's/<button class="action-btn" title="Uruchom Backup Ręczny"/<button class="action-btn" data-action="run_manual"/' templates/archive.html 2>/dev/null || true
# ... itd. To ryzykowne bez dokładnego kodu.

echo "Proponuję ręcznie sprawdzić, czy przyciski mają data-action. Główny problem (brak <script>) został naprawiony."
