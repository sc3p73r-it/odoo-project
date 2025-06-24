# How To Usage?

### Project Structure
```
odoo-docker/
├── docker-compose.yml
├── nginx/
│   └── default.conf
└── odoo/
    └── Dockerfile
```
### Create docker-compose.yml

```
version: '3.8'

services:
  db:
    image: postgres:15
    container_name: odoo-db
    environment:
      POSTGRES_DB: odoo
      POSTGRES_USER: odoo
      POSTGRES_PASSWORD: odoo
    volumes:
      - odoo-db-data:/var/lib/postgresql/data
    restart: unless-stopped

  odoo:
    build: ./odoo
    container_name: odoo-app
    depends_on:
      - db
    environment:
      HOST: db
      USER: odoo
      PASSWORD: odoo
    command: >
      odoo
      --db_host=db
      --db_user=odoo
      --db_password=odoo
      -d odoo
      --init=base
      --without-demo=all
      --xmlrpc-interface=0.0.0.0
    ports:
      - "8069:8069"
    volumes:
      - odoo-web-data:/var/lib/odoo
      - ./odoo/custom-addons:/mnt/extra-addons
    restart: unless-stopped

  nginx:
    image: nginx:alpine
    container_name: odoo-nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/default.conf:/etc/nginx/conf.d/default.conf
    depends_on:
      - odoo
    restart: unless-stopped

volumes:
  odoo-db-data:
  odoo-web-data:
```
###  Create odoo/Dockerfile
```
FROM odoo:17.0

USER root

# (Optional) Install additional packages
RUN apt-get update && apt-get install -y \
    nano \
    net-tools \
    && rm -rf /var/lib/apt/lists/*

USER odoo
```
### nginx/default.conf
```
server {
    listen 80;

    server_name _;

    location / {
        proxy_pass http://odoo:8069;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /longpolling {
        proxy_pass http://odoo:8072;
    }
}
```
### Run the Setup
```
docker-compose up --build -d
```
### Login Odoo on Browser
```
http://localhost
```
