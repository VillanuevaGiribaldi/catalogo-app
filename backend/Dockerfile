FROM python:3.12-slim AS builder
COPY ./requirements.txt .
RUN python -m venv .venv 
ENV PATH="/app/.venv/bin:$PATH"
RUN pip install --user --no-cache-dir -r requirements.txt


FROM python:3.12-slim 
#WORKDIR /app
RUN useradd --create-home appuser
COPY --from=builder /root/.local /home/appuser/.local
WORKDIR /app
COPY . .
ENV PATH=/home/appuser/.local/bin:$PATH
ENV PYTHONUNBUFFERED=1

ENV DB_HOST=localhost \
    DB_PORT=27017 \
    DB_NAME=catalogodb \
    DB_USER=catalogo_user \
    DB_PASSWORD=catalogo_pass \
    ROOT_PATH="/api"

ARG APP_VERSION=v1
ENV APP_VERSION=${APP_VERSION}
USER appuser
EXPOSE 8000
CMD ["uvicorn", "main:app", "--host", "0.0.0.0" ,"--port", "8000"]