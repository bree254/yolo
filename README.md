# YOLO Platform

1. **Docker Compose microservice stack** – local-first workflow for building/publishing the frontend, backend, and MongoDB images.
2. **Stage 1 infrastructure** – a reproducible Vagrant VM that provisions the same stack with Ansible roles and tags.

---

## Docker Compose Microservice Stack

### Overview
The containerized YOLO dashboard runs as three services managed by `docker-compose.yaml`:
- `yolo-frontend`: React SPA compiled to static assets and served by NGINX
- `yolo-backend`: Express API that persists products to MongoDB
- `yolo-mongo`: MongoDB with a named volume for data durability

### Requirements
- Docker Engine 24+ with the Compose plugin
- Docker Hub account for `docker compose push` (images reference `bree254/*`)

### Getting Started
1. Clone the repo and stay in the project root.
2. Build and launch the stack:
   ```bash
   docker compose up --build
   ```
3. Verify the services:
   ```bash
   docker compose ps
   docker compose logs -f yolo-backend
   ```
   ![Docker compose ps image](screenshots/docker-compose-ps.png)
   ![Docker compose logs image](screenshots/docker-compose-logs.png)
4. Inspect image sizes:
   ```bash
   docker images
   docker images bree254/yolo-frontend
   docker images bree254/yolo-backend
   docker images mongo:3.0
   ```
   ![Docker image sizes](screenshots/docker-images.png)
   ![Frontend image size](screenshots/frontend-image.png)
   ![Backend image size](screenshots/backend-image.png)
   ![Mongo image size](screenshots/mongo-image.png)
5. Size target: combined images (frontend 50.4 MB, backend 78.9 MB, Mongo 232 MB) total 362 MB, comfortably below the 400 MB rubric threshold.
6. Browse http://localhost:3000 to add products. Backend runs on http://localhost:5000 and writes to the `mongo-data` volume.
   ![Yolo website](screenshots/yolo-website.png)
   ![Yolo website with products](screenshots/yolo-image-with-products.png)
7. Validate CRUD + persistence:
   - Add a product through the UI.
   - Bring the stack down and back up to confirm data remains:
     ```bash
     docker compose down
     docker compose up -d --pull always
     ```

### Publishing Images
```bash
docker compose build
docker login
docker compose push
```
![Docker hub repository](screenshots/dockerhub-repo.png)
![Docker Hub – backend tags](screenshots/dockerhub-frontend.png)
![Docker Hub – frontend tags](screenshots/dockerhub-backend.png)

---

## Stage 1 Infrastructure (Vagrant + Ansible)

## Project Overview
This repository contains the Stage 1 deliverable for the YOLO e-commerce platform. The `vagrant up` command boots an `ubuntu/jammy64` virtual machine, installs Docker, clones the web application, builds the frontend and backend images, and deploys the three containerized services (frontend, backend API, MongoDB) with persistence.

## Prerequisites
1. [VirtualBox](https://www.virtualbox.org/) 7.x
2. [Vagrant](https://developer.hashicorp.com/vagrant/downloads) 2.4+

> The VM uses the `ansible_local` provisioner so no SSH keys or controller-side Ansible installation are required.

## Getting Started
```bash
# 1. Clone this repo
$ git clone https://github.com/bree254/yolo.git
$ cd yolo

# 2. Start and provision the VM (downloads box on first run)
$ vagrant up

# 3. When provisioning finishes, verify the containers
$ vagrant ssh -c "sudo docker ps"
$ curl http://localhost:5000/api/products
```
Docker ps

![Docker ps](screenshots/vagrant-docker-compose.png)

Products

![Products](screenshots/curl-add-products.png)

## Ports and Access
- Frontend: http://localhost:3000
- Backend API: http://localhost:5000
- MongoDB: localhost:27017 (forwarded for local debugging)

Products added via the UI persist because Mongo mounts the managed `mongo-data` volume.

Frontend UI

![Frontend UI](screenshots/vagrant-yolo-website.png)

Add Product

![Add Product](screenshots/vagrant-add-product.png)

## Inventory and Tags
The playbook runs inside the guest with ansible_local, so tags are invoked from the VM:

```bash
$ vagrant ssh
vagrant@...$ cd /vagrant
vagrant@...$ ansible-playbook -i inventory.yml playbook.yml --tags frontend
```

Available tags: `project_setup`, `docker`, `app_code`, `frontend`, `backend`, `database`.

 Examples:

Frontend Tag
![Frontend Tag](screenshots/vagrant-frontend.png)

Backend Tag
![Backend Tag](screenshots/vagrant-backend.png)

Database Tag
![Database Tag](screenshots/vagrant-database.png)

## Repo Structure

- `Vagrantfile` – VM and Ansible provisioner configuration
- `playbook.yml` – Orchestrates the roles (see [explanation.md](explanation.md) for ordering decisions and Ansible module choices)
- `group_vars/all.yml` – Shared variables (paths, container names, ports)
- `roles/` – Individual roles for setup, Docker, code checkout, and services
- `screenshots/` – Reference images used in this README
- [`Structure`](Structure) – Directory tree of the repository

## Troubleshooting
- If provisioning fails or you change code, rerun:

```bash
vagrant provision
```

![Vagrant provision](screenshots/vagrant-provision.png)

- Re-run a single component via tags (inside the VM):
```bash
ansible-playbook -i inventory.yml playbook.yml --tags backend
```
    
- Check container logs:
```bash
 vagrant ssh -c "docker logs yolo-backend"
```

## Cleaning Up
```bash
$ vagrant halt        # stop the VM
$ vagrant destroy -f  # remove the VM and all containers
```

## Stage 2 (optional)
Stage 2 (Terraform + Ansible) is not implemented yet.
