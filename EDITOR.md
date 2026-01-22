#!/bin/bash
# SKRYPT: POPRAWA LAYOUTU I DODANIE MENU BOCZNEGO
# Data: 2026-01-23
# Autor: Wojtek (asystent CEO)

echo "=== POPRAWA LAYOUTU DASHBOARD_V2 ==="

# 1. BACKUP - LINIA OBOWIĄZKOWA (§2)
TIMESTAMP=$(date +%s)
echo "1. Tworzę backup szablonów..."
cp /var/www/dashboard_v2/templates/layout.html /var/www/dashboard_v2/templates/layout.html.backup_${TIMESTAMP}
cp /var/www/dashboard_v2/templates/index.html /var/www/dashboard_v2/templates/index.html.backup_${TIMESTAMP}
echo "   Backupy utworzone z timestamp: ${TIMESTAMP}"

# 2. Modyfikacja layout.html - dodanie menu bocznego
echo "2. Dodaję menu boczne do layout.html..."

# Tworzymy nowy layout z menu bocznym
cat > /tmp/new_layout.html << 'EOF'
<!DOCTYPE html>
<html lang="pl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>{% block title %}AI Firma Dashboard v2{% endblock %}</title>
    <style>
        /* ===== PODSTAWOWY CIEMNY MOTYW ===== */
        :root {
            --bg-primary: #0d1117;
            --bg-secondary: #161b22;
            --bg-sidebar: #010409;
            --text-primary: #e6edf3;
            --text-secondary: #8b949e;
            --accent: #1f6feb;
            --accent-hover: #2b81ff;
            --success: #238636;
            --error: #da3633;
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
            display: flex;
        }
        
        /* ===== MENU BOCZNE ===== */
        .sidebar {
            width: 250px;
            background-color: var(--bg-sidebar);
            border-right: 1px solid var(--border);
            min-height: 100vh;
            position: fixed;
            left: 0;
            top: 0;
            z-index: 1000;
        }
        
        .logo-area {
            padding: 1.5rem 1rem;
            border-bottom: 1px solid var(--border);
            display: flex;
            align-items: center;
            gap: 0.75rem;
        }
        
        .logo-icon {
            font-size: 1.5rem;
            color: var(--accent);
        }
        
        .logo-text {
            font-size: 1.1rem;
            font-weight: 600;
            color: var(--text-primary);
        }
        
        .nav-menu {
            padding: 1rem 0;
        }
        
        .nav-item {
            display: flex;
            align-items: center;
            gap: 0.75rem;
            padding: 0.75rem 1.5rem;
            color: var(--text-secondary);
            text-decoration: none;
            transition: all 0.2s ease;
            border-left: 3px solid transparent;
        }
        
        .nav-item:hover {
            background-color: var(--bg-secondary);
            color: var(--text-primary);
            border-left-color: var(--accent);
        }
        
        .nav-item.active {
            background-color: var(--bg-secondary);
            color: var(--text-primary);
            border-left-color: var(--accent);
            font-weight: 500;
        }
        
        .nav-icon {
            font-size: 1.1rem;
            width: 20px;
            text-align: center;
        }
        
        /* ===== GŁÓWNA ZAWARTOŚĆ ===== */
        .main-content {
            flex: 1;
            margin-left: 250px;
            min-height: 100vh;
            display: flex;
            flex-direction: column;
        }
        
        .top-header {
            background-color: var(--bg-secondary);
            border-bottom: 1px solid var(--border);
            padding: 1rem 2rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 1rem;
        }
        
        .page-title {
            font-size: 1.4rem;
            font-weight: 600;
        }
        
        .user-info {
            display: flex;
            align-items: center;
            gap: 1rem;
        }
        
        .status-badge {
            display: flex;
            align-items: center;
            gap: 0.5rem;
            padding: 0.4rem 0.8rem;
            background-color: var(--success);
            border-radius: 20px;
            font-size: 0.85rem;
            font-weight: 500;
        }
        
        .status-dot {
            width: 8px;
            height: 8px;
            border-radius: 50%;
            background-color: white;
        }
        
        .username {
            color: var(--text-secondary);
            font-weight: 500;
        }
        
        .content-wrapper {
            flex: 1;
            padding: 2rem;
        }
        
        /* ===== STOPKA ===== */
        .footer {
            margin-top: auto;
            padding: 1.5rem 2rem;
            border-top: 1px solid var(--border);
            color: var(--text-secondary);
            font-size: 0.85rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
            flex-wrap: wrap;
            gap: 1rem;
        }
        
        .footer-left {
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }
        
        .footer-status {
            display: flex;
            align-items: center;
            gap: 0.5rem;
        }
        
        .timestamp {
            font-family: monospace;
            color: var(--text-secondary);
            font-size: 0.9rem;
        }
        
        /* ===== RESPONSYWNOŚĆ ===== */
        @media (max-width: 768px) {
            .sidebar {
                width: 70px;
            }
            
            .logo-text, .nav-text {
                display: none;
            }
            
            .main-content {
                margin-left: 70px;
            }
            
            .nav-item {
                justify-content: center;
                padding: 0.75rem;
            }
        }
    </style>
    {% block extra_css %}{% endblock %}
    {% block extra_head %}{% endblock %}
