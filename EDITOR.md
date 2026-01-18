#!/bin/bash
# ============================================================================
# SKRYPT: tworzenie_index_html.sh
# CEL: Stworzenie strony głównej z 6 kafelkami nawigacyjnymi
# ============================================================================

echo "=== TWORZĘ INDEX.HTML ==="

cd /opt/ai_firma_dashboard || exit 1

cat > templates/index.html << 'EOF'
{% extends "layout.html" %}

{% block title %}Centrum Kontroli - AI Firma{% endblock %}

{% block extra_css %}
<style>
    /* ===== KAFELKI NAWIGACYJNE ===== */
    .dashboard-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
        gap: 1.5rem;
        margin-top: 2rem;
    }
    
    .tile {
        background: var(--bg-secondary);
        border: 1px solid var(--border);
        border-radius: 10px;
        padding: 1.5rem;
        text-decoration: none;
        color: var(--text-primary);
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;
        text-align: center;
        transition: all 0.3s ease;
        min-height: 180px;
        position: relative;
        overflow: hidden;
    }
    
    .tile:hover {
        transform: translateY(-5px);
        border-color: var(--accent);
        box-shadow: 0 10px 20px rgba(0, 0, 0, 0.2);
    }
    
    .tile-icon {
        font-size: 2.5rem;
        margin-bottom: 1rem;
        color: var(--accent);
    }
    
    .tile-title {
        font-size: 1.2rem;
        font-weight: 600;
        margin-bottom: 0.5rem;
    }
    
    .tile-description {
        color: var(--text-secondary);
        font-size: 0.9rem;
        line-height: 1.4;
    }
    
    .tile-badge {
        position: absolute;
        top: 10px;
        right: 10px;
        background: var(--accent);
        color: white;
        font-size: 0.7rem;
        padding: 2px 8px;
        border-radius: 10px;
    }
    
    /* Specjalne style dla kluczowych kafelków */
    .tile-archive {
        border-left: 4px solid #57ab5a;
    }
    
    .tile-terminal {
        border-left: 4px solid #da8543;
    }
    
    .tile-critical {
        border-left: 4px solid #e5534b;
    }
    
    /* Responsywność */
    @media (max-width: 768px) {
        .dashboard-grid {
            grid-template-columns: repeat(2, 1fr);
            gap: 1rem;
        }
        
        .tile {
            min-height: 150px;
            padding: 1rem;
        }
    }
    
    @media (max-width: 480px) {
        .dashboard-grid {
            grid-template-columns: 1fr;
        }
    }
</style>
{% endblock %}

{% block content %}
<div class="dashboard-header">
    <h1 style="margin-bottom: 0.5rem;">Centrum Kontroli AI Firma</h1>
    <p style="color: var(--text-secondary); margin-bottom: 2rem;">
        Zintegrowany panel zarządzania systemem i usługami
    </p>
</div>

<div class="dashboard-grid">
    <!-- ARCHIWUM (priorytet) -->
    <a href="{{ url_for('archive') }}" class="tile tile-archive">
        <div class="tile-icon">📁</div>
        <div class="tile-title">ARCHIWUM</div>
        <div class="tile-description">
            Zarządzanie backupami, Gold Image, przywracanie systemu
        </div>
        <span class="tile-badge">KRYTYCZNE</span>
    </a>
    
    <!-- TERMINAL -->
    <a href="{{ url_for('terminal') }}" class="tile tile-terminal">
        <div class="tile-icon">💻</div>
        <div class="tile-title">TERMINAL</div>
        <div class="tile-description">
            Dostęp do konsoli, wykonywanie skryptów, monitoring procesów
        </div>
    </a>
    
    <!-- EKSPLORER -->
    <a href="{{ url_for('explorer') }}" class="tile">
        <div class="tile-icon">📂</div>
        <div class="tile-title">EKSPLORER</div>
        <div class="tile-description">
            Przeglądarka plików, zarządzanie zasobami, upload/download
        </div>
    </a>
    
    <!-- KONFIGURACJA -->
    <a href="{{ url_for('config') }}" class="tile">
        <div class="tile-icon">⚙️</div>
        <div class="tile-title">KONFIGURACJA</div>
        <div class="tile-description">
            Ustawienia systemu, zmiana parametrów, zarządzanie usługami
        </div>
    </a>
    
    <!-- PULS SYSTEMU -->
    <a href="{{ url_for('pulse') }}" class="tile tile-critical">
        <div class="tile-icon">📈</div>
        <div class="tile-title">PULS SYSTEMU</div>
        <div class="tile-description">
            Monitoring w czasie rzeczywistym, metryki, alerty, wykresy
        </div>
        <span class="tile-badge">LIVE</span>
    </a>
    
    <!-- NARZĘDZIA -->
    <a href="{{ url_for('tools') }}" class="tile">
        <div class="tile-icon">🔧</div>
        <div class="tile-title">NARZĘDZIA</div>
        <div class="tile-description">
            Narzędzia diagnostyczne, szybkie akcje, automatyzacje
        </div>
    </a>
