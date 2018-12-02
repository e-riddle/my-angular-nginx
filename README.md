# My Angular 7 App With Nginx Deployment

This project was generated with [Angular CLI](https://github.com/angular/angular-cli) version 7.0.6.

## Development server

Run `ng serve` for a dev server. Navigate to `http://localhost:4200/`. The app will automatically reload if you change any of the source files.

## Code scaffolding

Run `ng generate component component-name` to generate a new component. You can also use `ng generate directive|pipe|service|class|guard|interface|enum|module`.

## Build

Run `ng build` to build the project. The build artifacts will be stored in the `dist/` directory. Use the `--prod` flag for a production build.

## Running unit tests

Run `ng test` to execute the unit tests via [Karma](https://karma-runner.github.io).

## Running end-to-end tests

Run `ng e2e` to execute the end-to-end tests via [Protractor](http://www.protractortest.org/).

## Further help

To get more help on the Angular CLI use `ng help` or go check out the [Angular CLI README](https://github.com/angular/angular-cli/blob/master/README.md).


# Docker Nginx Deployment

## Standard Nginx Docker Deployment

DockerFile Contents

```
FROM nginx:alpine

COPY nginx.conf /etc/nginx/nginx.conf

WORKDIR /usr/share/nginx/html
COPY dist/my-angular-app .
```
Nginx Config Contents
```
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    server {
        listen 80;
        server_name  localhost;

        root   /usr/share/nginx/html;
        index  index.html index.htm;
        include /etc/nginx/mime.types;

        gzip on;
        gzip_min_length 1000;
        gzip_proxied expired no-cache no-store private auth;
        gzip_types text/plain text/css application/json application/javascript application/x-javascript text/xml application/xml application/xml+rss text/javascript;

        location / {
            try_files $uri $uri/ /index.html;
        }
    }
}
```

### Docker Run
```
ng build --prod

docker image build -t my-angular-app .

docker run -p 3000:80 --rm my-angular-app
```

### Docker Compose

Compose File Contents
```
version: '3.1'

services:
    app:
        image: 'my-angular-app'
        build: '.'
        ports:
            - 3000:80
```

```
docker-compose up -d
```

App will be running on http://localhost:3000

Then to down it and reset

```
docker-compose down --rmi all
```

## Docker Nginx Deployment With Build Image

Inspired by dotnet dockerfile

https://docs.docker.com/engine/examples/dotnetcore/#build-and-run-the-docker-image


### Create Build Image

Place this in a docker file
```
FROM ubuntu:latest
LABEL maintainer="Eric Riddle"

RUN apt-get update && \
    apt-get install -y sudo curl git-core gnupg linuxbrew-wrapper locales nodejs zsh wget nano nodejs npm fonts-powerline && \
    sudo npm install -g @angular/cli && \
    mkdir publish
```

Build the Build Image
```
docker build --rm -f Dockerfile -t ubuntu:angular-7-install .
```
Make sure build image has all of the needed software
```
docker run -it ubuntu:angular-7-install /bin/bash
```

Nginx DockerFile Contents

```
FROM ubuntu:angular-7-install AS build-env
LABEL maintainer="Eric Riddle"
WORKDIR /publish

COPY . ./
RUN npm install
RUN ng build --prod

FROM nginx:alpine
WORKDIR /usr/share/nginx/html
COPY --from=build-env /publish/nginx.conf /etc/nginx/nginx.conf
COPY --from=build-env /publish/dist/my-angular-app .
```


Run Nginx Build and run
```
docker build --rm -f Dockerfile-build -t nginx:my-angular-app2 .

docker run -p 3000:80 --rm nginx:my-angular-app2
```

App will be running on http://localhost:3000



