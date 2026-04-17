# ============================================================
# Scaling Lab Dockerfile
# ============================================================
FROM python:3.11-slim

WORKDIR /app

# Install standard dependencies
RUN pip install --no-cache-dir fastapi uvicorn[standard] redis psutil

# Copy current folder code
COPY . .

# Expose port
EXPOSE 8000

# Start command
CMD ["uvicorn", "app:app", "--host", "0.0.0.0", "--port", "8000"]
