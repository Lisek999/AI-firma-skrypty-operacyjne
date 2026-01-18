#!/bin/bash
# SKRYPT: Zastąpienie pliku archive.html nową, działającą wersją
# Dostarczany przez system EDITOR.md

cd /opt/ai_firma_dashboard || exit 1

# Backup obecnego pliku
BACKUP_FILE="templates/archive.html.backup_$(date +%Y%m%d_%H%M%S)"
cp templates/archive.html "$BACKUP_FILE"
echo "Utworzono backup: $BACKUP_FILE"

# Nowa zawartość archive.html (TEN SAM KOD HTML CO WYŻEJ, ALE W JEDNEJ ZMIENNEJ BASH)
NEW_CONTENT='{% extends "layout.html" %}
{% block title %}Archiwum - System Backupów 3-stopniowych{% endblock %}
{% block extra_css %}
<style>
    .archive-container {
        max-width: 1000px;
        margin: 0 auto;
    }
    .section-header {
        background: linear-gradient(135deg, var(--bg-secondary), rgba(45, 51, 59, 0.8));
        border-radius: 12px;
        padding: 2rem;
        margin-bottom: 2rem;
        border: 1px solid var(--border);
        text-align: center;
    }
    .section-header h1 {
        margin-top: 0;
        color: var(--accent);
        font-size: 2.2rem;
    }
    .section-header .subtitle {
        color: var(--text-secondary);
        font-size: 1.1rem;
        max-width: 700px;
        margin: 0 auto;
    }
    .backup-cards {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
        gap: 1.5rem;
        margin-bottom: 2rem;
    }
    .backup-card {
        background: var(--bg-secondary);
        border: 1px solid var(--border);
        border-radius: 10px;
        padding: 1.5rem;
        transition: transform 0.3s ease, border-color 0.3s ease;
    }
    .backup-card:hover {
        transform: translateY(-5px);
        border-color: var(--accent);
    }
    .card-header {
        display: flex;
        align-items: center;
        gap: 1rem;
        margin-bottom: 1rem;
    }
    .card-icon {
        font-size: 2.5rem;
        width: 60px;
        height: 60px;
        background: rgba(83, 155, 245, 0.1);
        border-radius: 50%;
        display: flex;
        align-items: center;
        justify-content: center;
    }
    .card-title {
        font-size: 1.3rem;
        font-weight: bold;
        margin: 0;
    }
    .card-status {
        display: inline-block;
        padding: 0.3rem 0.8rem;
        border-radius: 20px;
        font-size: 0.85rem;
        font-weight: bold;
        margin-top: 0.5rem;
    }
    .status-planned {
        background: rgba(218, 133, 67, 0.2);
        color: #da8543;
    }
    .card-description {
        color: var(--text-secondary);
        margin: 1rem 0;
        line-height: 1.5;
    }
    .card-features {
        list-style: none;
        padding: 0;
        margin: 1rem 0;
    }
    .card-features li {
        padding: 0.3rem 0;
        color: var(--text-secondary);
        display: flex;
        align-items: center;
        gap: 0.5rem;
    }
    .card-features li:before {
        content: "→";
        color: var(--accent);
    }
    .info-panel {
        background: rgba(83, 155, 245, 0.1);
        border: 1px solid var(--accent);
        border-radius: 10px;
        padding: 1.5rem;
        margin: 2rem 0;
    }
    .info-panel h3 {
        margin-top: 0;
        color: var(--accent);
        display: flex;
        align-items: center;
        gap: 0.5rem;
    }
    .action-buttons {
        display: flex;
        flex-wrap: wrap;
        gap: 1rem;
        margin: 2rem 0;
        justify-content: center;
    }
    .action-btn {
        background: var(--bg-secondary);
        border: 2px solid var(--accent);
        color: var(--accent);
        padding: 0.8rem 1.5rem;
        border-radius: 8px;
        font-weight: bold;
        cursor: pointer;
        transition: all 0.3s ease;
        display: flex;
        align-items: center;
        gap: 0.5rem;
    }
    .action-btn:hover {
        background: var(--accent);
        color: white;
    }
    .action-btn.disabled {
        opacity: 0.5;
        cursor: not-allowed;
        border-color: var(--text-secondary);
        color: var(--text-secondary);
    }
    .action-btn.disabled:hover {
        background: var(--bg-secondary);
        color: var(--text-secondary);
    }
    .roadmap {
        background: var(--bg-secondary);
        border-radius: 10px;
        padding: 1.5rem;
        margin: 2rem 0;
        border: 1px solid var(--border);
    }
    .roadmap-item {
        display: flex;
        align-items: flex-start;
        gap: 1rem;
        margin: 1rem 0;
        padding: 1rem;
        background: rgba(0, 0, 0, 0.2);
        border-radius: 8px;
    }
    .roadmap-checkbox {
        width: 24px;
        height: 24px;
        border: 2px solid var(--border);
        border-radius: 50%;
        flex-shrink: 0;
        margin-top: 0.2rem;
    }
    .roadmap-checkbox.completed {
        background: var(--success);
        border-color: var(--success);
        position: relative;
    }
    .roadmap-checkbox.completed:after {
        content: "✓";
        color: white;
        position: absolute;
        top: 50%;
        left: 50%;
        transform: translate(-50%, -50%);
        font-size: 0.8rem;
    }
    @media (max-width: 768px) {
        .backup-cards { grid-template-columns: 1fr; }
        .section-header { padding: 1.5rem; }
        .action-buttons { flex-direction: column; }
        .action-btn { width: 100%; justify-content: center; }
    }