</div>

<div style="margin-top: 3rem; padding: 1.5rem; background: var(--bg-secondary); border-radius: 8px; border: 1px solid var(--border);">
    <h3 style="margin-top: 0;">📊 Status Systemu</h3>
    <div style="display: grid; grid-template-columns: repeat(auto-fit, minmax(200px, 1fr)); gap: 1rem; margin-top: 1rem;">
        <div>
            <strong>Usługi:</strong>
            <div style="margin-top: 0.5rem;">
                <span style="display: inline-flex; align-items: center; gap: 0.5rem;">
                    <span style="display: inline-block; width: 10px; height: 10px; border-radius: 50%; background: #57ab5a;"></span>
                    Nginx: działą
                </span>
            </div>
            <div>
                <span style="display: inline-flex; align-items: center; gap: 0.5rem;">
                    <span style="display: inline-block; width: 10px; height: 10px; border-radius: 50%; background: #57ab5a;"></span>
                    Supervisor: działą
                </span>
            </div>
        </div>
        <div>
            <strong>Ostatnia aktualizacja:</strong>
            <div style="margin-top: 0.5rem; font-family: monospace;" id="status-update-time">
                {{ status_message }}
            </div>
        </div>
    </div>
</div>
{% endblock %}

{% block extra_scripts %}
<script>
    // Automatyczne odświeżanie czasu
    function updateStatusTime() {
        const now = new Date();
        const timeStr = now.toLocaleTimeString('pl-PL');
        const dateStr = now.toLocaleDateString('pl-PL');
        document.getElementById('status-update-time').textContent = 
            `${dateStr} ${timeStr} - ${document.getElementById('status-text').textContent}`;
    }
    
    // Odświeżaj co 30 sekund
    updateStatusTime();
    setInterval(updateStatusTime, 30000);
    
    // Animacja ładowania kafelków
    document.addEventListener('DOMContentLoaded', function() {
        const tiles = document.querySelectorAll('.tile');
        tiles.forEach((tile, index) => {
            tile.style.opacity = '0';
            tile.style.transform = 'translateY(20px)';
            
            setTimeout(() => {
                tile.style.transition = 'opacity 0.5s ease, transform 0.5s ease';
                tile.style.opacity = '1';
                tile.style.transform = 'translateY(0)';
            }, index * 100);
        });
    });
</script>
{% endblock %}
EOF

echo "✅ Utworzono templates/index.html"
echo ""
echo "=== PODGLĄD STRUKTURY ==="
ls -la templates/
echo ""
echo "=== WERYFIKACJA ==="
head -20 templates/index.html
echo "..."
echo "=== ROZMIAR PLIKU ==="
wc -l templates/index.html
echo ""
echo "✅ INDEX.HTML GOTOWY"
echo ""
echo "=== NASTĘPNY KROK ==="
echo "Tworzenie archive.html z istniejącej funkcjonalności backupów"
