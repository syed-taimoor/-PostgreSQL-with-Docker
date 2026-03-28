# 🐘 PostgreSQL with Docker — Complete Setup Guide

> A step-by-step guide to running PostgreSQL in Docker with persistent storage, environment configuration, and Node.js integration.

---

## Prerequisites

Make sure Docker is installed and running on your server.

```bash
docker --version
docker info
```

---

## Step 1 — Create Persistent Data Directory

Create a directory on the host to store PostgreSQL data. This ensures your data **survives container restarts and deletions**.

```bash
mkdir -p /var/lib/postgresql/data
```

---

## Step 2 — Pull PostgreSQL Image

```bash
docker pull postgres:16
```

---

## Step 3 — Run the Container

```bash
docker run -d \
  --name postgres \
  --restart unless-stopped \
  -e POSTGRES_PASSWORD=your_strong_password \
  -e POSTGRES_USER=admin \
  -e POSTGRES_DB=mydb \
  -p 5432:5432 \
  -v /var/lib/postgresql/data:/var/lib/postgresql/data \
  postgres:16
```

### Flag Reference

| Flag | Description |
|------|-------------|
| `-d` | Run in background (detached mode) |
| `--restart unless-stopped` | Auto-restart on server reboot |
| `-e POSTGRES_PASSWORD` | Set the database password |
| `-e POSTGRES_USER` | Set the database user |
| `-e POSTGRES_DB` | Create a default database |
| `-p 5432:5432` | Expose port to host |
| `-v /var/lib/...` | Mount host directory for persistent storage |

---

## Step 4 — Verify It's Running

```bash
docker ps
```

You should see your `postgres` container listed with status `Up`.

---

## Step 5 — Install PostgreSQL Client (Node.js)

Install the `pg` package in your project:

```bash
yarn add pg
```

Or with npm:

```bash
npm install pg
```

---

## Step 6 — Connect From Your App

### Environment Variables

Add the following to your `.env` file:

```env
HOST=your_server_ip or localhost(when you have to install Docker locally)
POSTGRES_USER=admin
POSTGRES_PASSWORD=your_strong_password
POSTGRES_DB=mydb
```

### Node.js Connection Example

```javascript
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.HOST,
  user: process.env.POSTGRES_USER,
  password: process.env.POSTGRES_PASSWORD,
  database: process.env.POSTGRES_DB,
  port: 5432,
});

// Test connection
pool.query('SELECT NOW()', (err, res) => {
  if (err) {
    console.error('Connection error:', err);
  } else {
    console.log('Connected to PostgreSQL:', res.rows[0]);
  }
});
```

### With TypeScript / ES Modules

```typescript
import { Pool } from 'pg';
import dotenv from 'dotenv';

dotenv.config();

const pool = new Pool({
  host: process.env.HOST,
  user: process.env.POSTGRES_USER,
  password: process.env.POSTGRES_PASSWORD,
  database: process.env.POSTGRES_DB,
  port: 5432,
});

export default pool;
```

---

## Useful Management Commands

```bash
# Stop the container
docker stop postgres

# Start it again (data is preserved!)
docker start postgres

# View logs
docker logs postgres

# Restart the container
docker restart postgres
```

---

## Firewall (Optional but Recommended)

If your server is internet-facing, restrict port `5432` to trusted IPs only:

```bash
ufw allow from YOUR_IP to any port 5432
```

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `Connection refused` | Check container is running: `docker ps` |
| `Password authentication failed` | Verify `POSTGRES_PASSWORD` in your `.env` matches the one used in `docker run` |
| `Data not persisting` | Ensure the `-v` volume flag is correct and the host directory exists |
| `Port already in use` | Change host port: `-p 5433:5432` and update `PORT` in `.env` |

---

## Project Structure Suggestion

```
your-project/
├── .env                  # Environment variables (never commit this!)
├── .env.example          # Template for other developers
├── src/
│   └── db.js             # PostgreSQL pool connection
└── docker-compose.yml    # Optional: manage with Compose
```

### `.env.example` (commit this to GitHub)

```env
HOST=your_server_ip
POSTGRES_USER=your_db_user
POSTGRES_PASSWORD=your_db_password
POSTGRES_DB=your_db_name
```

---

> **Security tip:** Never commit your `.env` file. Add it to `.gitignore` immediately.

```bash
echo ".env" >> .gitignore
```
