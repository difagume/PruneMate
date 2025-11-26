# PruneMate

<p align="center">
  <img width="400" height="400" alt="prunemate-logo" src="static/prunemate.png" />
</p>

<h1 align="center">PruneMate</h1>
<p align="center"><em>Docker Cleanup Helper - Automated & Scheduled</em></p>

<p align="center">
  <img src="https://img.shields.io/badge/version-1.2.2-blue?style=flat-square"/>
  <img src="https://img.shields.io/badge/python-3.10%2B-green?style=flat-square"/>
  <img src="https://img.shields.io/badge/docker-compose-0db7ed?style=flat-square"/>
  <img src="https://img.shields.io/badge/license-MIT-yellow?style=flat-square"/>
</p>

A sleek web interface to **automatically clean up Docker resources** on a schedule. Built with Python (Flask) · Docker SDK · APScheduler · Gunicorn

**Keep your Docker host tidy with scheduled cleanup of unused images, containers, networks, and volumes.**

---

## 🔗 Quick links

- 📦 **Docker Hub:** (coming soon)
- 🐙 **GitHub:** <https://github.com/anoniemerd/PruneMate>
- 📸 **Screenshots:** See below

---

## ✨ Features

- 🕐 **Flexible scheduling** - Daily, Weekly, or Monthly cleanup runs
- 🌍 **Timezone aware** - Configure your local timezone for accurate scheduling
- 🕒 **12/24-hour time format** - Choose your preferred time display
- 🧹 **Selective cleanup** - Choose what to prune: containers, images, networks, volumes
- 🔔 **Smart notifications** - Gotify or ntfy.sh support with optional change-only alerts
- 🎨 **Modern UI** - Dark theme with smooth animations and responsive design
- 🔒 **Safe & controlled** - Manual trigger option and detailed logging
- 📊 **Detailed reports** - See exactly what was cleaned and how much space was reclaimed

---

## 📷 Screenshots

<p align="center">
  <img alt="prunemate-schedule" src="docs/screenshot-schedule.png" />
</p>

<p align="center">
  <img alt="prunemate-notifications" src="docs/screenshot-notifications.png" />
</p>

---

## 🚀 Quick Start with Docker Compose

### Prerequisites

- Docker and Docker Compose installed
- Access to Docker socket

### Installation

1. **Clone the repository:**

```bash
git clone https://github.com/anoniemerd/PruneMate.git
cd PruneMate
```

2. **Configure your settings** (optional):

Edit `docker-compose.yaml` to set your timezone and preferences:

```yaml
environment:
  - PRUNEMATE_TZ=Europe/Amsterdam     # Your timezone
  - PRUNEMATE_TIME_24H=true           # true for 24h, false for 12h (AM/PM)
```

3. **Start PruneMate:**

```bash
docker-compose up -d
```

4. **Access the dashboard:**

```
http://<your-server-ip>:7676/
```

---

## 🐳 Docker Compose Configuration

```yaml
services:
  prunemate:
    build: .
    container_name: prunemate
    ports:
      - "7676:8080"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock  # Required for Docker access
      - ./logs:/var/log                             # Persistent logs
      - ./config:/config                            # Persistent configuration
    environment:
      - PRUNEMATE_TZ=Europe/Amsterdam               # Your timezone
      - PRUNEMATE_TIME_24H=true                     # Time format preference
    restart: unless-stopped
```

---

## ⚙️ Configuration

### Environment Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `PRUNEMATE_TZ` | `UTC` | Timezone for scheduling (e.g., `Europe/Amsterdam`, `America/New_York`) |
| `PRUNEMATE_TIME_24H` | `true` | Time format: `true` for 24-hour, `false` for 12-hour (AM/PM) |
| `PRUNEMATE_CONFIG` | `/config/config.json` | Path to configuration file |
| `PRUNEMATE_SECRET` | `prunemate-secret-key` | Flask secret key (change in production) |

### Web Interface Settings

Access the web interface at `http://<your-server-ip>:7676/` to configure:

**Schedule:**
- Frequency: Daily, Weekly, or Monthly
- Time: When to run the cleanup
- Day: Day of week (weekly) or day of month (monthly)

**Cleanup Options:**
- All unused containers
- All unused images
- All unused networks
- All unused volumes

**Notifications:**
- Provider: Gotify or ntfy.sh
- Configuration: URL, token/topic
- Only notify on changes (optional)

---

## 🧠 How it works

1. **Scheduler runs** every minute checking if it's time to execute
2. **Loads latest config** from persistent storage
3. **Executes Docker prune** commands for selected resource types
4. **Collects statistics** on what was removed and space reclaimed
5. **Sends notification** (if configured and enabled)
6. **Logs everything** with timezone-aware timestamps

### File Structure

```
/config/
├── config.json          # Your configuration (persistent)
├── prunemate.lock       # Prevents concurrent runs
└── last_run_key         # Tracks last successful run

/var/log/
└── prunemate.log        # Application logs (rotating, 5MB max)
```

