# UGC SaaS

**Deskripsi:** User-Generated Content SaaS Platform  
**URL:** https://ugc.arzastore.com  
**Port:** 3400  
**Tech Stack:** Node.js + Next.js + PostgreSQL

## Fitur
- UGC (User-Generated Content) management platform
- Content moderation & approval workflow
- User dashboard & submission portal
- Admin panel
- Media upload & storage

## Database
- Type: PostgreSQL
- Database: `ugc_saas`
- User: `ugc`
- Backup: `backups/ugc_saas_*.sql.gz` (5 terakhir)

## URL & Access
- Public: https://ugc.arzastore.com
- Admin: https://ugc.arzastore.com/admin (check credentials di .env)

## Backup
Otomatis **00:00 WIB** setiap hari:
- Source code → GitHub
- Postgres dump → `backups/ugc_saas_YYYY-MM-DD_HHMM.sql.gz`

## Restore di VPS Baru

### 1. Clone
```bash
cd /opt
git clone git@github.com:arkhindi12/ugc-saas.git
cd ugc-saas
```

### 2. Database restore
```bash
# Create DB & user
sudo -u postgres createdb ugc_saas
sudo -u postgres createuser ugc
sudo -u postgres psql -c "ALTER USER ugc WITH PASSWORD '***';"
sudo -u postgres psql -c "GRANT ALL ON DATABASE ugc_saas TO ugc;"

# Restore latest backup
gunzip -c backups/ugc_saas_*.sql.gz | sudo -u ugc psql ugc_saas
```

### 3. Install
```bash
npm install
# atau
pnpm install
```

### 4. Environment
```bash
cp .env.example .env
nano .env
# Set DATABASE_URL=postgresql://ugc:***@localhost:5432/ugc_saas
# Set PORT=3400, UPLOAD_DIR, ADMIN credentials
```

### 5. Build
```bash
npm run build
```

### 6. Start
```bash
pm2 start server.js --name ugc-saas
# atau jika ada ecosystem.config.js
pm2 start ecosystem.config.js
pm2 save
```

### 7. Nginx config
```bash
sudo nano /etc/nginx/sites-available/ugc.arzastore.com
```

Config:
```nginx
server {
    listen 80;
    server_name ugc.arzastore.com;
    
    location / {
        proxy_pass http://127.0.0.1:3400;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

```bash
sudo ln -s /etc/nginx/sites-available/ugc.arzastore.com /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

### 8. SSL
```bash
sudo certbot --nginx -d ugc.arzastore.com
```

### 9. Verify
```bash
pm2 logs ugc-saas
curl http://localhost:3400
curl https://ugc.arzastore.com
```

## Notes
- Check `public/` untuk static assets
- Media uploads mungkin butuh storage config (S3/local)

---
*Auto-backup enabled. Last update: 2026-06-06*
