FROM python:3.13-alpine3.22

WORKDIR /app
ARG APP_VERSION=dev
ENV PYTHONDONTWRITEBYTECODE=1 \
    PYTHONUNBUFFERED=1 \
    APP_VERSION=${APP_VERSION}

RUN addgroup -S festpass && adduser -S -G festpass festpass
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY --chown=festpass:festpass app.py .

EXPOSE 5000
USER festpass
CMD ["python", "app.py"]
