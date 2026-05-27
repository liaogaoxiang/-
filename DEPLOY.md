# 云服务器部署说明

下面按 Ubuntu 云服务器部署。底表建议固定放在 `/app/data/latest.xlsx`，以后只要覆盖这个文件，刷新网页即可看到最新进展。

## 1. 准备服务器

登录服务器后安装 Python：

```bash
sudo apt update
sudo apt install -y python3 python3-pip python3-venv
```

## 2. 上传项目

在服务器创建目录：

```bash
sudo mkdir -p /app/progress-dashboard /app/data
sudo chown -R $USER:$USER /app/progress-dashboard /app/data
```

上传这些文件到 `/app/progress-dashboard`：

- `server.py`
- `index.html`
- `requirements.txt`

上传最新底表到：

```text
/app/data/latest.xlsx
```

## 3. 安装依赖

```bash
cd /app/progress-dashboard
python3 -m venv .venv
. .venv/bin/activate
pip install -r requirements.txt
```

## 4. 启动服务

```bash
cd /app/progress-dashboard
HOST=0.0.0.0 PORT=8765 DATA_FILE=/app/data/latest.xlsx .venv/bin/python server.py
```

如果云服务器安全组已开放 `8765` 端口，就可以访问：

```text
http://服务器公网IP:8765
```

## 5. 后台长期运行

创建 systemd 服务：

```bash
sudo tee /etc/systemd/system/progress-dashboard.service >/dev/null <<'EOF'
[Unit]
Description=Progress Dashboard
After=network.target

[Service]
WorkingDirectory=/app/progress-dashboard
Environment=HOST=0.0.0.0
Environment=PORT=8765
Environment=DATA_FILE=/app/data/latest.xlsx
ExecStart=/app/progress-dashboard/.venv/bin/python /app/progress-dashboard/server.py
Restart=always
RestartSec=3

[Install]
WantedBy=multi-user.target
EOF
```

启动并设置开机自启：

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now progress-dashboard
sudo systemctl status progress-dashboard
```

更新底表时，只需要覆盖：

```text
/app/data/latest.xlsx
```

然后刷新网页。

## 6. 可选：绑定域名

如果需要域名和 HTTPS，建议用 Nginx 反向代理到 `127.0.0.1:8765`，再用 Let's Encrypt 配证书。
