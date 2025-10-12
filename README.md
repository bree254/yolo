# Yolo microservice platform

## Overview
This project involves a containerized e-commerce dashboard YOLO.The docker compose orchestrates three services :
- `yolo-frontend` - spa compiled to static assets and served by nginx
- `yolo-backend` - an express API for CRUD operations on products
- `yolo-mongo` - MongoDB with persisted volume for product data

## Requirements
- Docker Engine 24+ with the compose plugin
- A docker hub account for publishing the images  in the `docker-compose.yaml`

## Getting started
1. Clone the repository and be on the root directory of the project
2. Build and start the stack 
```bash 
docker compose up --build
```
3. To verify the stack we run
```bash 
docker compose ps
```
 ![Docker compose ps image](screenshots/docker-compose-ps.png)
 ```bash 
docker compose logs -f yolo-backend
```
 ![Docker compose logs image](screenshots/docker-compose-logs.png)

4. To see the images and check their sizes we run 
```bash 
docker images 
```
 ![Docker image sizes](screenshots/docker-images.png)
For specific image sizes you can run 
```bash 
docker images bree254/yolo-frontend
```
 ![Frontend image size](screenshots/frontend-image.png)
```bash 
docker images bree254/yolo-backend
```
 ![Backend image size](screenshots/backend-image.png)
```bash 
docker images mongo:3.0.
```
 ![Mongo image size](screenshots/mongo-image.png)

5. From the rubric the image requirements were for the images to be below 400 mbs and ours with the frontend backend and database images they come to 362mbs which is right below the 400mbs

6. Once you do that , go to http://localhost:3000 and add products. The backend is on http://localhost:5000 and the data is persisted to the Mongo volume.
![Yolo website](screenshots/yolo-website.png)

7. To check if everything worked well ,once on the website you click the add product button and see if you can add the product and if the products you added can be seen
![Yolo website with products](screenshots/yolo-image-with-products.png)

8. Then to check the data persistence you can bring the slack down by first `CTRL + C` and then running these commands 
```bash 
docker compose down
docker compose up -d --pull always
```
when you check the website again and see your products still there you know the persistence works 

## Publishing images 
We push the images to Docker hub by running the following commands 
```bash
docker compose build
docker login
docker compose push
```
### Docker Hub repository
![Docker hub repository](screenshots/dockerhub-repo.png)
### Docker Hub Frontend tag
![Docker Hub – backend tags](screenshots/dockerhub-frontend.png)
### Docker Hub Backend tag
![Docker Hub – frontend tags](screenshots/dockerhub-backend.png)

