# Ready Docker & k8s

```bash
# setup qemu socket_vvmnet (for service tunnel)
brew install socket_vmnet
brew tap homebrew/services
HOMEBREW=$(which brew) && sudo ${HOMEBREW} services start socket_vmnet

# install on mac
brew install docker docker-compose docker-credential-helper minikube

# minikube configurations
minikube config set cpus 2
minikube config set memory 2048
minikube config set disk-size 20gb
minikube start --driver qemu --network socket_vmnet

# set docker daemon
eval $(minikube docker-env)

# install vscode plugins : docker, kubernetes, bridge to kubernetes
```

## Spring Boot

1. Project
   [spring initializr](https://start.spring.io/)
2. Docker
   ```Dockerfile
   FROM openjdk:8
   VOLUME /tmp
   COPY build/libs/*-SNAPSHOT.jar app.jar
   ENTRYPOINT ["java","-jar","/app.jar"]
   ```
   ```bash
   # build image
   ./gradlew build
   docker build -t positoy/boot .
   # create container with default "bridge" network with private IPAddress
   docker run --name boot -d --rm positoy/boot
   # only containers in the same host can access.
   docker run --rm curlimages/curl -L -v http://`docker inspect -f "{{ .NetworkSettings.IPAddress }}" boot`:8080
   ```
3. Kubernetes
   ```bash
   # push the docker image to registry
   docker push positoy/boot
   # create deployment, port is just meta info
   kubectl create deployment boot --image=positoy/boot --port=8080
   # create service, deployment port info is helpful here.
   # kubectl create service nodeport boot --tcp=8080:8080
   kubectl expose deployment boot --type=NodePort --port=8080
   # open browser for the service
   minikube service boot
   # service update if needed
   kubectl rollout restart deployment/boot
   ```

# Next.js

1. Project
   ```bash
   npx create-next-app@latest --template typescript next
   ```
2. Docker
   ```Dockerfile
   FROM node:18.17.1-alpine
   VOLUME /tmp
   COPY next /tmp/next
   WORKDIR /tmp/next
   ENTRYPOINT ["npm","run","start"]
   ```
   ```bash
   cd next
   npm run build
   cd ..
   docker build -t positoy/nextjs .
   docker run --name nextjs -d --rm positoy/nextjs
   ```
3. Kubernetes
   ```bash
   docker push positoy/nextjs
   kubectl create deployment nextjs --image=positoy/nextjs --port=3000
   kubectl expose deployment nextjs --type=NodePort --port=3000
   minikube service nextjs
   ```

# Node.js Express

1. Project
   ```bash
   mkdir express
   cd express
   npm init -y
   npm add express typescript @types/express @types/node
   npx tsc --init
   ```
   ```json
   # tsconfig.js
   {
       ...
       "compilerOptions": {
           "outDir": "dist"
       }
       ...
   }
   # package.json
   {
       ...
       "scripts": {
           "build": "npx tsc",
           "start": "node dist/server.js"
       }
       ...
   }
   ```
2. Docker

   ```Dockerfile
   FROM node:18.17.1-alpine
   VOLUME /tmp
   COPY express /tmp/express
   WORKDIR /tmp/express
   ENTRYPOINT ["npm","run","start"]
   ```

   ```bash
   cd express
   npm run build
   cd ..
   docker build -t positoy/express .
   docker run --name express -d --rm positoy/express
   ```

3. Kubernetes
   ```bash
   docker push positoy/exprss
   kubectl create deployment express --image=positoy/express --port=3000
   kubectl expose deployment express --type=NodePort --port=3000
   minikube service express
   ```

# MySQL

```bash
# mysql-server
docker run --name mysql -e MYSQL_ROOT_PASSWORD=password -d mysql:5.7
docker exec -it mysql bash
mysql -uroot -p
```

```sql
-- create 'user' user
create user 'user' identified by 'password';
```

```bash
# mysql-client
docker run -it --rm --name mysql-client mysql:5.7 mysql -h`docker inspect -f "{{ .NetworkSettings.IPAddress }}" mysql` -uuser -p
```
