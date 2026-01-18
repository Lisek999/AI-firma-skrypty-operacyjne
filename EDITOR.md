#!/bin/bash
# ============================================================================
# SKRYPT: tworzenie_placeholder_szablonow.sh
# CEL: Stworzenie 5 placeholderowych szablonów dla pozostałych kafelków
# ============================================================================

echo "=== TWORZĘ 5 PLACEHOLDEROWYCH SZABLONÓW ==="

cd /opt/ai_firma_dashboard || exit 1

# 1. TERMINAL
cat > templates/terminal.html << 'EOF'
{% extends "layout.html" %}

{% block title %}Terminal - Konsola Systemowa{% endblock %}

{% block extra_css %}
<style>
    .terminal-container {
        max-width: 1000px;
        margin: 0 auto;
    }
    
    .placeholder-hero {
        text-align: center;
        padding: 4rem 2rem;
        background: linear-gradient(135deg, rgba(45, 51, 59, 0.8), rgba(26, 29, 35, 0.9));
        border-radius: 15px;
        margin-bottom: 2rem;
        border: 1px solid var(--border);
    }
    
    .placeholder-icon {
        font-size: 4rem;
        margin-bottom: 1.5rem;
        display: inline-block;
        background: rgba(83, 155, 245, 0.1);
        width: 120px;
        height: 120px;
        line-height: 120px;
        border-radius: 50%;
    }
    
    .placeholder-title {
        font-size: 2rem;
        color: var(--accent);
        margin-bottom: 1rem;
    }
    
    .placeholder-description {
        color: var(--text-secondary);
        max-width: 600px;
        margin: 0 auto 2rem;
        line-height: 1.6;
    }
    
    .features-grid {
        display: grid;
        grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
        gap: 1.5rem;
        margin: 2rem 0;
    }
    
    .feature-card {
        background: var(--bg-secondary);
        border: 1px solid var(--border);
        border-radius: 10px;
        padding: 1.5rem;
        text-align: center;
    }
    
    .feature-icon {
        font-size: 2rem;
        margin-bottom: 1rem;
        color: var(--accent);
    }
    
    .coming-soon-badge {
        display: inline-block;
        background: rgba(218, 133, 67, 0.2);
        color: #da8543;
        padding: 0.5rem 1rem;
        border-radius: 20px;
        font-weight: bold;
        margin: 1rem 0;
    }
</style>
{% endblock %}

{% block content %}
<div class="terminal-container">
    <div class="placeholder-hero">
        <div class="placeholder-icon">💻</div>
        <h1 class="placeholder-title">Terminal Systemowy</h1>
        <p class="placeholder-description">
            Bezpieczny dostęp do konsoli serwera, wykonywanie komend, monitorowanie procesów
            i zarządzanie usługami bezpośrednio z przeglądarki.
        </p>
        <div class="coming-soon-badge">W TRAKCIE ROZWOJU</div>
    </div>
    
    <div class="features-grid">
        <div class="feature-card">
            <div class="feature-icon">⌨️</div>
            <h3>Konsola w przeglądarce</h3>
            <p>Pełnoprawny terminal z historią komend i autouzupełnianiem</p>
        </div>
        
        <div class="feature-card">
            <div class="feature-icon">📊</div>
            <h3>Monitorowanie procesów</h3>
            <p>Live view uruchomionych procesów, zużycia CPU i pamięci</p>
        </div>
        
        <div class="feature-card">
            <div class="feature-icon">⚡</div>
            <h3>Szybkie akcje</h3>
            <p>Predefiniowane skrypty i często używane komendy</p>
        </div>
    </div>
    
    <div style="text-align: center; margin-top: 3rem;">
        <a href="{{ url_for('index') }}" style="
            color: var(--accent);
            text-decoration: none;
            font-weight: bold;
            display: inline-flex;
            align-items: center;
            gap: 0.5rem;
            padding: 0.8rem 1.5rem;
            border: 1px solid var(--accent);
            border-radius: 6px;
        ">
            ← Powrót do Centrum Kontroli
        </a>
    </div>
</div>
{% endblock %}
EOF
echo "✅ 1/5: templates/terminal.html"

# 2. EKSPLORER
cat > templates/explorer.html << 'EOF'
{% extends "layout.html" %}

{% block title %}Eksplorator - Przeglądarka Plików{% endblock %}

{% block content %}
<div style="max-width: 1000px; margin: 0 auto;">
    <div style="text-align: center; padding: 4rem 2rem;">
        <div style="font-size: 4rem; margin-bottom: 1.5rem;">📂</div>
        <h1 style="font-size: 2rem; color: var(--accent);">Eksplorator Plików</h1>
        <p style="color: var(--text-secondary); max-width: 600px; margin: 0 auto 2rem;">
            Zaawansowana przeglądarka plików serwera z możliwością przeglądania, 
            edycji, uploadu i downloadu plików bezpośrednio przez przeglądarkę.
        </p>
        <div style="display: inline-block; background: rgba(218, 133, 67, 0.2); color: #da8543; 
             padding: 0.5rem 1rem; border-radius: 20px; font-weight: bold;">
            W TRAKCIE ROZWOJU
        </div>
    </div>
    
    <div style="text-align: center; margin-top: 3rem;">
        <a href="{{ url_for('index') }}" style="
            color: var(--accent);
            text-decoration: none;
            font-weight: bold;
            display: inline-flex;
            align-items: center;
            gap: 0.5rem;
            padding: 0.8rem 1.5rem;
            border: 1px solid var(--accent);
            border-radius: 6px;
        ">
            ← Powrót do Centrum Kontroli
        </a>
    </div>
</div>
{% endblock %}
EOF
echo "✅ 2/5: templates/explorer.html"

