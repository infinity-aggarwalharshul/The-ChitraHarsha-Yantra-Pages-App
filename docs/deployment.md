# Deployment Guide

## Overview

This guide covers deployment options for the ChitraHarsha VPK Yantra platform, from simple static hosting to enterprise-scale global deployments.

## Quick Deployment Options

### 1. Static File Hosting

#### GitHub Pages
```bash
# 1. Push to GitHub repository
git add .
git commit -m "Deploy to GitHub Pages"
git push origin main

# 2. Enable GitHub Pages in repository settings
# Go to Settings > Pages > Source: Deploy from branch (main)
```

#### Netlify
```bash
# 1. Install Netlify CLI
npm install -g netlify-cli

# 2. Deploy
netlify deploy --prod --dir .
```

#### Vercel
```bash
# 1. Install Vercel CLI
npm install -g vercel

# 2. Deploy
vercel --prod
```

### 2. CDN Deployment

#### Cloudflare Pages
1. Connect your GitHub repository
2. Set build command: `echo "No build required"`
3. Set publish directory: `/`
4. Deploy automatically on push

#### AWS CloudFront + S3
```bash
# 1. Create S3 bucket
aws s3 mb s3://${BUCKET_NAME:-chitraharsha-yantra-prod-$(date +%s)}

# 2. Upload files
aws s3 sync . s3://chitraharsha-yantra-prod-bucket --exclude ".*" --exclude "node_modules/*"

# 3. Create CloudFront distribution
aws cloudfront create-distribution --distribution-config file://cloudfront-config.json
```

## Enterprise Deployment

### Docker Deployment

#### Dockerfile
```dockerfile
FROM nginx:alpine

# Copy static files
COPY . /usr/share/nginx/html

# Copy custom nginx config
COPY nginx.conf /etc/nginx/nginx.conf

# Security headers
COPY security-headers.conf /etc/nginx/conf.d/security-headers.conf

EXPOSE 80 443

CMD ["nginx", "-g", "daemon off;"]
```

#### docker-compose.yml
```yaml
version: '3.8'

services:
  yantra-web:
    build: .
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./ssl:/etc/nginx/ssl
      - ./logs:/var/log/nginx
    environment:
      - ENVIRONMENT=production
    restart: unless-stopped
    
  yantra-api:
    image: chitraharsha/yantra-api:latest
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
      - DATABASE_URL=${DATABASE_URL}
      - JWT_SECRET=${JWT_SECRET}
    restart: unless-stopped
    
  redis:
    image: redis:alpine
    ports:
      - "6379:6379"
    volumes:
      - redis-data:/data
    restart: unless-stopped

volumes:
  redis-data:
```

### Kubernetes Deployment

#### deployment.yaml
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: yantra-web
  labels:
    app: yantra-web
spec:
  replicas: 3
  selector:
    matchLabels:
      app: yantra-web
  template:
    metadata:
      labels:
        app: yantra-web
    spec:
      containers:
      - name: yantra-web
        image: chitraharsha/yantra-web:latest
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: "64Mi"
            cpu: "50m"
          limits:
            memory: "128Mi"
            cpu: "100m"
        env:
        - name: ENVIRONMENT
          value: "production"
---
apiVersion: v1
kind: Service
metadata:
  name: yantra-web-service
spec:
  selector:
    app: yantra-web
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
  type: LoadBalancer
```

#### ingress.yaml
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: yantra-ingress
  annotations:
    kubernetes.io/ingress.class: nginx
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/force-ssl-redirect: "true"
spec:
  tls:
  - hosts:
    - yantra.chitraharsha.com
    secretName: yantra-tls
  rules:
  - host: yantra.chitraharsha.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: yantra-web-service
            port:
              number: 80
```

## Cloud Provider Deployments

### AWS Deployment

#### Using AWS Amplify
```bash
# 1. Install Amplify CLI
npm install -g @aws-amplify/cli

# 2. Initialize Amplify
amplify init

# 3. Add hosting
amplify add hosting

# 4. Deploy
amplify publish
```

#### Using AWS ECS
```json
{
  "family": "yantra-web",
  "networkMode": "awsvpc",
  "requiresCompatibilities": ["FARGATE"],
  "cpu": "256",
  "memory": "512",
  "executionRoleArn": "arn:aws:iam::account:role/ecsTaskExecutionRole",
  "containerDefinitions": [
    {
      "name": "yantra-web",
      "image": "chitraharsha/yantra-web:latest",
      "portMappings": [
        {
          "containerPort": 80,
          "protocol": "tcp"
        }
      ],
      "essential": true,
      "logConfiguration": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/yantra-web",
          "awslogs-region": "us-east-1",
          "awslogs-stream-prefix": "ecs"
        }
      }
    }
  ]
}
```

### Google Cloud Platform

