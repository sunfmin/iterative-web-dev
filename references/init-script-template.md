# init.sh Template

The `init.sh` script sets up the development environment. It must be idempotent (safe to run multiple times).

## Requirements

1. **Kill existing servers** — Clean slate
2. **Delete old screenshots** — Fresh test results
3. **Rebuild all necessary files** — Ensure latest code
4. **Restart all servers** — Backend, frontend, database
5. **Be idempotent** — Safe to run multiple times

## Template

```bash
#!/bin/bash
set -e

echo "=== Project Development Environment ==="

# 1. Kill existing servers
echo "Stopping existing servers..."
pkill -f 'go run' 2>/dev/null || true
pkill -f 'vite' 2>/dev/null || true
pkill -f 'node.*dev' 2>/dev/null || true
sleep 1

# 2. Delete old screenshots for fresh test results
echo "Cleaning old screenshots..."
rm -rf frontend/e2e/screenshots/*.png 2>/dev/null || true
rm -rf frontend/test-results 2>/dev/null || true
mkdir -p frontend/e2e/screenshots

# 3. Install/update dependencies
echo "Installing dependencies..."
cd frontend && npm install && cd ..
cd backend && go mod download && cd ..

# 4. Rebuild backend
echo "Building backend..."
cd backend && go build -o backend . && cd ..

# 5. Start database (if needed)
echo "Ensuring database is running..."
brew services start postgresql@18 2>/dev/null || true

# 6. Start backend
echo "Starting backend on port 8082..."
cd backend && ./backend &
BACKEND_PID=$!
cd ..

# 7. Start frontend
echo "Starting frontend on port 3000..."
cd frontend && npm run dev &
FRONTEND_PID=$!
cd ..

# 8. Wait for servers to be ready
echo "Waiting for servers..."
sleep 3

# 9. Verify servers are running
if lsof -i :8082 > /dev/null 2>&1; then
    echo "✓ Backend running on port 8082"
else
    echo "✗ Backend failed to start"
fi

if lsof -i :3000 > /dev/null 2>&1; then
    echo "✓ Frontend running on port 3000"
else
    echo "✗ Frontend failed to start"
fi

echo ""
echo "=== Environment Ready ==="
echo "Frontend: http://localhost:3000"
echo "Backend:  http://localhost:8082"
echo ""
echo "Active scope: $(cat .active-scope 2>/dev/null || echo 'none')"
```

## Customization

Adapt the template for your project:

### Node.js Backend
```bash
# Replace Go build with Node
cd backend && npm install && cd ..
cd backend && npm run build && cd ..
cd backend && npm start &
```

### Python Backend
```bash
# Replace Go build with Python
cd backend && pip install -r requirements.txt && cd ..
cd backend && python app.py &
```

### Docker Services
```bash
# Start Docker containers
docker-compose up -d
```

### Different Ports
```bash
# Adjust port checks
lsof -i :5173  # Vite default
lsof -i :4000  # Custom backend
```

## Verification Commands

After running init.sh, verify services:

```bash
# Check what services are needed
grep -A 10 "webServer" playwright*.config.ts 2>/dev/null || true

# Check ports
lsof -i :3000  # Frontend
lsof -i :8082  # Backend

# Test endpoints
curl -s http://localhost:8082/health || echo "Backend not responding"
curl -s http://localhost:3000 | head -5 || echo "Frontend not responding"
```
