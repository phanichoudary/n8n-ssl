# Ennetan (n8n) Docker Setup with Nginx Reverse Proxy and SSL

This repository contains a production-ready Docker Compose setup for running n8n (Ennetan) with automatic HTTPS via Let's Encrypt.

## Features

- ✅ Nginx reverse proxy with automatic SSL certificate management
- ✅ Let's Encrypt SSL certificates (auto-renewal)
- ✅ n8n workflow automation platform
- ✅ PostgreSQL database for persistence
- ✅ Automatic container restart on failure
- ✅ WebSocket support for real-time features

## Prerequisites

- Docker and Docker Compose installed on your server
- A domain name pointing to your server's IP address
- Ports 80 and 443 open on your firewall

## Quick Start

### 1. Clone this repository

```bash
git clone https://github.com/YOUR_USERNAME/ennetan-docker-setup.git
cd ennetan-docker-setup
```

### 2. Configure environment variables

```bash
# Copy the example environment file
cp .env.example .env

# Edit the .env file with your actual values
nano .env
```

**Important:** Update these values in `.env`:
- `DOMAIN` - Your domain name (e.g., `ennetan.yourdomain.com`)
- `LETSENCRYPT_EMAIL` - Your email for SSL certificate notifications
- `POSTGRES_PASSWORD` - A strong database password
- `POSTGRES_NON_ROOT_PASSWORD` - Another strong password
- `N8N_ENCRYPTION_KEY` - Generate with: `openssl rand -base64 32`

### 3. Start the services

```bash
# Start all containers
docker-compose up -d

# View logs
docker-compose logs -f

# Check status
docker-compose ps
```

### 4. Access your n8n instance

Visit `https://your-domain.com` in your browser. The first time you access it, you'll create your admin account.

## Project Structure

```
.
├── docker-compose.yml      # Main Docker Compose configuration
├── .env                    # Environment variables (create from .env.example)
├── .env.example           # Environment variables template
├── nginx-custom.conf      # Custom Nginx configuration
└── README.md              # This file
```

## Management Commands

### Start services
```bash
docker-compose up -d
```

### Stop services
```bash
docker-compose down
```

### Restart services
```bash
docker-compose restart
```

### View logs
```bash
# All services
docker-compose logs -f

# Specific service
docker-compose logs -f n8n
docker-compose logs -f nginx-proxy
```

### Update n8n to latest nightly
```bash
docker-compose pull n8n
docker-compose up -d n8n
```

### Backup database
```bash
docker-compose exec postgres pg_dump -U n8n n8n > backup_$(date +%Y%m%d_%H%M%S).sql
```

### Restore database
```bash
cat backup_file.sql | docker-compose exec -T postgres psql -U n8n -d n8n
```

## Security Recommendations

1. **Encryption Key**: Keep your `N8N_ENCRYPTION_KEY` secure and backed up. Losing it means losing access to encrypted credentials.

2. **Database Passwords**: Use strong, unique passwords for database access.

3. **Basic Auth** (Optional): Uncomment and configure basic auth in `.env` for an additional security layer:
   ```
   N8N_BASIC_AUTH_ACTIVE=true
   N8N_BASIC_AUTH_USER=admin
   N8N_BASIC_AUTH_PASSWORD=your_secure_password
   ```

4. **Firewall**: Ensure only ports 80 and 443 are open to the internet.

5. **Updates**: Regularly update your containers:
   ```bash
   docker-compose pull
   docker-compose up -d
   ```

## Troubleshooting

### SSL Certificate Issues

If Let's Encrypt fails to issue certificates:

1. Ensure your domain points to the server's IP
2. Check ports 80 and 443 are accessible
3. View Let's Encrypt logs: `docker-compose logs letsencrypt`
4. Let's Encrypt has rate limits (5 certificates per domain per week)

### n8n Not Accessible

1. Check if containers are running: `docker-compose ps`
2. View n8n logs: `docker-compose logs n8n`
3. Verify environment variables in `.env`
4. Ensure PostgreSQL is healthy: `docker-compose exec postgres pg_isready`

### Database Connection Issues

```bash
# Check PostgreSQL logs
docker-compose logs postgres

# Verify database exists
docker-compose exec postgres psql -U n8n -d n8n -c "\l"
```

## Data Persistence

All data is stored in Docker volumes:
- `n8n-data`: n8n workflows and settings
- `n8n-files`: Uploaded files
- `postgres-data`: Database data
- `nginx-certs`: SSL certificates

To backup all data:
```bash
docker run --rm -v ennetan-docker-setup_n8n-data:/data -v $(pwd):/backup alpine tar czf /backup/n8n-backup.tar.gz /data
```

## Custom Domain Configuration

To use a subdomain (recommended):

1. Create an A record: `ennetan.yourdomain.com` → Your server IP
2. Update `.env`: `DOMAIN=ennetan.yourdomain.com`
3. Restart: `docker-compose up -d`

## Environment Variables Reference

| Variable | Description | Required |
|----------|-------------|----------|
| DOMAIN | Your domain name | Yes |
| LETSENCRYPT_EMAIL | Email for SSL notifications | Yes |
| POSTGRES_DB | Database name | Yes |
| POSTGRES_USER | Database user | Yes |
| POSTGRES_PASSWORD | Database password | Yes |
| N8N_ENCRYPTION_KEY | Encryption key for credentials | Yes |
| TIMEZONE | Server timezone | No |

## Support

For n8n-specific issues, visit the [n8n documentation](https://docs.n8n.io/) or [community forum](https://community.n8n.io/).

For issues with this setup, please open an issue in this repository.

## License

This configuration is provided as-is under the MIT License.
