# Deployment Guide

## 1. Local Development

### Backend API (Port 5000)
```bash
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python train.py
python app.py
```
> Flask runs as a pure JSON API at `http://localhost:5000`.

### Frontend Dashboard (Port 3000)
```bash
cd frontend
npm install
npm run dev
```
> The user interface renders exclusively at `http://localhost:3000`.

---

## 2. Production Deployment

### Backend API (Gunicorn / WSGI)
```bash
gunicorn app:app --bind 0.0.0.0:$PORT --workers 1 --threads 2 --timeout 120
```

### Frontend Dashboard (Next.js Production Build)
```bash
cd frontend
npm run build
npm run start -p 3000
```

---

## 3. Docker Deployment (Backend API)
```dockerfile
FROM python:3.14-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt
COPY . .
EXPOSE 8080
CMD ["sh", "-c", "gunicorn app:app --bind 0.0.0.0:${PORT:-8080} --workers 1 --threads 2 --timeout 120"]
```
Build & run:
```bash
docker build -t aqi-backend .
docker run -p 5000:5000 aqi-backend
```

---

## 4. Cloud Platforms
- **Backend (Render / Railway / Fly.io)**: Point to the repository root and set the start command to `gunicorn app:app --bind 0.0.0.0:$PORT --workers 1 --threads 2 --timeout 120`. The platform provides the port.
- **Frontend (Vercel / Cloudflare / Netlify)**: Point to `frontend/` directory, set build command `npm run build`, output `.next`. Set `NEXT_PUBLIC_API_BASE` to the backend URL.

## Environment Variables
- `PORT` — Flask port provided by the hosting platform.
- `WAQI_API_KEY` — WAQI token configured only in the backend hosting provider.
- `NEXT_PUBLIC_API_BASE` — Backend API base URL for Next.js (defaults to same-origin requests locally).

The production model is intentionally trained with a compact Random Forest
configuration so it remains within Render's free 512 MB memory limit. Keep
the backend at one Gunicorn worker; additional workers load a separate copy
of the model into memory.
