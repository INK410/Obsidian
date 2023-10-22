## sa-logic
```Dockerfile
FROM python:3.6
WORKDIR /root

COPY requirements.txt /root/requirements.txt
COPY app.py /root/app.py
ADD python-packages.tar.gz /root
RUN pip3 install --no-index --find-links=/root/python-packages -r /root/requirements.txt
EXPOSE 5000
CMD ["python3","app.py"]
```

## sa-webapp
```Dockerfile
FROM golang:1.15.5
WORKDIR /root

COPY go.sum go.mod /root
COPY . /root

RUN GOPROXY=https://goproxy.cn go mod download
RUN go build -o webapp .

EXPOSE 8080
CMD ["./webapp"]
```

## sa-frontend
```dockerfile
FROM node:14 as builder
WORKDIR /root

ADD sa-frontend.tar.gz /root
ENV VUE_APP_API_HOST=http://192.168.1.132:9002
RUN npm run build

FROM nginx
RUN rm -rf /usr/share/nginx/html
COPY --from=builder /root/dist/ /usr/share/nginx/html 
EXPOSE 80
CMD ["nginx","-g","daemon off;"]
```

## docker_compose_yaml
```yaml
---
version: "3"
services:
  sa-logic:
    container_name: sa-logic
    image: sa-logic1
    ports:
      - 9001:5000
  sa-webapp:
    container_name: sa-webapp
    image: sa-webapp1
    depends_on:
      - sa-logic
    ports:
      - 9002:8080
    environment:
      - API_HOST=http://sa-logic:5000
  sa-frontend:
    container_name: sa-frontend
    image: sa-front
    depends_on:
      - sa-webapp
    ports:
      - 9003:80
```