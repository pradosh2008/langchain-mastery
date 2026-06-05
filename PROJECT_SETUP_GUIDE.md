# 🚀 Project Setup Guide
> Follow this every time you create a new project in GitHub Codespaces.

---

## Step 1 — Create GitHub Repo
1. Go to [github.com](https://github.com) → **New repository**
2. Add a name (e.g. `langchain-mastery`)
3. Check **Add a README**
4. Click **Create repository**
5. Open in Codespace: **Code → Codespaces → Create codespace on main**

---

## Step 2 — Branch Setup (Enterprise Git Flow)
```bash
# Check you are on main
git branch

# Create dev branch
git checkout -b dev

# Create feature branch from dev
git checkout -b feature/<your-feature-name>
# e.g. git checkout -b feature/lesson-01-travel-agent

# Push feature branch to remote
git push -u origin feature/<your-feature-name>
```

**Branch strategy:**
```
main                          ← protected, production
  └── dev                     ← integration branch
        └── feature/xxx       ← your working branch
```

---

## Step 3 — Create .gitignore
```bash
cat > .gitignore << 'EOF'
.venv/
.env
__pycache__/
*.pyc
.ipynb_checkpoints/
*.egg-info/
dist/
.DS_Store
EOF

git add .gitignore
git commit -m "chore: add gitignore"
git push
```

---

## Step 4 — Create Project Structure
```bash
mkdir -p notebooks
touch .env requirements.txt README.md
```

**Project layout:**
```
project/
├── .env                  ← secrets (never commit)
├── .gitignore
├── requirements.txt      ← dependencies
├── README.md
├── .venv/                ← virtual environment (never commit)
└── notebooks/            ← Jupyter notebooks
```

---

## Step 5 — Create requirements.txt
```bash
cat > requirements.txt << 'EOF'
langchain
langchain-openai
langchain-core
langgraph
langgraph-prebuilt
azure-ai-projects
azure-identity
python-dotenv
openai
ipykernel
nest-asyncio
EOF
```

> ✏️ Modify packages based on your project needs.

---

## Step 6 — Create Virtual Environment & Install Kernel
```bash
# Create venv
python -m venv .venv

# Activate venv
source .venv/bin/activate

# Install dependencies
pip install -r requirements.txt -q

# Register kernel for Jupyter
python -m ipykernel install --user --name=ai-agents --display-name "AI Agents"
```

> In your notebook → top right corner → select kernel → **"AI Agents"**

---

## Step 7 — Set Up .env
```bash
# Edit .env and add your secrets
# NEVER commit this file
cat > .env << 'EOF'
AZURE_AI_PROJECT_ENDPOINT=https://<your-hub>.services.ai.azure.com/api/projects/<your-project>
AZURE_AI_MODEL_DEPLOYMENT_NAME=<your-deployment-name>
EOF
```

---

## Step 8 — Azure Login (Codespaces)
```bash
# Use device code flow — works reliably in Codespaces
az login --use-device-code
```
1. Open the URL shown in terminal in your **local browser**
2. Enter the code
3. Sign in with your Azure account
4. Return to terminal — login confirmed

---

## Step 9 — Commit Project Setup
```bash
git add requirements.txt README.md
git commit -m "chore: project setup"
git push
```

---

## Step 10 — Create Your Notebook
```bash
touch notebooks/lesson-01.ipynb
```
Open in VS Code → select **"AI Agents"** kernel → start coding.

---

## Step 11 — Daily Git Flow (While Working)
```bash
# Stage changes
git add .

# Commit with meaningful message
git commit -m "feat: add travel agent with tool calling"

# Push
git push
```

**Commit message conventions:**
| Prefix | When to use |
|--------|-------------|
| `feat:` | New feature or notebook |
| `fix:` | Bug fix |
| `chore:` | Setup, config, tooling |
| `docs:` | README or documentation |
| `refactor:` | Code restructure |

---

## Step 12 — Raise PR When Feature is Done
```bash
# On GitHub → Pull Requests → New Pull Request
# base: dev  ←  compare: feature/xxx
# Add title, description → Create PR → Merge
```

---

## ✅ Checklist
- [ ] Repo created on GitHub
- [ ] Opened in Codespace
- [ ] `dev` branch created
- [ ] `feature/xxx` branch created and pushed
- [ ] `.gitignore` committed
- [ ] `requirements.txt` created
- [ ] `.venv` created and kernel registered
- [ ] `.env` created (not committed)
- [ ] Azure login done
- [ ] Project setup committed
- [ ] Notebook created with correct kernel