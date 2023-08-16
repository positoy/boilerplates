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
   - [spring initializr](https://start.spring.io/)
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
   cd next
   npm run build
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
   docker run --name nextjs -d --rm positoy/nextjs
   docker push positoy/nextjs
   ```

3. Kubernetes
   ```bash
   docker build -t positoy/nextjs .
   kubectl create deployment nextjs --image=positoy/nextjs --port=3000
   kubectl expose deployment nextjs --type=NodePort --port=3000
   minikube service nextjs
   ```