</head>
<body>
    <!-- MENU BOCZNE -->
    <nav class="sidebar">
        <div class="logo-area">
            <div class="logo-icon">🤖</div>
            <div class="logo-text">AI FIRMA</div>
        </div>
        
        <div class="nav-menu">
            <a href="/" class="nav-item {% if request.path == '/' %}active{% endif %}">
                <span class="nav-icon">📊</span>
                <span class="nav-text">Pulpit</span>
            </a>
            <a href="/archive" class="nav-item {% if request.path == '/archive' %}active{% endif %}">
                <span class="nav-icon">🗃️</span>
                <span class="nav-text">Archiwum</span>
            </a>
            <a href="/pulse" class="nav-item {% if request.path == '/pulse' %}active{% endif %}">
                <span class="nav-icon">❤️</span>
                <span class="nav-text">Puls Systemu</span>
            </a>
            <a href="/terminal" class="nav-item {% if request.path == '/terminal' %}active{% endif %}">
                <span class="nav-icon">💻</span>
                <span class="nav-text">Terminal</span>
            </a>
            <a href="/explorer" class="nav-item {% if request.path == '/explorer' %}active{% endif %}">
                <span class="nav-icon">📁</span>
                <span class="nav-text">Eksplorator</span>
            </a>
            <a href="/config" class="nav-item {% if request.path == '/config' %}active{% endif %}">
                <span class="nav-icon">⚙️</span>
                <span class="nav-text">Konfiguracja</span>
            </a>
        </div>
    </nav>
    
    <!-- GŁÓWNA ZAWARTOŚĆ -->
    <div class="main-content">
        <header class="top-header">
            <div class="page-title">{% block page_title %}Centrum Kontroli AI Firma{% endblock %}</div>
            <div class="user-info">
                <div class="status-badge">
                    <div class="status-dot"></div>
                    <span>Systemy sprawne</span>
                </div>
                <div class="username">Tomasz Lis</div>
            </div>
        </header>
        
        <div class="content-wrapper">
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
            <div class="footer-left">
                <span>© 2025 AI Firma Dashboard</span>
                <span class="footer-status">
                    <span class="status-dot"></span>
                    <span>v2.0</span>
                </span>
            </div>
            <div class="timestamp" id="footer-timestamp">
                {% if request.path == '/' %}
                <script>
                    function updateFooterTime() {
                        const now = new Date();
                        document.getElementById('footer-timestamp').textContent = 
                            now.toLocaleString('pl-PL');
                    }
                    updateFooterTime();
                    setInterval(updateFooterTime, 1000);
                </script>
                {% endif %}
            </div>
        </footer>
    </div>
    
    {% block scripts %}{% endblock %}
    {% block extra_scripts %}{% endblock %}
</body>
</html>
EOF

# 3. Zastępujemy stary layout nowym
cp /tmp/new_layout.html /var/www/dashboard_v2/templates/layout.html
echo "   Layout.html zaktualizowany z menu bocznym"

# 4. Modyfikacja index.html - 6 kafelków zgodnie ze zdjęciami
echo "3. Aktualizuję index.html z 6 kafelkami..."

cat > /tmp/new_index.html << 'EOF'
{% extends "layout.html" %}

{% block title %}Centrum Kontroli - AI Firma{% endblock %}

{% block page_title %}Centrum Kontroli AI Firma{% endblock %}

