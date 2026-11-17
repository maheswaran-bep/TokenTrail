
FROM python:3.12-slim

WORKDIR /app

ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    UV_LINK_MODE=copy

RUN pip install --no-cache-dir uv

# Copy the complete backend, including the local SDK workspace.
COPY . .

# Install dependencies from the committed lock file.
RUN uv sync --frozen --no-dev

EXPOSE 8000

# Apply database migrations before starting FastAPI.
CMD ["sh", "-c", "uv run --no-sync alembic upgrade head && uv run --no-sync python run.py prod"]
