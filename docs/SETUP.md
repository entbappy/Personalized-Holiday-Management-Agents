# Setup and Installation Guide

## Table of Contents

1. [System Requirements](#system-requirements)
2. [Pre-Installation](#pre-installation)
3. [Installation Methods](#installation-methods)
4. [Configuration](#configuration)
5. [Verification](#verification)
6. [Troubleshooting](#troubleshooting)
7. [Development Setup](#development-setup)

---

## System Requirements

### Minimum Requirements

- **OS**: Windows 10+, macOS 10.14+, Linux (Ubuntu 18.04+)
- **Python**: 3.10 or higher
- **Memory**: 4GB RAM minimum (8GB recommended)
- **Disk Space**: 500MB for dependencies
- **Internet**: Required for OpenAI API access

### Recommended Setup

- **OS**: Windows 11, macOS 12+, Ubuntu 20.04 LTS
- **Python**: 3.11 or 3.12
- **Memory**: 8GB RAM
- **CPU**: Multi-core processor
- **Disk**: SSD (faster package installation)

### Verify Python Installation

```bash
python --version
# Should output: Python 3.10.x or higher

python -m pip --version
# Should output: pip version and location
```

---

## Pre-Installation

### 1. Clone the Repository

```bash
git clone https://github.com/yourusername/Personalized-Holiday-Management-Agent.git
cd Personalized-Holiday-Management-Agent
```

### 2. Get OpenAI API Key

1. Go to [OpenAI API Platform](https://platform.openai.com)
2. Sign up or log in to your account
3. Navigate to API keys section
4. Create a new API key
5. Copy and save it securely (you'll need it later)

### 3. Prepare Environment File

Create a `.env` file in the project root:

```bash
# Windows
copy .env.example .env

# macOS/Linux
cp .env.example .env
```

Or create manually:

```bash
cat > .env << EOF
OPENAI_API_KEY=your_api_key_here
OPENAI_MODEL=gpt-4o
HOST=0.0.0.0
PORT=8000
DEBUG=True
LOG_LEVEL=INFO
EOF
```

---

## Installation Methods

### Method 1: Using pip (Standard)

#### Step 1: Create Virtual Environment

**Windows**:
```bash
python -m venv venv
venv\Scripts\activate
```

**macOS/Linux**:
```bash
python3 -m venv venv
source venv/bin/activate
```

#### Step 2: Upgrade pip

```bash
python -m pip install --upgrade pip
```

#### Step 3: Install Dependencies

```bash
pip install -r requirements.txt
```

#### Step 4: Verify Installation

```bash
pip list
# Should show all packages from requirements.txt
```

---

### Method 2: Using UV (Recommended ⚡ Faster)

UV is a blazing-fast Python package manager written in Rust. Installation is 10-100x faster!

#### Step 1: Install UV

**Windows**:
```bash
powershell -ExecutionPolicy BypassCurrentUser -c "irm https://astral.sh/uv/install.ps1 | iex"
```

**macOS**:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

**Linux**:
```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

#### Step 2: Create Virtual Environment with UV

```bash
uv venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
```

#### Step 3: Install Dependencies with UV

```bash
# Option A: Install from requirements.txt
uv pip install -r requirements.txt

# Option B: Install in editable mode (if using pyproject.toml)
uv pip install -e .
```

#### Step 4: Verify Installation

```bash
uv pip list
```

**Why UV?**
- ⚡ 10-100x faster installations
- 🔒 Better dependency resolution
- 📦 Lower memory footprint
- 🚀 Simplified workflow with `uv run`

---

### Method 3: Using conda

#### Step 1: Create Conda Environment

```bash
conda create -n holiday_agent python=3.12 -y
conda activate holiday_agent
```

#### Step 2: Install Dependencies

```bash
pip install -r requirements.txt
```

Or using conda channels:

```bash
conda install -c conda-forge -r requirements.txt
```

#### Step 3: Verify Installation

```bash
conda list
python --version
```

---

### Method 4: Using Poetry

If your project has a `pyproject.toml`:

#### Step 1: Install Poetry

```bash
# Windows
(Invoke-WebRequest -Uri https://install.python-poetry.org -UseBasicParsing).Content | python -

# macOS/Linux
curl -sSL https://install.python-poetry.org | python3 -
```

#### Step 2: Install Dependencies

```bash
poetry install
```

#### Step 3: Activate Virtual Environment

```bash
poetry shell
```

---

## Configuration

### 1. Environment Variables

Edit your `.env` file with the following variables:

```env
# OpenAI Configuration (REQUIRED)
OPENAI_API_KEY=sk-your-api-key-here
OPENAI_MODEL=gpt-4o

# Server Configuration
HOST=0.0.0.0
PORT=8000
RELOAD=true

# Logging
DEBUG=True
LOG_LEVEL=INFO

# Optional: API Keys for Future Integrations
BOOKING_API_KEY=
SKYSCANNER_API_KEY=
GOOGLE_MAPS_API_KEY=
```

### 2. Application Settings

Edit `holiday_management/config/settings.py`:

```python
from pydantic_settings import BaseSettings

class Settings(BaseSettings):
    openai_api_key: str
    openai_model: str = "gpt-4o"
    host: str = "0.0.0.0"
    port: int = 8000
    debug: bool = True
    
    class Config:
        env_file = ".env"

settings = Settings()
```

### 3. Logging Configuration

Create `logging.conf` (optional):

```ini
[loggers]
keys=root

[handlers]
keys=console,file

[formatters]
keys=standard

[logger_root]
level=INFO
handlers=console,file

[handler_console]
class=StreamHandler
level=DEBUG
formatter=standard
args=(sys.stdout,)

[handler_file]
class=FileHandler
level=DEBUG
formatter=standard
args=('holiday_agent.log',)

[formatter_standard]
format=%(asctime)s - %(name)s - %(levelname)s - %(message)s
```

---

## Verification

### 1. Test Python Installation

```bash
python -c "import sys; print(f'Python {sys.version}')"
```

### 2. Test Dependencies

```bash
python -c "import fastapi, pydantic, autogen_agentchat; print('All dependencies installed!')"
```

### 3. Test OpenAI Connection

```bash
python -c "from openai import OpenAI; print('OpenAI client ready')"
```

### 4. Start Application

```bash
# Using standard uvicorn
uvicorn app:app --reload

# Or using uv run
uv run uvicorn app:app --reload
```

### 5. Verify API is Running

```bash
# In another terminal
curl http://localhost:8000/

# Should return HTML content
```

### 6. Test Planning Endpoint

```bash
curl -X POST "http://localhost:8000/plan" \
  -H "Content-Type: application/json" \
  -d '{"content": "3-day trip to Paris"}'

# Should return agent responses
```

---

## Troubleshooting

### Common Issues & Solutions

#### Issue 1: Python Not Found

**Error**: `'python' is not recognized`

**Solutions**:
```bash
# Try python3
python3 --version

# Check installation
which python  # macOS/Linux
where python  # Windows

# Reinstall Python from python.org
```

#### Issue 2: Virtual Environment Not Activating

**Error**: `venv not recognized` or `source: command not found`

**Solutions**:
```bash
# Windows - Try PowerShell
& ".\venv\Scripts\Activate.ps1"

# macOS/Linux - Check permissions
chmod +x venv/bin/activate
source venv/bin/activate

# Recreate environment
rm -rf venv
python -m venv venv
```

#### Issue 3: OpenAI API Key Not Recognized

**Error**: `AuthenticationError: Invalid API key`

**Solutions**:
```bash
# 1. Verify .env file exists
ls -la .env  # macOS/Linux
dir .env     # Windows

# 2. Check format (no quotes needed)
OPENAI_API_KEY=sk-...

# 3. Verify environment variable is loaded
python -c "from dotenv import load_dotenv; import os; load_dotenv(); print(os.getenv('OPENAI_API_KEY'))"

# 4. Get new API key from openai.com if needed
```

#### Issue 4: Port Already in Use

**Error**: `Address already in use: ('0.0.0.0', 8000)`

**Solutions**:
```bash
# Use different port
uvicorn app:app --port 8001

# Kill process on port 8000
# Windows
netstat -ano | findstr :8000
taskkill /PID <PID> /F

# macOS/Linux
lsof -i :8000
kill -9 <PID>
```

#### Issue 5: Module Not Found

**Error**: `ModuleNotFoundError: No module named 'fastapi'`

**Solutions**:
```bash
# Activate virtual environment
source venv/bin/activate  # or equivalent for your system

# Reinstall all dependencies
pip install -r requirements.txt --force-reinstall

# Or using UV
uv pip install -r requirements.txt --force-reinstall
```

#### Issue 6: Permission Denied

**Error**: `PermissionError: [Errno 13] Permission denied`

**Solutions**:
```bash
# macOS/Linux - Fix permissions
chmod +x app.py
chmod +x holiday_management/*.py

# Windows - Run as Administrator or use WSL
```

#### Issue 7: CORS Issues

**Error**: `Access to XMLHttpRequest blocked by CORS`

**Solutions**:
Add to `app.py`:
```python
from fastapi.middleware.cors import CORSMiddleware

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],  # Restrict in production
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```

#### Issue 8: Slow Installation with pip

**Solution**: Use UV instead
```bash
# Much faster
uv pip install -r requirements.txt
```

---

## Development Setup

### Install Development Dependencies

```bash
# Using pip
pip install -r requirements.txt
pip install pytest pytest-cov black flake8 mypy

# Using UV
uv pip install -r requirements.txt pytest pytest-cov black flake8 mypy
```

### Code Formatting

```bash
# Format code with Black
black holiday_management/ app.py

# Check formatting
black --check holiday_management/
```

### Linting

```bash
# Check code quality
flake8 holiday_management/ app.py

# Type checking
mypy holiday_management/
```

### Running Tests

```bash
# Run all tests
pytest

# With coverage
pytest --cov=holiday_management

# Verbose output
pytest -v
```

### Git Hooks (Optional)

Create `.git/hooks/pre-commit`:

```bash
#!/bin/bash
black --check .
flake8 .
mypy .
pytest
```

Make executable:
```bash
chmod +x .git/hooks/pre-commit
```

---

## Docker Setup (Optional)

Create `Dockerfile`:

```dockerfile
FROM python:3.12-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

COPY . .

EXPOSE 8000

CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
```

Build and run:

```bash
# Build image
docker build -t holiday-agent .

# Run container
docker run -p 8000:8000 -e OPENAI_API_KEY=your_key holiday-agent
```

---

## Next Steps

1. **Read the Documentation**:
   - [Architecture Guide](ARCHITECTURE.md)
   - [API Documentation](API.md)

2. **Explore Examples**:
   - Check `all-utils/` folder for example notebooks

3. **Try the Interface**:
   - Visit `http://localhost:8000`
   - Test with sample trip requests

4. **Customize**:
   - Modify agent prompts in `holiday_management/agents/`
   - Adjust settings in `holiday_management/config/`

---

## Getting Help

- **Documentation**: Read the [docs](../) folder
- **Issues**: Check [GitHub Issues](https://github.com/yourusername/repo/issues)
- **Discussion**: Start a [GitHub Discussion](https://github.com/yourusername/repo/discussions)
- **Email**: support@example.com

---

## Quick Reference

| Task | Command |
|------|---------|
| Create environment (venv) | `python -m venv venv` |
| Activate (Windows) | `venv\Scripts\activate` |
| Activate (macOS/Linux) | `source venv/bin/activate` |
| Install dependencies | `pip install -r requirements.txt` |
| Install with UV | `uv pip install -r requirements.txt` |
| Start server | `uvicorn app:app --reload` |
| Run with UV | `uv run uvicorn app:app --reload` |
| Test API | `curl http://localhost:8000/plan -X POST -H "Content-Type: application/json" -d '{"content":"test"}'` |
| Run tests | `pytest` |
| Format code | `black .` |
| Check quality | `flake8 .` |

---

Last updated: April 2026