{% block content %}
<div class="dashboard-grid">
    <!-- KAFELEK 1: PULPIT -->
    <a href="/" class="tile">
        <div class="tile-icon">📊</div>
        <div class="tile-title">Pulpit</div>
        <div class="tile-description">Podgląd statusu systemu, metryki i alerty</div>
        <div class="tile-status">
            <span class="status-indicator active"></span>
            <span>Aktywny</span>
        </div>
    </a>
    
    <!-- KAFELEK 2: ARCHIWUM -->
    <a href="/archive" class="tile">
        <div class="tile-icon">🗃️</div>
        <div class="tile-title">Archiwum</div>
        <div class="tile-description">Zarządzanie backupami, Gold Image, przywracanie systemu</div>
        <div class="tile-status">
            <span class="status-indicator active"></span>
            <span>5 funkcji</span>
        </div>
    </a>
    
    <!-- KAFELEK 3: PULS SYSTEMU -->
    <a href="/pulse" class="tile">
        <div class="tile-icon">❤️</div>
        <div class="tile-title">Puls Systemu</div>
        <div class="tile-description">Monitorowanie usług, logi, wydajność w czasie rzeczywistym</div>
        <div class="tile-status">
            <span class="status-indicator warning"></span>
            <span>W budowie</span>
        </div>
    </a>
    
    <!-- KAFELEK 4: TERMINAL -->
    <a href="/terminal" class="tile">
        <div class="tile-icon">💻</div>
        <div class="tile-title">Terminal</div>
        <div class="tile-description">Bezpieczny dostęp do konsoli, wykonywanie poleceń</div>
        <div class="tile-status">
            <span class="status-indicator warning"></span>
            <span>W budowie</span>
        </div>
    </a>
    
    <!-- KAFELEK 5: EKSPLORATOR -->
    <a href="/explorer" class="tile">
        <div class="tile-icon">📁</div>
        <div class="tile-title">Eksplorator</div>
        <div class="tile-description">Przeglądanie plików systemu, menedżer zasobów</div>
        <div class="tile-status">
            <span class="status-indicator warning"></span>
            <span>W budowie</span>
        </div>
    </a>
    
    <!-- KAFELEK 6: KONFIGURACJA -->
    <a href="/config" class="tile">
        <div class="tile-icon">⚙️</div>
        <div class="tile-title">Konfiguracja</div>
        <div class="tile-description">Ustawienia systemu, użytkownicy, integracje</div>
        <div class="tile-status">
            <span class="status-indicator warning"></span>
            <span>W budowie</span>
        </div>
    </a>
</div>

<style>
    .dashboard-grid {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
        gap: 1.5rem;
        margin-top: 2rem;
    }
    
    .tile {
        background: var(--bg-secondary);
        border: 1px solid var(--border);
        border-radius: 12px;
        padding: 1.5rem;
        text-decoration: none;
        color: var(--text-primary);
        display: flex;
        flex-direction: column;
        transition: all 0.3s ease;
        min-height: 180px;
        position: relative;
        overflow: hidden;
    }
    
    .tile:hover {
        transform: translateY(-5px);
        border-color: var(--accent);
        box-shadow: 0 10px 25px rgba(31, 111, 235, 0.15);
    }
    
    .tile-icon {
        font-size: 2.8rem;
        margin-bottom: 1rem;
        opacity: 0.9;
    }
    
    .tile-title {
        font-size: 1.3rem;
        font-weight: 600;
        margin-bottom: 0.5rem;
    }
    
    .tile-description {
        color: var(--text-secondary);
        font-size: 0.95rem;
        line-height: 1.4;
        margin-bottom: 1rem;
        flex-grow: 1;
    }
    
    .tile-status {
        display: flex;
        align-items: center;
        gap: 0.5rem;
        font-size: 0.85rem;
        color: var(--text-secondary);
    }
    
    .status-indicator {
        width: 8px;
        height: 8px;
        border-radius: 50%;
        background-color: var(--success);
    }
    
    .status-indicator.active {
        background-color: var(--success);
    }
    
    .status-indicator.warning {
        background-color: #d29922;
    }
    
    @media (max-width: 768px) {
        .dashboard-grid {
            grid-template-columns: 1fr;
        }
    }
</style>
{% endblock %}

{% block scripts %}
<script>
    // Aktualizacja timestamp na stronie głównej
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
        const timestampEl = document.getElementById('live-timestamp');
        if (timestampEl) {
            timestampEl.textContent = now.toLocaleDateString('pl-PL', options);
        }
    }
    
    updateTimestamp();
    setInterval(updateTimestamp, 1000);
    
    // Pobieranie statusu systemu
    async function fetchSystemStatus() {
        try {
            const response = await fetch('/api/health');
            const data = await response.json();
            
            // Można tutaj zaktualizować statusy kafelków na podstawie API
            console.log('System status:', data);
        } catch (error) {
            console.error('Błąd pobierania statusu:', error);
        }
    }
    
    // Pobierz status przy załadowaniu strony
    fetchSystemStatus();
</script>
{% endblock %}
EOF

# 5. Zastępujemy stary index nowym
cp /tmp/new_index.html /var/www/dashboard_v2/templates/index.html
echo "   Index.html zaktualizowany z 6 kafelkami"

# 6. Poprawiamy uprawnienia
echo "4. Ustawiam uprawnienia..."
chown ubuntu:ubuntu /var/www/dashboard_v2/templates/layout.html
chown ubuntu:ubuntu /var/www/dashboard_v2/templates/index.html
chmod 644 /var/www/dashboard_v2/templates/layout.html
chmod 644 /var/www/dashboard_v2/templates/index.html

echo "=== ZAKOŃCZONO ==="
echo "Dashboard v2 został zaktualizowany:"
echo "1. Dodano menu boczne (6 pozycji)"
echo "2. Dodano stronę główną z 6 kafelkami"
echo "3. Backupy zapisane w templates/*.backup_${TIMESTAMP}"
echo ""
echo "Następny krok: Test w przeglądarce"
echo "Adres: http://57.128.247.215:5001 (gdy uruchomimy)"
EOF
