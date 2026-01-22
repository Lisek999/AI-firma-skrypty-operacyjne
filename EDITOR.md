#!/bin/bash
# SKRYPT: ZASTĄPIENIE DASHBOARD V1 PRZEZ V2 JAKO GŁÓWNY
# Data: 2026-01-23
# Autor: Wojtek (asystent CEO)

echo "=== ZASTĘPOWANIE DASHBOARD V1 PRZEZ V2 W NGINX ==="

# 1. BACKUP - LINIA OBOWIĄZKOWA (§2)
TIMESTAMP=$(date +%s)
echo "1. Tworzę backup konfiguracji Nginx..."
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.backup_v2_replacement_${TIMESTAMP}
echo "   Backup utworzony: default.backup_v2_replacement_${TIMESTAMP}"

# 2. Tworzymy nową konfigurację Nginx
echo "2. Konfiguruję Nginx do proxy dashboard v2..."

cat > /tmp/nginx_dashboard_v2.conf << 'EOF'
# Konfiguracja Nginx dla AI Firma Dashboard v2
# Główny dashboard: v2 (port 5001)
# Stary dashboard: v1 (port 5000) - tylko dla kompatybilności

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    server_name _;
    
    # GŁÓWNA LOKALIZACJA - dashboard v2
    location / {
        proxy_pass http://127.0.0.1:5001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # Cache control
        proxy_cache_bypass $http_pragma;
        proxy_cache_revalidate on;
        expires off;
    }
    
    # Static files - serwowane bezpośrednio przez Nginx dla wydajności
    location /static/ {
        alias /var/www/dashboard_v2/static/;
        expires 1h;
        add_header Cache-Control "public, immutable";
    }
    
    # API endpointy - też do v2
    location /api/ {
        proxy_pass http://127.0.0.1:5001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        
        # CORS headers
        add_header 'Access-Control-Allow-Origin' '*' always;
        add_header 'Access-Control-Allow-Methods' 'GET, POST, OPTIONS' always;
        add_header 'Access-Control-Allow-Headers' 'DNT,User-Agent,X-Requested-With,If-Modified-Since,Cache-Control,Content-Type,Range,X-API-Key' always;
    }
    
    # Specjalna lokalizacja dla starego dashboard v1 (tylko jeśli potrzebny)
    location /legacy/ {
        proxy_pass http://127.0.0.1:5000/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    
    # Health check endpoint
    location /health {
        access_log off;
        return 200 "healthy\n";
        add_header Content-Type text/plain;
    }
    
    # Deny access to . files
    location ~ /\. {
        deny all;
        access_log off;
        log_not_found off;
    }
}
EOF

# 3. Zastępujemy konfigurację
echo "3. Zastępuję konfigurację Nginx..."
sudo cp /tmp/nginx_dashboard_v2.conf /etc/nginx/sites-available/default

# 4. Test konfiguracji
echo "4. Testuję konfigurację Nginx..."
sudo nginx -t

if [ $? -eq 0 ]; then
    echo "5. Restartuję Nginx..."
    sudo systemctl restart nginx
    echo "   ✅ Nginx zrestartowany pomyślnie"
    
    echo ""
    echo "=== DASHBOARD V2 JEST TERAZ GŁÓWNY ==="
    echo "Adres: http://57.128.247.215/"
    echo "Archiwum: http://57.128.247.215/archive"
    echo "API Health: http://57.128.247.215/api/health"
    echo ""
    echo "=== DASHBOARD V1 (stary) DOSTĘPNY JAKO: ==="
    echo "http://57.128.247.215/legacy/  (lub bezpośrednio :5000)"
    echo ""
    echo "=== DASHBOARD V2 BEZPOŚREDNIO: ==="
    echo "http://57.128.247.215:5001/"
else
    echo "   ❌ BŁĄD: Konfiguracja Nginx nieprawidłowa"
    echo "   Przywracam backup..."
    sudo cp /etc/nginx/sites-available/default.backup_v2_replacement_${TIMESTAMP} /etc/nginx/sites-available/default
    sudo nginx -t && sudo systemctl reload nginx
    echo "   ✅ Przywrócono oryginalną konfigurację"
fi

echo "=== ZAKOŃCZONO ==="
