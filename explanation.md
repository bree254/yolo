# Project Explanation

## 1. Base Image Selection
- **Frontend**
Uses `node:16-alpine` ,I chose alpine to keep it lightweight ,and on the version I decided to not use the version 14 cause it's a little outdated ,first tried using version 18 but it was quite big ,so I downgraded to a smaller version and 16 was the right balance.
For the runtime stage I used `nginx:1.25-alpine` cause after doing my research i realized it has a low footprint and the delivery performance is high.
- **Backend**
Runs on `node:16-alpine` ,which gives a supported LTS version of Node.js ,it provides the node 16 toolchain to run npm ci on top of Alpine Linux for minimal image size .
The runtime stage switches to `alpine:3.16.7` so the final container is smaller
- **Database**
Relies on mongo:3.0 
I tried pulling a few mongo images (4.2.24 was ~388 MB)(6.0 was ~765 MB), the images were verylarge so i decided to downgrade and this version had the least image size 232 mbs which fit into the rubric requirements .It's an older release but it's stil compatible with the Mongoose 5.x client in the setup.


## 2. Dockerfile Directives

### Frontend
- **Building the react bundle**
  - `FROM node:16-alpine AS build` ,`WORKDIR /app` and `COPY package*.json ./` set up the builder with node
  - `RUN npm ci` used this instead of npm install because it removes node_modules,installs what is the package-lock.json ,which makes the installs faster and more suitable for docker builds.
  - `RUN npm run build` compiles the production bundle into `/app/build`
- **Building the react bundle**
  - `FROM nginx:1.25-alpine` provides a smaller image
  - `COPY --from=build /app/build /usr/share/nginx/html` publishes the compiled version
  - `EXPOSE 80` the public http port that Nginx also runs on
  and `CMD ["nginx" , "-g" , "daemon off;"]` starts Nginx in the foreground and keeps the docker container alive

### Backend
I went with a two-stage build
- **Stage 1 Installing dependencies**
  - `FROM node:16-alpine AS build`
  - `WORKDIR /app` , `COPY package*.json ./` and `RUN npm ci --omit=dev` installs what is the package-lock.json,removes node_modules which is more suitable for docker builds

- **Stage 2 Runtime image**
  - `FROM alpine:3.16.7 AS prod` provides a smaller image
  - `ENV NODE_ENV=production` and `WORKDIR /app` sets up the runtime environment
  - `RUN apk add --no-cache nodejs` installs node.js in the final alpine stage to avoid  the container failing to build with a "node not found error"
  -  `COPY --from=build /app /app` copies the compiled files out of the builder stage
  -  `EXPOSE 5000` the port the server runs
  - `CMD ["node","server.js"]` whenever the container starts it boots the express server in production mode


## 3. Docker-compose networking
- **Networking Implemantation**
From the rubric ,we are required to implement custom bridge networks that connect the containers .So I decided to create yolo-net instead of the default bridge so the three services can live on their own subnet and avoid clashing with other projects.I chose a 172.30 range so it wouldnt overlap with anything else I run locally
- **Port allocation**
From my frontend container the port is pointed to port 80 but for compose I published it on port 3000 so I can acceess it on the browser.
The backend exposes 5000 for API calls and I kept Mongo's 27017 open in case I need to connect with a local shell


## 4. Docker-compose volume definition and usage
For this , I defined mongo-data a named Docker colume and mounted it at /data/db inside the Mongo container.Docker keeps the volume outside the container filesystem ,so even if I run `docker compose down` or rebuild the yolo-mongo container ,the BSON files remain untouched.
From the rubric requirements : the database survives the container restarts without any manual export/import .I tested this by removing the container ,bringing it back up and confirming that all previously added products were still there.