</style>
{% endblock %}
{% block content %}
<div class="archive-container">
    <div class="section-header">
        <h1>🗃️ System Archiwizacji 3-stopniowej</h1>
        <p class="subtitle">Kompleksowy system backupów: codzienne przyrostowe, tygodniowe pełne i strategiczne Gold Image dla szybkiego odzyskiwania systemu.</p>
    </div>
    <div class="backup-cards">
        <div class="backup-card">
            <div class="card-header">
                <div class="card-icon">📅</div>
                <div><h3 class="card-title">Backup Codzienny</h3><span class="card-status status-planned">W TRAKCIE WDRAŻANIA</span></div>
            </div>
            <p class="card-description">Automatyczne, przyrostowe kopie zapasowe skryptów, konfiguracji i danych. Wykonywane każdej nocy o 02:00.</p>
            <ul class="card-features"><li>Przyrostowe kopie (zmiany od ostatniego backupu)</li><li>Automatyczne czyszczenie starych backupów</li><li>Szyfrowanie danych wrażliwych</li><li>Powiadomienia o statusie</li></ul>
        </div>
        <div class="backup-card">
            <div class="card-header">
                <div class="card-icon">🗓️</div>
                <div><h3 class="card-title">Backup Tygodniowy</h3><span class="card-status status-planned">W TRAKCIE WDRAŻANIA</span></div>
            </div>
            <p class="card-description">Pełne archiwum całego systemu. Tworzone w każdą niedzielę, przechowywane przez 4 tygodnie.</p>
            <ul class="card-features"><li>Kompletna kopia systemu</li><li>Kompresja i optymalizacja</li><li>Weryfikacja integralności</li><li>Przechowywanie wielowariantowe</li></ul>
        </div>
        <div class="backup-card">
            <div class="card-header">
                <div class="card-icon">👑</div>
                <div><h3 class="card-title">Gold Image</h3><span class="card-status status-planned">W TRAKCIE WDRAŻANIA</span></div>
            </div>
            <p class="card-description">Strategiczy obraz systemu w idealnym, przetestowanym stanie. Gotowy do natychmiastowego wdrożenia w razie awarii.</p>
            <ul class="card-features"><li>Zweryfikowany stan systemu</li><li>Automatyczne testy po utworzeniu</li><li>Szybkie wdrożenie (< 10 minut)</li><li>Wersjonowanie i historia</li></ul>
        </div>
    </div>
    <div class="info-panel">
        <h3>ℹ️ Informacja o statusie</h3>
        <p><strong>System backupów 3-stopniowych jest w trakcie wdrażania.</strong> Obecna strona stanowi kompletną obudowę interfejsu użytkownika.</p>
        <p>Funkcjonalność backendowa została zaimplementowana - endpointy API są aktywne. Przyciski poniżej wykonują testowe połączenia z backendem.</p>
    </div>
    <div class="action-buttons">
        <button class="action-btn" data-action="status"><span>📊</span>Sprawdź Status Backupów</button>
        <button class="action-btn" data-action="run_manual"><span>🔄</span>Uruchom Backup Ręczny</button>
        <button class="action-btn" id="btnManageGoldImage"><span>👑</span>Zarządzaj Gold Image</button>
        <button class="action-btn" data-action="restore"><span>⚡</span>Przywróć z Backupu</button>
        <button class="action-btn" data-action="configure"><span>⚙️</span>Konfiguruj System Backupów</button>
    </div>
    <div class="roadmap">
        <h3 style="margin-top: 0; color: var(--accent);">🛣️ Plan rozwoju systemu</h3>
        <div class="roadmap-item"><div class="roadmap-checkbox completed"></div><div><strong>Etap 1: Obudowa interfejsu</strong><p style="margin: 0.3rem 0 0 0; color: var(--text-secondary);">Stworzenie kompletnego UI z nawigacją i strukturą - <strong>ZAKOŃCZONY</strong></p></div></div>
        <div class="roadmap-item"><div class="roadmap-checkbox"></div><div><strong>Etap 2: Integracja z istniejącymi skryptami</strong><p style="margin: 0.3rem 0 0 0; color: var(--text-secondary);">Podłączenie endpointów API do skryptów backupów w <code>/home/ubuntu/skrypty/</code></p></div></div>
        <div class="roadmap-item"><div class="roadmap-checkbox"></div><div><strong>Etap 3: System powiadomień i monitoringu</strong><p style="margin: 0.3rem 0 0 0; color: var(--text-secondary);">Real-time status, alerty, logi i raporty</p></div></div>
        <div class="roadmap-item"><div class="roadmap-checkbox"></div><div><strong>Etap 4: Zaawansowane funkcje</strong><p style="margin: 0.3rem 0 0 0; color: var(--text-secondary);">Szyfrowanie, wersjonowanie, automatyczne testy backupów</p></div></div>
    </div>
    <div style="text-align: center; margin-top: 3rem; padding-top: 2rem; border-top: 1px solid var(--border);">
        <a href="{{ url_for('index') }}" style="color: var(--accent); text-decoration: none; font-weight: bold; display: inline-flex; align-items: center; gap: 0.5rem; padding: 0.8rem 1.5rem; border: 1px solid var(--accent); border-radius: 6px; transition: all 0.3s ease;"><span>←</span>Powrót do Centrum Kontroli</a>
        <p style="color: var(--text-secondary); margin-top: 1rem; font-size: 0.9rem;">System backupów będzie rozwijany w kolejnych sesjach zgodnie z Planem Ataku</p>
    </div>