#### Using Cloud Run
```bash
# 1. Build and push container
gcloud builds submit --tag gcr.io/PROJECT-ID/yantra-web

# 2. Deploy to Cloud Run
gcloud run deploy yantra-web \
  --image gcr.io/PROJECT-ID/yantra-web \
  --platform managed \
  --region us-central1 \
  --allow-unauthenticated
```

#### Using App Engine
```yaml
# app.yaml
runtime: nodejs18

handlers:
- url: /.*
  static_files: index.html
  upload: index.html
  secure: always
  redirect_http_response_code: 301

- url: /(.*)
  static_files: \1
  upload: (.*)
  secure: always
```

### Microsoft Azure

#### Using Azure Static Web Apps
```bash
# 1. Install Azure CLI
az extension add --name staticwebapp

# 2. Create static web app
az staticwebapp create \
  --name yantra-web \
  --resource-group myResourceGroup \
  --source https://github.com/username/yantra \
  --location "Central US" \
  --branch main \
  --app-location "/" \
  --output-location "/"
```

## Configuration Files

### nginx.conf
```nginx
events {
    worker_connections 1024;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    
    # Security headers
    add_header X-Frame-Options "DENY" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
    add_header Referrer-Policy "strict-origin-when-cross-origin" always;
    add_header Content-Security-Policy "default-src 'self'; script-src 'self' 'unsafe-inline' https://cdn.tailwindcss.com https://unpkg.com; style-src 'self' 'unsafe-inline' https://fonts.googleapis.com; font-src 'self' https://fonts.gstatic.com;" always;
    
    # Gzip compression
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;
    
    # Cache static assets
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    server {
        listen 80;
        server_name _;
        root /usr/share/nginx/html;
        index index.html;
        
        # Handle SPA routing
        location / {
            try_files $uri $uri/ /index.html;
        }
        
        # Health check endpoint
        location /health {
            access_log off;
            return 200 "healthy\n";
            add_header Content-Type text/plain;
        }
    }
}
```

### Environment Configuration

#### .env.production
```bash
# API Configuration
REACT_APP_API_URL=https://api.chitraharsha.com/v1
REACT_APP_ENVIRONMENT=production

# Firebase Configuration
REACT_APP_FIREBASE_API_KEY=your-firebase-api-key
REACT_APP_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
REACT_APP_FIREBASE_PROJECT_ID=your-project-id

# Analytics
REACT_APP_GOOGLE_ANALYTICS_ID=GA-XXXXXXXXX
REACT_APP_MIXPANEL_TOKEN=your-mixpanel-token

# Feature Flags
REACT_APP_ENABLE_QUANTUM_FEATURES=true
REACT_APP_ENABLE_BIOMETRIC_AUTH=true
```

## SSL/TLS Configuration

### Let's Encrypt with Certbot
```bash
# Install certbot
sudo apt-get install certbot python3-certbot-nginx

# Obtain certificate
sudo certbot --nginx -d yantra.chitraharsha.com

# Auto-renewal
sudo crontab -e
# Add: 0 12 * * * /usr/bin/certbot renew --quiet
```

### Custom SSL Certificate
```nginx
server {
    listen 443 ssl http2;
    server_name yantra.chitraharsha.com;
    
    ssl_certificate /etc/nginx/ssl/yantra.crt;
    ssl_certificate_key /etc/nginx/ssl/yantra.key;
    
    # SSL Security
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384;
    ssl_prefer_server_ciphers off;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    
    # HSTS
    add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
}
```

## Monitoring and Logging

### Application Monitoring
```yaml
# docker-compose.monitoring.yml
version: '3.8'

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      
  grafana:
    image: grafana/grafana
    ports:
      - "3000:3000"
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=admin
    volumes:
      - grafana-data:/var/lib/grafana

  nginx-exporter:
    image: nginx/nginx-prometheus-exporter
    ports:
      - "9113:9113"
    command:
      - -nginx.scrape-uri=http://yantra-web/nginx_status

volumes:
  grafana-data:
```

### Log Aggregation
```yaml
# ELK Stack
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:8.5.0
    environment:
      - discovery.type=single-node
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ports:
      - "9200:9200"
      
  logstash:
    image: docker.elastic.co/logstash/logstash:8.5.0
    volumes:
      - ./logstash.conf:/usr/share/logstash/pipeline/logstash.conf
    ports:
      - "5044:5044"
      
  kibana:
    image: docker.elastic.co/kibana/kibana:8.5.0
    ports:
      - "5601:5601"
    environment:
      - ELASTICSEARCH_HOSTS=http://elasticsearch:9200
```

## Performance Optimization