---

## 🔧 Managing PruneMate

### View Logs

```bash
docker logs -f prunemate
```

Or check the persistent log file:

```bash
tail -f logs/prunemate.log
```

### Restart

```bash
docker-compose restart
```

### Update

```bash
docker-compose pull
docker-compose up -d
```

### Stop

```bash
docker-compose down
```

---

## 🔔 Notification Setup

### Gotify

1. Install Gotify server
2. Create an application token
3. Configure in PruneMate:
   - Provider: Gotify
   - URL: `https://your-gotify-server.com`
   - Token: Your application token

### ntfy.sh

1. Choose a unique topic name
2. Configure in PruneMate:
   - Provider: ntfy
   - URL: `https://ntfy.sh` (or self-hosted)
   - Topic: Your chosen topic name

Subscribe to your topic:
```bash
# Web: https://ntfy.sh/your-topic
# Mobile: Install ntfy app and subscribe to your-topic
```

---

## 🧩 Advanced Configuration

### Custom Port

Change the port in `docker-compose.yaml`:

```yaml
ports:
  - "8080:8080"  # Change first number to your preferred port
```

### Timezone List

Common timezone examples:
- `Europe/Amsterdam`
- `Europe/London`
- `America/New_York`
- `America/Los_Angeles`
- `Asia/Tokyo`
- `Australia/Sydney`

Full list: <https://en.wikipedia.org/wiki/List_of_tz_database_time_zones>

### 12-Hour Time Format

Set `PRUNEMATE_TIME_24H=false` for AM/PM time display:
- UI shows time picker with hour (1-12), minutes, and AM/PM selector
- Notifications display times like "3:00 AM" or "5:30 PM"
- Internal storage remains 24-hour format for consistency

---

## 🧠 Troubleshooting

| Problem | Solution |
|--------|----------|
| ❌ Can't access web interface | Check if port 7676 is open and not blocked by firewall |
| ⚙️ Container not starting | Check logs: `docker logs prunemate` |
| 🔒 Permission denied errors | Ensure Docker socket is accessible: `/var/run/docker.sock` |
| 🕐 Wrong timezone | Set `PRUNEMATE_TZ` environment variable to your timezone |
| 📧 Notifications not working | Verify notification provider settings and network connectivity |
| 🗂️ Config not persisting | Ensure `./config` volume is mounted correctly |

### Debug Mode

To see detailed logs:

```bash
docker logs -f prunemate
```

The log shows:
- Scheduler heartbeats (every minute)
- Configuration changes
- Prune job executions
- Notification delivery status

---

## 🔐 Security Notes

- **Docker socket access:** PruneMate needs access to `/var/run/docker.sock` to manage Docker resources
- **Network exposure:** By default, the web interface is exposed on port 7676. Use a reverse proxy with authentication for production
- **Secrets:** Store sensitive tokens in environment variables or Docker secrets
- **Updates:** Keep PruneMate updated to get security patches

---

## 🛠️ Development

### Requirements

- Python 3.10+
- Docker SDK for Python
- Flask, APScheduler, Gunicorn

### Local Development

```bash
# Install dependencies
pip install -r requirements.txt

# Run locally
python prunemate.py
```

### Build Docker Image

```bash
docker build -t prunemate:latest .
```

---

## 📝 Changelog

### Version 1.2.2
- ✨ Added 12/24-hour time format support
- 🌍 Improved timezone handling across all components
- 🎨 Enhanced UI with better time picker for 12-hour mode
- 🐛 Fixed config synchronization across workers
- ⚡ Reduced workers to 1 for simplified architecture
- 📝 Silent config loading to reduce log noise
- 🔧 Better input validation and clamping

### Version 1.2.1
- 🐛 Fixed scheduler not triggering at configured times
- 🔄 Config now reloads before scheduled checks
- 🔒 Added thread-safe config saving

### Version 1.2.0
- 🔔 Added notification support (Gotify & ntfy.sh)
- 🎨 Redesigned UI with modern dark theme
- 📊 Enhanced statistics and reporting

---

## 👤 Author & License

- Author: **Anoniemerd** — <https://github.com/anoniemerd>
- Repository: <https://github.com/anoniemerd/PruneMate>

Released under the **MIT License**.  
© 2025 – PruneMate Project.

---

## 🙏 Acknowledgments

Built with:
- [Flask](https://flask.palletsprojects.com/) - Web framework
- [APScheduler](https://apscheduler.readthedocs.io/) - Job scheduling
- [Docker SDK](https://docker-py.readthedocs.io/) - Docker integration
- [Gunicorn](https://gunicorn.org/) - WSGI server

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📬 Support

- 🐛 **Bug reports:** Open an issue on GitHub
- 💡 **Feature requests:** Open an issue on GitHub
- 💬 **Questions:** Start a discussion on GitHub

---

**Keep your Docker host clean with PruneMate! 🧹✨**
