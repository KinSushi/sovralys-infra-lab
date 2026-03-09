# Data Science Environment — Ubuntu Host Stack

Jedha Bootcamp 2026 (RNCP6 + RNCP7 Data Science & AI) stack running natively on the Ubuntu host.

---

## Why on the Ubuntu Host (Not in the VM)

- Native performance (no virtualization overhead)
- Docker available natively (required for Jedha Bootcamp)
- Same Unix environment as macOS (Jedha reference)
- VM reserved for MT5 stability — no contamination from development tools

---

## Python via pyenv (Recommended over System Python)

**Dependencies:**
```bash
sudo apt install -y build-essential libssl-dev zlib1g-dev libbz2-dev \
  libreadline-dev libsqlite3-dev curl libncursesw5-dev xz-utils tk-dev \
  libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```

**Install pyenv:**
```bash
curl https://pyenv.run | bash
# Add to ~/.bashrc:
# export PYENV_ROOT="$HOME/.pyenv"
# export PATH="$PYENV_ROOT/bin:$PATH"
# eval "$(pyenv init -)"
```

**Install Python 3.11.9:**
```bash
pyenv install 3.11.9
pyenv global 3.11.9
python --version  # → Python 3.11.9
```

---

## Stack Installation

```bash
# Core ML/DS
pip3 install jupyter pandas numpy matplotlib scikit-learn seaborn plotly --break-system-packages

# Deep Learning
pip3 install tensorflow torch torchvision xgboost lightgbm --break-system-packages

# Data Engineering
pip3 install sqlalchemy psycopg2-binary mlflow --break-system-packages

# AWS CLI
pip3 install awscli --break-system-packages
```

---

## PostgreSQL

```bash
sudo apt install -y postgresql postgresql-contrib
sudo systemctl enable postgresql
```

---

## Jupyter as systemd Service

File `/etc/systemd/system/jupyter.service`:

```ini
[Unit]
Description=Jupyter Notebook Service
After=network.target

[Service]
Type=simple
User=$USER
ExecStart=/home/$USER/.local/bin/jupyter notebook --ip=0.0.0.0 --port=8888 --no-browser
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable jupyter
sudo systemctl start jupyter
```

**If Jupyter starts on port 8889 instead of 8888** (old process occupying 8888):
```bash
sudo pkill -f jupyter
sudo systemctl restart jupyter
```

**Access:**
```bash
sudo systemctl status jupyter | grep token
# Browser: http://YOUR_SERVER_IP:8888?token=...
```

---

## Stack Versions

| Tool | Version | Status |
|---|---|---|
| Python | 3.11.9 | ✅ Active (pyenv) |
| Jupyter | Latest | ✅ systemd service |
| Pandas | Latest | ✅ Installed |
| NumPy | Latest | ✅ Installed |
| Scikit-learn | Latest | ✅ Installed |
| TensorFlow | Latest | ✅ Installed |
| PyTorch | Latest | ✅ Installed |
| XGBoost | Latest | ✅ Installed |
| LightGBM | Latest | ✅ Installed |
| MLflow | Latest | ✅ Installed |
| PostgreSQL | System | ✅ Active |
| Docker | System | ✅ Active |
| AWS CLI | Latest | ✅ Installed |

---

## Jedha Bootcamp Alignment

| Module | Tools | Status |
|---|---|---|
| Essentials | Python, Pandas, Jupyter | ✅ Ready |
| Fullstack | scikit-learn, SQL, Docker, AWS | ✅ Ready |
| Lead | PyTorch, MLflow, Airflow | ✅ Installed |

> **Note on AWS:** Do not create an AWS account before the bootcamp starts (April 13, 2026) — maximize the free trial period.
