FROM python:3.12-slim

RUN useradd -r -s /usr/sbin/nologin emobile

WORKDIR /app

COPY app.py .

EXPOSE 8080

USER emobile

CMD ["python", "app.py"]

