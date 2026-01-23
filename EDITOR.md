#!/bin/bash
# SKRYPT: PRZEROBKA DASHBOARD V2 NA LAYOUT ZE ZDJĘCIA
# Data: 2026-01-23
# Autor: Wojtek (asystent CEO)

echo "=== PRZEROBKA DASHBOARD V2 NA LAYOUT ZE ZDJĘCIA ==="

# 1. BACKUP - LINIA OBOWIĄZKOWA (§2)
TIMESTAMP=$(date +%s)
echo "1. Tworzę backup obecnych szablonów..."
cp /var/www/dashboard_v2/templates/layout.html /var/www/dashboard_v2/templates/layout.html.backup_photo_layout_${TIMESTAMP}
cp /var/www/dashboard_v2/templates/index.html /var/www/dashboard_v2/templates/index.html.backup_photo_layout_${TIMESTAMP}
echo "   Backupy utworzone z timestamp: ${TIMESTAMP}"

# 2. Tworzymy nowy layout.html (bez menu bocznego, zgodnie ze zdjęciem)
echo "2. Tworzę nowy layout zgodny ze zdjęciem..."

cat > /tmp/new_layout_photo.html << 'EOF'
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}AI Firma Dashboard{% endblock %}</title>
    <style>
        /* ===== PODSTAWY ===== */
        :root {
            --bg-primary: #0d1117;
            --bg-secondary: #161b22;
            --bg-card: #21262d;
            --text-primary: #e6edf3;
            --text-secondary: #8b949e;
            --text-muted: #6e7681;
            --accent: #1f6feb;
            --accent-hover: #2b81ff;
            --success: #238636;
            --warning: #d29922;
            --critical: #da3633;
            --border: #30363d;
            --border-light: #21262d;
        }
        
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
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
        
        /* ===== KONTENER ===== */
        .container {
            max-width: 1200px;
            margin: 0 auto;
            padding: 0 20px;
        }
        
        /* ===== NAGŁÓWEK GÓRNY ===== */
        .top-bar {
            background-color: var(--bg-secondary);
            border-bottom: 1px solid var(--border);
            padding: 1rem 0;
        }
        
        .top-bar-content {
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 1rem;
        }
        
        .logo-section {
            display: flex;
            align-items: center;
            gap: 0.75rem;
        }
        
        .logo-icon {
            font-size: 1.5rem;
        }
        
        .logo-text {
            font-size: 1.25rem;
            font-weight: 600;
            color: var(--text-primary);
        }
        
        .status-section {
            display: flex;
            align-items: center;
            gap: 1rem;
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
        
        .status-text {
            font-size: 0.9rem;
            color: var(--text-secondary);
        }
        
        .username {
            color: var(--text-secondary);
            font-size: 0.9rem;
            font-weight: 500;
        }
        
        /* ===== TIMESTAMP ===== */
        .timestamp-bar {
            background-color: var(--bg-primary);
            border-bottom: 1px solid var(--border);
            padding: 0.75rem 0;
            text-align: center;
        }
        
        .timestamp-text {
            font-family: monospace;
            color: var(--text-secondary);
            font-size: 0.95rem;
        }
        
        /* ===== GŁÓWNA ZAWARTOŚĆ ===== */
        .main-content {
            padding: 2rem 0;
        }
        
        /* ===== HERO SEKCJA ===== */
        .hero-section {
            margin-bottom: 2.5rem;
        }
        
        .hero-title {
            font-size: 2.5rem;
            font-weight: 700;
            margin-bottom: 0.5rem;
            line-height: 1.2;
        }
        
        .hero-subtitle {
            font-size: 1.1rem;
            color: var(--text-secondary);
            max-width: 600px;
            line-height: 1.4;
        }
        
        /* ===== ALERT SECTION ===== */
        .alert-section {
            margin-bottom: 2.5rem;
        }
        
        .critical-alert {
            background-color: rgba(218, 54, 51, 0.15);
            border: 1px solid var(--critical);
            border-radius: 10px;
            padding: 1.5rem;
            display: flex;
            align-items: center;
            gap: 1rem;
        }
        
        .alert-icon {
            font-size: 2rem;
            color: var(--critical);
        }
        
        .alert-content h3 {
            font-size: 1.5rem;
            font-weight: 600;
            color: var(--critical);
            margin-bottom: 0.25rem;
        }
        
        .alert-content p {
            color: var(--text-secondary);
            font-size: 0.95rem;
        }
        
        /* ===== KAFELKI ===== */
        .tiles-section {
            display: flex;
            flex-direction: column;
            gap: 1.5rem;
        }
        
        .tile {
            background-color: var(--bg-card);
            border: 1px solid var(--border);
            border-radius: 10px;
            padding: 1.5rem;
            text-decoration: none;
            color: var(--text-primary);
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: all 0.2s ease;
        }
        
        .tile:hover {
            border-color: var(--accent);
            transform: translateX(5px);
        }
        
        .tile-content {
            flex: 1;
        }
        
        .tile-title {
            font-size: 1.3rem;
            font-weight: 600;
            margin-bottom: 0.5rem;
            display: flex;
            align-items: center;
            gap: 0.75rem;
        }
        
        .tile-icon {
            font-size: 1.25rem;
        }
        
        .tile-description {
            color: var(--text-secondary);
            font-size: 0.95rem;
            line-height: 1.4;
        }
        
        .tile-arrow {
            font-size: 1.5rem;
            color: var(--text-muted);
            margin-left: 1rem;
        }
        
        /* ===== STOPKA ===== */
        .footer {
            margin-top: 3rem;
            padding: 1.5rem 0;
            border-top: 1px solid var(--border);
            color: var(--text-secondary);
            font-size: 0.85rem;
            text-align: center;
        }
        
        /* ===== RESPONSYWNOŚĆ ===== */
        @media (max-width: 768px) {
            .container {
                padding: 0 15px;
            }
            
            .hero-title {
                font-size: 2rem;
            }
            
            .top-bar-content {
                flex-direction: column;
                align-items: flex-start;
                gap: 0.5rem;
            }
            
            .status-section {
                width: 100%;
                justify-content: space-between;
            }
            
            .tile {
                flex-direction: column;
                align-items: flex-start;
                gap: 1rem;
            }
            
            .tile-arrow {
                align-self: flex-end;
            }
        }
    </style>
    {% block extra_css %}{% endblock %}
    {% block extra_head %}{% endblock %}
</head>
<body>
    <!-- NAGŁÓWEK GÓRNY -->
    <header class="top-bar">
        <div class="container top-bar-content">
            <div class="logo-section">
                <div class="logo-icon">🤖</div>
                <div class="logo-text">AI FIRMA DASHBOARD</div>
            </div>
            <div class="status-section">
                <div class="status-indicator">
                    <div class="status-dot"></div>
                    <span class="status-text">Systemy sprawne</span>
                </div>
                <div class="username">Tomasz Lis</div>
            </div>
        </div>
    </header>
    
    <!-- TIMESTAMP -->
    <div class="timestamp-bar">
        <div class="container">
            <div class="timestamp-text" id="live-timestamp">
                Ładowanie czasu...
            </div>
        </div>
    </div>
    
    <!-- GŁÓWNA ZAWARTOŚĆ -->
    <main class="main-content">
        <div class="container">
            {% block content %}
            <!-- TREŚĆ BĘDZIE TU -->
            {% endblock %}
        </div>
    </main>
    
    <!-- STOPKA -->
    <footer class="footer">
        <div class="container">
            <p>© 2025 AI Firma Dashboard | Wersja 2.0 | <span id="footer-status">Status: <span class="status-indicator"><div class="status-dot"></div>online</span></span></p>
        </div>
    </footer>
    
    {% block scripts %}
    <script>
        // Aktualizacja timestamp
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
        
        updateTimestamp();
        setInterval(updateTimestamp, 1000);
    </script>
    {% endblock %}
    {% block extra_scripts %}{% endblock %}
</body>
</html>
EOF

# 3. Tworzymy nowy index.html (zgodny ze zdjęciem)
echo "3. Tworzę nowy index.html zgodny ze zdjęciem..."

cat > /tmp/new_index_photo.html << 'EOF'
{% extends "layout.html" %}

{% block title %}Centrum Kontroli - AI Firma{% endblock %}

{% block content %}
    <!-- HERO SEKCJA -->
    <section class="hero-section">
        <h1 class="hero-title">Centrum Kontroli AI Firma</h1>
        <p class="hero-subtitle">Zintegrowany panel zarządzania systemem i usługami</p>
    </section>
    
    <!-- ALERT SECTION (jak na zdjęciu "KRYTYCZNE") -->
    <section class="alert-section">
        <div class="critical-alert">
            <div class="alert-icon">⚠️</div>
            <div class="alert-content">
                <h3>KRYTYCZNE</h3>
                <p>Wymagana interwencja: System backupów wymaga konfiguracji. Brak aktywnych backupów z ostatnich 24 godzin.</p>
            </div>
        </div>
    </section>
    
    <!-- KAFELKI (jeden pod drugim jak na zdjęciu) -->
    <section class="tiles-section">
        <!-- KAFELEK 1: ARCHIWUM -->
        <a href="/archive" class="tile">
            <div class="tile-content">
                <div class="tile-title">
                    <span class="tile-icon">🗃️</span>
                    <span>ARCHIWUM</span>
                </div>
                <p class="tile-description">Zarządzanie backupami, Gold Image, przywracanie systemu. Kompleksowy system backupów 3-stopniowych.</p>
            </div>
            <div class="tile-arrow">→</div>
        </a>
        
        <!-- KAFELEK 2: PULS SYSTEMU -->
        <a href="/pulse" class="tile">
            <div class="tile-content">
                <div class="tile-title">
                    <span class="tile-icon">❤️</span>
                    <span>PULS SYSTEMU</span>
                </div>
                <p class="tile-description">Monitorowanie usług w czasie rzeczywistym, metryki wydajności, logi systemowe i alerty.</p>
            </div>
            <div class="tile-arrow">→</div>
        </a>
        
        <!-- KAFELEK 3: TERMINAL -->
        <a href="/terminal" class="tile">
            <div class="tile-content">
                <div class="tile-title">
                    <span class="tile-icon">💻</span>
                    <span>TERMINAL</span>
                </div>
                <p class="tile-description">Bezpieczny dostęp do konsoli serwera, wykonywanie poleceń, zarządzanie procesami.</p>
            </div>
            <div class="tile-arrow">→</div>
        </a>
        
        <!-- KAFELEK 4: EKSPLORATOR -->
        <a href="/explorer" class="tile">
            <div class="tile-content">
                <div class="tile-title">
                    <span class="tile-icon">📁</span>
                    <span>EKSPLORATOR</span>
                </div>
                <p class="tile-description">Przeglądanie plików systemu, menedżer zasobów, zarządzanie uprawnieniami.</p>
            </div>
            <div class="tile-arrow">→</div>
        </a>
        
        <!-- KAFELEK 5: KONFIGURACJA -->
        <a href="/config" class="tile">
            <div class="tile-content">
                <div class="tile-title">
                    <span class="tile-icon">⚙️</span>
                    <span>KONFIGURACJA</span>
                </div>
                <p class="tile-description">Ustawienia systemu, zarządzanie użytkownikami, integracje zewnętrzne, backup konfiguracji.</p>
            </div>
            <div class="tile-arrow">→</div>
        </a>
    </section>
{% endblock %}

{% block scripts %}
{{ super() }}
<script>
    // Pobieranie statusu systemu
    async function fetchSystemStatus() {
        try {
            const response = await fetch('/api/health');
            const data = await response.json();
            
            // Aktualizacja statusu w nagłówku
            const statusDot = document.querySelector('.status-dot');
            const statusText = document.querySelector('.status-text');
            
            if (data.status === 'online') {
                statusDot.style.backgroundColor = 'var(--success)';
                statusText.textContent = 'Systemy sprawne';
            } else {
                statusDot.style.backgroundColor = 'var(--critical)';
                statusText.textContent = 'Problemy z systemem';
            }
            
        } catch (error) {
            console.error('Błąd pobierania statusu:', error);
        }
    }
    
    // Pobierz status przy załadowaniu
    fetchSystemStatus();
    setInterval(fetchSystemStatus, 30000); // Co 30 sekund
</script>
{% endblock %}
EOF

# 4. Zastępujemy pliki
cp /tmp/new_layout_photo.html /var/www/dashboard_v2/templates/layout.html
cp /tmp/new_index_photo.html /var/www/dashboard_v2/templates/index.html

# 5. Ustawiamy uprawnienia
chown ubuntu:ubuntu /var/www/dashboard_v2/templates/layout.html
chown ubuntu:ubuntu /var/www/dashboard_v2/templates/index.html
chmod 644 /var/www/dashboard_v2/templates/layout.html
chmod 644 /var/www/dashboard_v2/templates/index.html

echo "=== ZAKOŃCZONO ==="
echo "Dashboard v2 został przerobiony na layout ze zdjęcia:"
echo "1. Nowy layout bez menu bocznego"
echo "2. Nagłówek z timestampem"
echo "3. Hero sekcja z tytułem"
echo "4. Alert 'KRYTYCZNE'"
echo "5. Kafelki jeden pod drugim"
echo ""
echo "Odśwież stronę: http://57.128.247.215/"
echo "Backupy zachowane w templates/*.backup_photo_layout_${TIMESTAMP}"
EOF
