#!/bin/bash
# ============================================================================
# SKRYPT: tworzenie_systemu_szablonow.sh
# CEL: Stworzenie podstawowego systemu szablonów Jinja2 dla Dashboard AI Firma
# DATA: $(date)
# AUTOR: Wojtek (AI Programista)
# ============================================================================

echo "=== ROZPOCZYNAM TWORZENIE SYSTEMU SZABLONÓW ==="

# 1. Tworzymy katalog templates
mkdir -p /opt/ai_firma_dashboard/templates
echo "✅ Utworzono katalog /opt/ai_firma_dashboard/templates/"

# 2. Tworzymy podstawowy szablon layout.html
cat > /opt/ai_firma_dashboard/templates/layout.html << 'EOF'
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}AI Firma Dashboard{% endblock %}</title>
    <style>
        /* ===== PODSTAWOWY CIEMNY MOTYW ===== */
        :root {
            --bg-primary: #1a1d23;
            --bg-secondary: #2d333b;
            --text-primary: #e6edf3;
            --text-secondary: #adbac7;
            --accent: #539bf5;
            --success: #57ab5a;
            --error: #e5534b;
            --border: #444c56;
        }
        
        * {
            box-sizing: border-box;
        }
        
        body {
            margin: 0;
            padding: 0;
            background-color: var(--bg-primary);
            color: var(--text-primary);
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif;
            line-height: 1.5;
            min-height: 100vh;
        }
        
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 20px;
        }
        
        /* ===== NAGŁÓWEK ===== */
        .header {
            background: var(--bg-secondary);
            border-bottom: 1px solid var(--border);
            padding: 1rem 0;
            margin-bottom: 2rem;
        }
        
        .header-content {
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 1rem;
        }
        
        .logo {
            font-size: 1.5rem;
            font-weight: bold;
            color: var(--accent);
        }
        
        .user-info {
            color: var(--text-secondary);
            font-size: 0.9rem;
        }
        
        .status-indicator {
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }
        
        .status-dot {
            width: 10px;
            height: 10px;
            border-radius: 50%;
            background-color: var(--success);
        }
        
        .status-dot.error {
            background-color: var(--error);
        }
        
        .timestamp {
            color: var(--text-secondary);
            font-size: 0.9rem;
            font-family: monospace;
        }
        
        /* ===== STOPKA ===== */
        .footer {
            margin-top: 3rem;
            padding: 1.5rem 0;
            border-top: 1px solid var(--border);
            color: var(--text-secondary);
            font-size: 0.9rem;
            text-align: center;
        }
    </style>
    {% block extra_css %}{% endblock %}
    
    {% block extra_head %}{% endblock %}
</head>
<body>
    <header class="header">
        <div class="container header-content">
            <div class="logo">AI FIRMA DASHBOARD</div>
            <div class="status-indicator">
                <div class="status-dot {{ 'error' if status == 'error' else '' }}"></div>
                <span id="status-text">{{ status_message }}</span>
            </div>
            <div class="user-info">Tomasz Lis</div>
        </div>
    </header>

    <div class="container">
        <!-- TIMESTAMP NA STRONIE GŁÓWNEJ TYLKO -->
        {% if request.path == '/' %}
        <div class="timestamp" id="live-timestamp">
            Ładowanie czasu...
        </div>
        {% endif %}
        
        {% block content %}
        <!-- TREŚĆ STRONY BĘDZIE TU -->
        {% endblock %}
    </div>

    <footer class="footer">
        <div class="container">
            © 2024 AI Firma Dashboard | Wersja 1.0 | 
            <span id="footer-status">Status: <span class="status-indicator">
                <div class="status-dot {{ 'error' if status == 'error' else '' }}"></div>
                {{ status }}
            </span></span>
        </div>
    </footer>

    {% block scripts %}
    <!-- SKRYPT DO ODSWIEŻANIA CZASU NA STRONIE GŁÓWNEJ -->
    {% if request.path == '/' %}
    <script>
        function updateTimestamp() {
            const now = new Date();
            const options = { 
                weekday: 'long', 
                year: 'numeric', 
                month: 'long', 
                day: 'numeric',
                hour: '2-digit',
                minute: '2-digit',
                second: '2-digit',
                hour12: false
            };
            document.getElementById('live-timestamp').textContent = 
                now.toLocaleDateString('pl-PL', options);
        }
        
        // Odświeżaj co sekundę
        updateTimestamp();
        setInterval(updateTimestamp, 1000);
    </script>
    {% endif %}
    {% endblock %}
    
    {% block extra_scripts %}{% endblock %}
</body>
</html>
EOF

echo "✅ Utworzono szablon /opt/ai_firma_dashboard/templates/layout.html"

# 3. Weryfikacja
echo ""
echo "=== WERYFIKACJA ==="
ls -la /opt/ai_firma_dashboard/templates/
echo ""
echo "=== PODGLĄD STRUKTURY SZABLONU ==="
head -30 /opt/ai_firma_dashboard/templates/layout.html
echo "..."
tail -10 /opt/ai_firma_dashboard/templates/layout.html

echo ""
echo "=== INSTRUKCJE DALSZE ==="
echo "1. Szablon layout.html jest gotowy"
echo "2. Następny krok: zaktualizować app.py o kontekst statusu"
echo "3. Potem: stworzyć index.html z kafelkami"
echo ""
echo "✅ SYSTEM SZABLONÓW ZAINICJOWANY"