### CDN Configuration
```javascript
// Cloudflare Workers script for edge optimization
addEventListener('fetch', event => {
  event.respondWith(handleRequest(event.request))
})

async function handleRequest(request) {
  const cache = caches.default
  const cacheKey = new Request(request.url, request)
  
  // Check cache first
  let response = await cache.match(cacheKey)
  
  if (!response) {
    // Fetch from origin
    response = await fetch(request)
    
    // Cache static assets
    if (request.url.includes('.js') || request.url.includes('.css')) {
      const headers = new Headers(response.headers)
      headers.set('Cache-Control', 'public, max-age=31536000')
      response = new Response(response.body, {
        status: response.status,
        statusText: response.statusText,
        headers: headers
      })
      
      event.waitUntil(cache.put(cacheKey, response.clone()))
    }
  }
  
  return response
}
```

### Image Optimization
```bash
# Install imagemin for build process
npm install --save-dev imagemin imagemin-webp imagemin-mozjpeg imagemin-pngquant

# Optimize images during build
npx imagemin assets/*.{jpg,png} --out-dir=assets/optimized --plugin=webp --plugin=mozjpeg --plugin=pngquant
```

## Security Hardening

### Security Headers
```nginx
# Additional security headers
add_header Permissions-Policy "geolocation=(), microphone=(), camera=()" always;
add_header Cross-Origin-Embedder-Policy "require-corp" always;
add_header Cross-Origin-Opener-Policy "same-origin" always;
add_header Cross-Origin-Resource-Policy "same-origin" always;
```

### Rate Limiting
```nginx
# Rate limiting configuration
http {
    limit_req_zone $binary_remote_addr zone=api:10m rate=10r/s;
    limit_req_zone $binary_remote_addr zone=login:10m rate=1r/s;
    
    server {
        location /api/ {
            limit_req zone=api burst=20 nodelay;
        }
        
        location /auth/login {
            limit_req zone=login burst=5 nodelay;
        }
    }
}
```

## Backup and Disaster Recovery

### Automated Backups
```bash
#!/bin/bash
# backup.sh

DATE=$(date +%Y%m%d_%H%M%S)
BACKUP_DIR="/backups/yantra"

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup application files
tar -czf $BACKUP_DIR/yantra_app_$DATE.tar.gz /var/www/yantra

# Backup database (if applicable)
pg_dump yantra_db > $BACKUP_DIR/yantra_db_$DATE.sql

# Upload to S3
aws s3 cp $BACKUP_DIR/ s3://yantra-backups/ --recursive

# Cleanup old backups (keep last 30 days)
find $BACKUP_DIR -name "*.tar.gz" -mtime +30 -delete
find $BACKUP_DIR -name "*.sql" -mtime +30 -delete
```

### Disaster Recovery Plan
1. **RTO (Recovery Time Objective)**: 4 hours
2. **RPO (Recovery Point Objective)**: 1 hour
3. **Backup Frequency**: Every 6 hours
4. **Geographic Redundancy**: 3 regions minimum
5. **Failover Process**: Automated with manual override

## Troubleshooting

### Common Issues

#### Build Failures
```bash
# Clear cache and rebuild
rm -rf node_modules package-lock.json
npm install
npm run build
```

#### SSL Certificate Issues
```bash
# Check certificate validity
openssl x509 -in certificate.crt -text -noout

# Test SSL configuration
openssl s_client -connect yantra.chitraharsha.com:443
```

#### Performance Issues
```bash
# Check resource usage
docker stats

# Monitor nginx logs
tail -f /var/log/nginx/access.log

# Check application metrics
curl http://localhost:9090/metrics
```

### Health Checks
```bash
#!/bin/bash
# health-check.sh

# Check application availability
if curl -f http://localhost/health > /dev/null 2>&1; then
    echo "Application is healthy"
else
    echo "Application is unhealthy"
    exit 1
fi

# Check SSL certificate expiry
EXPIRY=$(openssl x509 -enddate -noout -in /etc/nginx/ssl/yantra.crt | cut -d= -f2)
EXPIRY_EPOCH=$(date -d "$EXPIRY" +%s)
CURRENT_EPOCH=$(date +%s)
DAYS_LEFT=$(( ($EXPIRY_EPOCH - $CURRENT_EPOCH) / 86400 ))

if [ $DAYS_LEFT -lt 30 ]; then
    echo "SSL certificate expires in $DAYS_LEFT days"
fi
```

## Support and Maintenance

### Deployment Support
- **Email**: deployment@chitraharsha.com
- **Documentation**: https://docs.chitraharsha.com/deployment
- **Status Page**: https://status.chitraharsha.com

### Maintenance Windows
- **Scheduled Maintenance**: First Sunday of each month, 2-4 AM UTC
- **Emergency Maintenance**: As needed with 1-hour notice
- **Notification Channels**: Email, Slack, Status page

---

For enterprise deployment support and custom configurations, contact: enterprise@chitraharsha.com