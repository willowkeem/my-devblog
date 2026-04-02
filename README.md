# Willow Kim — Dev Blog

A static personal blog deployed on AWS EC2 with Nginx. Built with plain HTML and CSS — no frameworks, no build tools.

**Live:** http://54.246.17.88

---

## 1. Project Overview

This project documents my journey as a software development student. The goal was to go beyond local development and ship something real: provision a cloud server, configure a web server, and deploy a static site manually — understanding every step along the way.

The blog itself is a single-page app written in vanilla HTML/CSS/JS. Posts are stored in `localStorage` and support Markdown-style formatting.

---

## 2. Architecture

```
Browser
  │
  └──▶ Elastic IP (54.246.17.88:80)
         │
         └──▶ AWS EC2 t3.micro (Ubuntu 22.04 LTS)
                │
                └──▶ Nginx (serves /var/www/html)
                       │
                       ├── index.html
                       └── write.html
```

| Component | Detail |
|-----------|--------|
| Cloud provider | AWS (eu-west-1) |
| Instance type | t3.micro |
| OS | Ubuntu 22.04 LTS |
| Web server | Nginx |
| IP | Elastic IP — 54.246.17.88 |
| Deployment | SCP (manual file transfer) |

---

## 3. Deployment Steps

**1. Launch EC2 instance**
- AMI: Ubuntu 22.04 LTS
- Instance type: t3.micro
- Security group: allow SSH (22) and HTTP (80) inbound

**2. Attach an Elastic IP**
- Allocate a new Elastic IP in EC2 → Elastic IPs
- Associate it with the instance so the IP doesn't change on restart

**3. SSH into the instance and install Nginx**
```bash
ssh -i your-key.pem ubuntu@54.246.17.88
sudo apt update && sudo apt install nginx -y
sudo systemctl enable nginx
```

**4. Deploy files with SCP**
```bash
scp -i your-key.pem index.html write.html ubuntu@54.246.17.88:/var/www/html/
```

**5. Verify**  
Open `http://54.246.17.88` in a browser.

---

## 4. Troubleshooting

### IP address changed after restarting the instance
AWS assigns a new public IP every time a regular EC2 instance is stopped and started. The fix is to allocate an **Elastic IP** and associate it with the instance — the address then stays fixed regardless of how many times the instance is restarted.

### Instance not showing up in EC2 console
The EC2 console filters by region. The instance appeared missing because the console was set to a different region. Switching to the correct region (eu-west-1 / Ireland) made it visible again.

### SCP command failing due to spaces in the file path
Running `scp` with a local path that contained spaces caused the command to fail, as the shell split it into separate arguments. The fix was to wrap the path in quotes:
```bash
scp -i "~/path/to/my key.pem" index.html ubuntu@54.246.17.88:/var/www/html/
```

---

## 5. What I Learned

- **Cloud infrastructure basics** — how EC2 instances, security groups, and Elastic IPs fit together, and why static IPs matter for production.
- **Linux fundamentals in practice** — file permissions, `systemctl`, and navigating a remote server over SSH felt abstract before this project; now they're second nature.
- **Nginx configuration** — understanding how Nginx serves static files and how to read its error logs when something goes wrong.
- **Debugging without a GUI** — when things broke, there was no error popup. Learning to check `nginx -t`, `systemctl status nginx`, and `/var/log/nginx/error.log` was the real skill.
- **The gap between localhost and production** — things that work perfectly on `file://` can behave differently when served over HTTP. Shipping to a real server exposed assumptions I didn't know I was making.

---

*Built by Willow Kim · Software Development student (Higher Diploma, NFQ Level 8)*