</div>
{% endblock %}
{% block extra_scripts %}
<script>
    document.addEventListener("DOMContentLoaded", function() {
        const endpoints = {
            status: { url: "/api/backup/status", method: "GET" },
            run_manual: { url: "/api/backup/run_manual", method: "POST" },
            restore: { url: "/api/backup/restore", method: "POST" },
            configure: { url: "/api/backup/configure", method: "POST" },
            manage_gold: { url: "/api/gold_image/manage", method: "POST" }
        };
        async function handleButtonClick(action, event) {
            const endpoint = endpoints[action];
            if (!endpoint) return;
            const button = event.target.closest(".action-btn");
            const originalText = button.innerHTML;
            button.innerHTML = "⏳ Przetwarzanie...";
            button.disabled = true;
            try {
                const response = await fetch(endpoint.url, { method: endpoint.method });
                const result = await response.json();
                alert("[" + (response.ok ? "SUCCESS" : "ERROR") + "] " + result.message);
            } catch (error) {
                alert("[ERROR] Błąd połączenia: " + error.message);
            } finally {
                button.innerHTML = originalText;
                button.disabled = false;
            }
        }
        document.querySelectorAll(".action-btn[data-action]").forEach(btn => {
            btn.addEventListener("click", (e) => handleButtonClick(btn.dataset.action, e));
        });
        document.getElementById("btnManageGoldImage").addEventListener("click", (e) => {
            handleButtonClick("manage_gold", e);
        });
    });
</script>
{% endblock %}'

# Zapisanie nowej zawartości do pliku
echo "$NEW_CONTENT" > templates/archive.html

echo "Plik archive.html został zastąpiony nową, działającą wersją."
echo "Odśwież stronę /archive i przetestuj przyciski."
