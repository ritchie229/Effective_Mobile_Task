## Тестирование

### Project structure

```bash
.
├── backend/
│   ├── Dockerfile
│   └── app.py
├── nginx/
│   └── nginx.conf
├── docker-compose.yml
└── README.md
```

### Простое приложение на Python app.py

```
from http.server import BaseHTTPRequestHandler, HTTPServer

class Handler(BaseHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            self.send_response(200)
            self.send_header("Content-Type", "text/plain")
            self.end_headers()
            self.wfile.write(b"Hello from Effective Mobile!")
        else:
            self.send_response(404)
            self.end_headers()

if __name__ == "__main__":
    server = HTTPServer(("0.0.0.0", 8080), Handler)
    server.serve_forever()

```

### Dockerfile

```yaml
FROM python:3.12-slim

RUN useradd -r -s /usr/sbin/nologin emobile

WORKDIR /app
COPY app.py .

EXPOSE 8080  # expose не публикует порт, снаружи порт недоступен

USER emobile

CMD ["python", "app.py"]

```

### Попутная проверка

#### Билд
```bash
docker build -t backend-app .
```

#### Чек
Запускаем:
```bash
docker run --name backend-app --rm --network test-project backend-app
```
на другом терминале:
```bash
docker run --rm --network test-project alpine/curl -fsSL http://172.21.0.3:8080
```
> [!NOTE]
> Приложение изолировано. <br>

Получаем: `Hello from Effective Mobile!`

### nginx.conf

```nginx
events {}

http {
    upstream backend_app {
        server backend-svc:8080;
    }

    server {
        listen 80;

        location / {
            proxy_pass http://backend_app;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
    }
}
```

### docker-compose.yml

```yaml
version: "3.9"

services:
  backend-svc:
    build: .
    container_name: backend-app
    expose:
      - "8080"
    networks:
      - test-project

  nginx:
    image: nginx:alpine
    container_name: nginx
    ports:
      - "80:80"
    volumes:
      - ./nginx/nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - backend-svc
    networks:
      - test-project

networks:
  test-project:
    driver: bridge
```

### Запуск

```bash
docker compose up -d --build
```

### Проверяем 

```bash
curl http://localhost
```

### Ответ 

`Hello from Effective Mobile!`

> [!NOTE]
> Порт приложения остается доступным только в докер сети, nginx выполняет функцию reverse proxy.




