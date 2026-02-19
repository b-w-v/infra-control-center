# Deployment Guide

## Table of Contents
- [Prerequisites](#prerequisites)
- [Static Hosting](#static-hosting)
- [Self-Hosted Options](#self-hosted-options)
- [Containerization](#containerization)
- [CI/CD Integration](#cicd-integration)
- [Performance Optimization](#performance-optimization)

## Prerequisites

Before deploying, ensure you have:
- Git installed
- A GitHub account (for repository hosting)
- Basic command line knowledge
- (Optional) Node.js for build tools
- (Optional) Docker for containerization

## Static Hosting

### GitHub Pages

**Method 1: Repository Settings**
1. Push your code to GitHub
2. Go to repository Settings → Pages
3. Select branch (usually `main`)
4. Select folder (`/ (root)`)
5. Click Save

Your site will be available at: `https://b-w-v.github.io/infra-control-center/`

**Method 2: GitHub Actions**
Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./
```

### Netlify

**One-Click Deploy:**
1. Go to [Netlify](https://www.netlify.com)
2. Click "Add new site" → "Import an existing project"
3. Connect to GitHub and select your repository
4. Build settings:
   - Build command: (leave empty)
   - Publish directory: `/`
5. Click "Deploy site"

**Custom Domain:**
1. Go to Site settings → Domain management
2. Add custom domain
3. Configure DNS records as shown

**netlify.toml** (optional):
```toml
[build]
  publish = "."
  
[[redirects]]
  from = "/*"
  to = "/index.html"
  status = 200

[[headers]]
  for = "/*"
  [headers.values]
    X-Frame-Options = "DENY"
    X-XSS-Protection = "1; mode=block"
    X-Content-Type-Options = "nosniff"
```

### Vercel

**CLI Deployment:**
```bash
npm install -g vercel
vercel login
vercel --prod
```

**vercel.json:**
```json
{
  "version": 2,
  "builds": [
    {
      "src": "*.html",
      "use": "@vercel/static"
    }
  ],
  "routes": [
    {
      "src": "/(.*)",
      "dest": "/$1"
    }
  ]
}
```

### Cloudflare Pages

1. Log in to Cloudflare dashboard
2. Go to Pages → Create a project
3. Connect to GitHub
4. Select repository
5. Build settings:
   - Framework preset: None
   - Build command: (leave empty)
   - Build output directory: `/`
6. Deploy

## Self-Hosted Options

### Nginx

**Install Nginx:**
```bash
sudo apt update
sudo apt install nginx
```

**Configuration** (`/etc/nginx/sites-available/infra-control`):
```nginx
server {
    listen 80;
    server_name your-domain.com;
    root /var/www/infra-control-center;
    index index.html;

    location / {
        try_files $uri $uri/ =404;
    }

    # Compression
    gzip on;
    gzip_vary on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # Security headers
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header X-Content-Type-Options "nosniff" always;
}
```

**Enable site:**
```bash
sudo ln -s /etc/nginx/sites-available/infra-control /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

**SSL with Let's Encrypt:**
```bash
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your-domain.com
```

### Apache

**Configuration** (`.htaccess`):
```apache
<IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteBase /
    
    # Security headers
    Header set X-Frame-Options "SAMEORIGIN"
    Header set X-XSS-Protection "1; mode=block"
    Header set X-Content-Type-Options "nosniff"
    
    # Compression
    <IfModule mod_deflate.c>
        AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css text/javascript application/javascript
    </IfModule>
</IfModule>
```

### Node.js Server

**server.js:**
```javascript
const express = require('express');
const path = require('path');
const compression = require('compression');
const helmet = require('helmet');

const app = express();
const PORT = process.env.PORT || 3000;

// Security
app.use(helmet());

// Compression
app.use(compression());

// Static files
app.use(express.static(path.join(__dirname), {
    maxAge: '1d'
}));

// SPA fallback
app.get('*', (req, res) => {
    res.sendFile(path.join(__dirname, 'index.html'));
});

app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
```

**package.json:**
```json
{
  "scripts": {
    "start": "node server.js"
  },
  "dependencies": {
    "express": "^4.18.2",
    "compression": "^1.7.4",
    "helmet": "^7.0.0"
  }
}
```

## Containerization

### Docker

**Dockerfile:**
```dockerfile
FROM nginx:alpine

# Copy application files
COPY . /usr/share/nginx/html

# Copy custom nginx config (optional)
# COPY nginx.conf /etc/nginx/nginx.conf

# Expose port
EXPOSE 80

# Health check
HEALTHCHECK --interval=30s --timeout=3s \
  CMD wget --quiet --tries=1 --spider http://localhost/ || exit 1

CMD ["nginx", "-g", "daemon off;"]
```

**Build and run:**
```bash
docker build -t infra-control-center .
docker run -d -p 8080:80 --name infra-app infra-control-center
```

**docker-compose.yml:**
```yaml
version: '3.8'

services:
  web:
    build: .
    ports:
      - "8080:80"
    restart: unless-stopped
    volumes:
      - ./logs:/var/log/nginx
    environment:
      - NODE_ENV=production
```

Run with:
```bash
docker-compose up -d
```

### Kubernetes

**deployment.yaml:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: infra-control-center
spec:
  replicas: 3
  selector:
    matchLabels:
      app: infra-control-center
  template:
    metadata:
      labels:
        app: infra-control-center
    spec:
      containers:
      - name: web
        image: your-registry/infra-control-center:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "100m"
          limits:
            memory: "128Mi"
            cpu: "200m"

---
apiVersion: v1
kind: Service
metadata:
  name: infra-control-center
spec:
  selector:
    app: infra-control-center
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

Deploy:
```bash
kubectl apply -f deployment.yaml
```

## CI/CD Integration

### GitHub Actions

**.github/workflows/deploy.yml:**
```yaml
name: Build and Deploy

on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - name: Checkout code
      uses: actions/checkout@v3
    
    - name: Validate HTML
      uses: Cyb3r-Jak3/html5validator-action@v7
      with:
        root: .
    
    - name: Deploy to Netlify
      uses: nwtgck/actions-netlify@v2
      with:
        publish-dir: '.'
        production-deploy: true
      env:
        NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
        NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### GitLab CI

**.gitlab-ci.yml:**
```yaml
stages:
  - test
  - deploy

validate:
  stage: test
  image: node:18
  script:
    - npm install -g html-validator-cli
    - html-validator --file=index.html

deploy_production:
  stage: deploy
  image: alpine:latest
  before_script:
    - apk add --no-cache curl
  script:
    - curl -X POST -d @. https://your-deployment-webhook
  only:
    - main
```

## Performance Optimization

### Minification

**HTML Minifier:**
```bash
npm install -g html-minifier
html-minifier --collapse-whitespace --remove-comments --minify-css --minify-js index.html -o index.min.html
```

**CSS/JS Minification:**
Create build script `build.sh`:
```bash
#!/bin/bash

# Install tools
npm install -g clean-css-cli uglify-js

# Extract and minify CSS
# (If you externalize CSS from HTML)

# Extract and minify JS
# (If you externalize JS from HTML)

echo "Build complete!"
```

### Content Delivery Network (CDN)

**Cloudflare CDN:**
1. Add your site to Cloudflare
2. Update nameservers
3. Enable caching and minification

**AWS CloudFront:**
```bash
aws cloudfront create-distribution \
  --origin-domain-name your-s3-bucket.s3.amazonaws.com \
  --default-root-object index.html
```

### Caching Strategy

**Nginx caching:**
```nginx
location ~* \.(jpg|jpeg|png|gif|ico|css|js)$ {
    expires 1y;
    add_header Cache-Control "public, immutable";
}
```

## Monitoring

### Uptime Monitoring

**UptimeRobot:**
1. Sign up at [UptimeRobot](https://uptimerobot.com)
2. Add new monitor
3. Set URL and check interval

**Pingdom:**
Similar setup at [Pingdom](https://www.pingdom.com)

### Analytics

**Google Analytics 4:**
```html
<head>
    <!-- Google tag (gtag.js) -->
    <script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXXXXXXXX"></script>
    <script>
        window.dataLayer = window.dataLayer || [];
        function gtag(){dataLayer.push(arguments);}
        gtag('js', new Date());
        gtag('config', 'G-XXXXXXXXXX');
    </script>
</head>
```

## Troubleshooting

### Common Issues

**CORS Errors:**
Add headers in server config or use `meta` tags:
```html
<meta http-equiv="Access-Control-Allow-Origin" content="*">
```

**404 Errors:**
Ensure your server is configured for SPA routing

**Slow Loading:**
- Enable compression (gzip/brotli)
- Optimize images
- Use CDN
- Implement caching

---

For additional help, see the main [README.md](../README.md) or open an issue on GitHub.