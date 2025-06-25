FROM ghcr.io/astral-sh/uv:alpine

RUN apk add curl

WORKDIR /home/app/

COPY . .

CMD ["uv", "run", "app.py"]
