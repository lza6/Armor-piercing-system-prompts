# ❓ FAQ

## Backend fails to start?

**Checklist**
- Confirm Python version >= 3.11
- Confirm all dependencies installed: `pip install -r requirements.txt`
- Check if port 8001 is in use (configurable in `config/main.yaml`)
- Check `.env` file configuration

**Solutions**
- **Change port**: Edit `config/main.yaml` server.backend_port
- **Check logs**: Review terminal error messages

---

## Port occupied after Ctrl+C?

**Problem**

After pressing Ctrl+C during a running task (e.g., deep research), restarting shows "port already in use" error.

**Cause**

Ctrl+C sometimes only terminates the frontend process while the backend continues running in the background.

**Solution**

```bash
# macOS/Linux
kill -9 $(lsof -t -i :8001)

# Windows
netstat -ano | findstr :8001
taskkill /PID <PID> /F
```

Then restart the service with `python scripts/start_web.py`.

---

## npm: command not found error?

**Problem**

Running `scripts/start_web.py` shows `npm: command not found` or exit status 127.

**Checklist**
- Check if npm is installed: `npm --version`
- Check if Node.js is installed: `node --version`
- Confirm conda environment is activated (if using conda)

**Solutions**
```bash
# Option A: Using Conda (Recommended)
conda install -c conda-forge nodejs

# Option B: Using Official Installer
# Download from https://nodejs.org/

# Option C: Using nvm
nvm install 18
nvm use 18
```

**Verify Installation**
```bash
node --version  # Should show v18.x.x or higher
npm --version   # Should show version number
```

---

## Frontend cannot connect to backend?

**Checklist**
- Confirm backend is running (visit `http://localhost:8001/docs`)
- Check browser console for error messages

**Solution**

Create `.env.local` in `web` directory:

```bash
NEXT_PUBLIC_API_BASE=http://localhost:8001
```

---

## WebSocket connection fails?

**Checklist**
- Confirm backend is running
- Check firewall settings
- Confirm WebSocket URL is correct

**Solution**
- **Check backend logs**
- **Confirm URL format**: `ws://localhost:8001/api/v1/...`

---

## Where are module outputs stored?

| Module | Output Path |
|:---:|:---|
| Solve | `data/user/solve/solve_YYYYMMDD_HHMMSS/` |
| Question | `data/user/question/question_YYYYMMDD_HHMMSS/` |
| Research | `data/user/research/reports/` |
| Co-Writer | `data/user/co-writer/` |
| Notebook | `data/user/notebook/` |
| Guide | `data/user/guide/session_{session_id}.json` |
| Logs | `data/user/logs/` |

---

## How to add a new knowledge base?

**Web Interface**
1. Visit `http://localhost:3782/knowledge`
2. Click "New Knowledge Base"
3. Enter knowledge base name
4. Upload PDF/TXT/MD documents
5. System will process documents in background

**CLI**
```bash
deeptutor kb create <kb_name> --doc <pdf_path>
```

---

## How to incrementally add documents to existing KB?

**CLI (Recommended)**
```bash
deeptutor kb add <kb_name> --doc <new_document.pdf>
```

**Benefits**
- Only processes new documents, saves time and API costs
- Automatically merges with existing vector index
- Preserves all existing data

---

## Numbered items extraction / uvloop issue

Numbered items extraction has been removed in llamaindex-only mode.
If you previously encountered `uvloop.Loop` errors from that flow, no action is required now.