# 3. KONFIGURACJA
cat > templates/config.html << 'EOF'
{% extends "layout.html" %}

{% block title %}Konfiguracja - Ustawienia Systemu{% endblock %}

{% block content %}
<div style="max-width: 1000px; margin: 0 auto;">
    <div style="text-align: center; padding: 4rem 2rem;">
        <div style="font-size: 4rem; margin-bottom: 1.5rem;">⚙️</div>
        <h1 style="font-size: 2rem; color: var(--accent);">Konfiguracja Systemu</h1>
        <p style="color: var(--text-secondary); max-width: 600px; margin: 0 auto 2rem;">
            Centralny panel zarządzania ustawieniami systemu, usługami, użytkownikami
            i parametrami działania całej platformy AI Firma.
        </p>
        <div style="display: inline-block; background: rgba(218, 133, 67, 0.2); color: #da8543; 
             padding: 0.5rem 1rem; border-radius: 20px; font-weight: bold;">
            W TRAKCIE ROZWOJU
        </div>
    </div>
    
    <div style="text-align: center; margin-top: 3rem;">
        <a href="{{ url_for('index') }}" style="
            color: var(--accent);
            text-decoration: none;
            font-weight: bold;
            display: inline-flex;
            align-items: center;
            gap: 0.5rem;
            padding: 0.8rem 1.5rem;
            border: 1px solid var(--accent);
            border-radius: 6px;
        ">
            ← Powrót do Centrum Kontroli
        </a>
    </div>
</div>
{% endblock %}
EOF
echo "✅ 3/5: templates/config.html"

# 4. PULS SYSTEMU
cat > templates/pulse.html << 'EOF'
{% extends "layout.html" %}

{% block title %}Puls Systemu - Monitoring{% endblock %}

{% block content %}
<div style="max-width: 1000px; margin: 0 auto;">
    <div style="text-align: center; padding: 4rem 2rem;">
        <div style="font-size: 4rem; margin-bottom: 1.5rem;">📈</div>
        <h1 style="font-size: 2rem; color: var(--accent);">Puls Systemu</h1>
        <p style="color: var(--text-secondary); max-width: 600px; margin: 0 auto 2rem;">
            Monitoring czasu rzeczywistego, metryki wydajności, wykresy wykorzystania zasobów
            i system alertów dla krytycznych parametrów systemu.
        </p>
        <div style="display: inline-block; background: rgba(218, 133, 67, 0.2); color: #da8543; 
             padding: 0.5rem 1rem; border-radius: 20px; font-weight: bold;">
            W TRAKCIE ROZWOJU
        </div>
    </div>
    
    <div style="text-align: center; margin-top: 3rem;">
        <a href="{{ url_for('index') }}" style="
            color: var(--accent);
            text-decoration: none;
            font-weight: bold;
            display: inline-flex;
            align-items: center;
            gap: 0.5rem;
            padding: 0.8rem 1.5rem;
            border: 1px solid var(--accent);
            border-radius: 6px;
        ">
            ← Powrót do Centrum Kontroli
        </a>
    </div>
</div>
{% endblock %}
EOF
echo "✅ 4/5: templates/pulse.html"

# 5. NARZĘDZIA
cat > templates/tools.html << 'EOF'
{% extends "layout.html" %}

{% block title %}Narzędzia - Utilitki Systemowe{% endblock %}

{% block content %}
<div style="max-width: 1000px; margin: 0 auto;">
    <div style="text-align: center; padding: 4rem 2rem;">
        <div style="font-size: 4rem; margin-bottom: 1.5rem;">🔧</div>
        <h1 style="font-size: 2rem; color: var(--accent);">Narzędzia Systemowe</h1>
        <p style="color: var(--text-secondary); max-width: 600px; margin: 0 auto 2rem;">
            Kolekcja przydatnych narzędzi diagnostycznych, skryptów automatyzacji,
            szybkich napraw i utility do codziennego zarządzania systemem.
        </p>
        <div style="display: inline-block; background: rgba(218, 133, 67, 0.2); color: #da8543; 
             padding: 0.5rem 1rem; border-radius: 20px; font-weight: bold;">
            W TRAKCIE ROZWOJU
        </div>
    </div>
    
    <div style="text-align: center; margin-top: 3rem;">
        <a href="{{ url_for('index') }}" style="
            color: var(--accent);
            text-decoration: none;
            font-weight: bold;
            display: inline-flex;
            align-items: center;
            gap: 0.5rem;
            padding: 0.8rem 1.5rem;
            border: 1px solid var(--accent);
            border-radius: 6px;
        ">
            ← Powrót do Centrum Kontroli
        </a>
    </div>
</div>
{% endblock %}
EOF
echo "✅ 5/5: templates/tools.html"

# WERYFIKACJA
echo ""
echo "=== WERYFIKACJA ==="
ls -la templates/
echo ""
echo "=== PODSUMOWANIE ==="
echo "✅ Wszystkie 8 szablonów gotowe:"
echo "   1. layout.html     - szablon bazowy"
echo "   2. index.html      - strona główna z 6 kafelkami"
echo "   3. archive.html    - archiwum/backupy (pełna obudowa)"
echo "   4. terminal.html   - placeholder"
echo "   5. explorer.html   - placeholder"
echo "   6. config.html     - placeholder"
echo "   7. pulse.html      - placeholder"
echo "   8. tools.html      - placeholder"
echo ""
echo "=== NASTĘPNY KROK ==="
echo "1. Zrestartować usługę Flask"
echo "2. Przetestować w przeglądarce"
echo "3. Sprawdzić responsywność i działanie statusu"
echo ""
echo "🎯 Cel główny Planu Ataku: PRZEPROWADZONY DO ETAPU CZ"
echo "   Pozostał tylko ETAP W (Weryfikacja) i restart usługi"
