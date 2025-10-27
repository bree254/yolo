# Stage 1 Playbook Explanation

## Role Order
This play runs in a dependency order :
 base OS setup -> Docker runtime -> application code checkout ->
  containerized services (mirroring `docker-compose.yaml`).
Each role builds on the previous one so later steps always find their prerequisites in place.

1. **project_setup**
This handles the project setup ,so that every other role has the base tooling it needs. 
It updates the apt cache then installs the shared packages required across the app e.g `git`,`curl`,`python3-pip`.

2. **app_code**
This ensures that the root directory exists.It uses the `file` and `git` module to clone the repository into `/opt/yolo` .Having the repository in place provides build contexts for the container roles.

3. **docker**
This installs the Docker engine (apt tasks + GPG key + repo ) ,starts  and enables the docker service ,adds vagrant to the docker group (giving root permission to run commands)
It also guarantees shared resources (`yolo-net` ,`mongo-data volume`) through `community.docker.docker_network` and `docker_volume`, ensuring the subsequent roles can attach containers immediately.

4. **frontend**
This builds the react/NGINX image through the `community.docker.docker_image` with the `API_BASE_URL` build argument.
It then runs the container on `yolo-net` exposing port 3000.It depends on the repo clone and Docker network.

5. **backend**
This builds the Node/Express image ,injects the `MONGODB_URI` through the environment variables ,and then runs the container exposing port 5000.
It relies on the Mongo being reachable on the Docker network.

6. **database**
The `community.docker.docker_container` launches `mongo:3.0` ,mounting the managed volume and adding the health check .
It provides persistence for backend operations.

## Variables and Defaults
- Created the `group_vars/all.yml` which holds the global settings: repo info, file paths, container names, host ports, and Docker network/volume names.
- Each role has its own defaults for role specific behaviour (image tags, env vars, container names,etc.) so overrides stay localized.

## Tags and Blocks
- Blocks ,group related tasks within each role and Tags let you rerun just a part of a play .
- Every role’s tasks sit inside a `block`, keeping related steps together and making tag assignment simple.
- Tags mirror the role names. After tagging each block, `playbook.yml` references roles as `{ role:frontend, tags: ['frontend'] }`, so Ansible can target just that component.
- After adding the tags to your tasks in the roles folder ,I updated the `playbook.yml` ,roles with the specific role and their tag
 - Inside the VM you can run:
    ```bash
    ansible-playbook -i inventory.yml playbook.yml --tags <tag-name>

  to execute a single slice (e.g., frontend, backend, docker) without touching the rest.

## Modules Used
- project_setup: `apt` (update cache, install base packages).
- app_code: `file`, `git`.
- docker: `apt`, `ansible.builtin.shell `(GPG key pipeline), `apt_repository`, `service`, `user`,`community.docker.docker_network`, `community.docker.docker_volume`.
- frontend/backend: `community.docker.docker_image` (build), `community.docker.docker_container` (run).
- database: `community.docker.docker_container` (with volume + healthcheck).


