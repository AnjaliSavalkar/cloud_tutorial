Sure! Here's the updated `documentation.md` that includes both the backend and frontend setup, Docker build instructions, and running instructions.

---

# Shopper Project Documentation

## Overview

The **Shopper Project** consists of three main components:

* **Frontend (`shopper-admin`)**: The user interface for interacting with the application.
* **Backend (`shopper-backend`)**: The server that handles API requests.
* **Admin Frontend (`shopper-admin`)**: A separate admin interface to manage the application.

All components are containerized using Docker and connected via a shared Docker network.

## Prerequisites

Ensure the following tools are installed on your system:

* **Docker**: For containerizing and running both the frontend and backend.
* **Node.js**: For building and running the application (this will be used during the Docker build process).
* **Docker Compose** (optional): For managing multi-container applications (you can skip if managing manually).

## Setup Instructions

### 1. **Clone the Repository**

First, clone the repository containing the frontend and backend code.

```bash
git clone <repository-url>
cd shopper
```

### 2. **Building the Docker Images**

#### Backend (`shopper-backend`)

Navigate to the `backend` directory and build the Docker image for the backend:

```bash
cd backend
docker build -t shopper-backend .
```

#### Admin Frontend (`shopper-admin`)

Navigate to the `admin/vite-project` directory and build the Docker image for the admin frontend:

```bash
cd admin/vite-project
docker build -t shopper-admin .
```

#### Frontend (`shopper-frontend`)

Navigate to the `frontend` directory and build the Docker image for the frontend:

```bash
cd frontend
docker build -t shopper-frontend .
```

---

## Running the Containers

### 1. **Create a Docker Network**

To ensure all components can communicate with each other, create a shared Docker network:

```bash
docker network create demo
```

### 2. **Running the Backend Container**

Run the backend container with port mapping. The backend container will be available on port `5050`:

```bash
docker run --network=demo --name=backend -d -p 5050:5050 shopper-backend
```

### 3. **Running the Admin Frontend Container**

Run the admin frontend container with port mapping. The admin frontend container will be available on port `5174`:

```bash
docker run --network=demo --name=admin-frontend -d -p 5174:5174 shopper-admin
```

### 4. **Running the User Frontend Container**

Run the user-facing frontend container with port mapping. The user frontend container will be available on port `5173`:

```bash
docker run --network=demo --name=user-frontend -d -p 5173:5173 shopper-frontend
```

---

## Verifying the Setup

### 1. **Check Running Containers**

Use the following command to verify that all containers are running:

```bash
docker ps
```

This will display a list of running containers. Ensure that `shopper-backend`, `shopper-admin`, and `shopper-frontend` are all listed.

### 2. **Access the User Frontend**

The user frontend should now be accessible at:

```
http://localhost:5173
```

### 3. **Access the Admin Frontend**

The admin frontend should now be accessible at:

```
http://localhost:5174
```

### 4. **Access the Backend API**

The backend API can be accessed at:

```
http://localhost:5050
```

Ensure that the frontend is making requests to `http://backend:5050`, as both containers are part of the same Docker network.

---

## Logs and Debugging

If you encounter any issues, you can check the logs of any of the containers using the following command:

```bash
docker logs <container-name>
```

For example, to view the logs of the user frontend:

```bash
docker logs user-frontend
```

To view the logs of the backend:

```bash
docker logs backend
```

---

## Dockerfile Explanation

### Backend Dockerfile (`shopper-backend`)

The backend Dockerfile builds the backend container. Here's the breakdown:

1. **Base Image**: `node:18` is used to run Node.js-based applications.
2. **Dependencies**: Copies `package.json` and installs the dependencies.
3. **Build**: Runs the build script using `npm run build`.
4. **Expose Port**: Exposes port `5050` for API access.
5. **Start**: Starts the backend server.

### Admin Frontend Dockerfile (`shopper-admin`)

The admin frontend Dockerfile builds the admin frontend container. Here's the breakdown:

1. **Base Image**: `node:18` is used for building the frontend with Vite.
2. **Dependencies**: Installs frontend dependencies with `npm install`.
3. **Build**: Builds the frontend using `npm run build`.
4. **Serve**: Installs the `serve` package to serve static files.
5. **Expose Port**: Exposes port `5174` for frontend access.
6. **Start**: Runs the frontend with `serve`.

### User Frontend Dockerfile (`shopper-frontend`)

The user frontend Dockerfile builds the user frontend container. Here's the breakdown:

1. **Base Image**: `node:18` is used for building the frontend.
2. **Dependencies**: Installs user frontend dependencies using `npm install`.
3. **Build**: Builds the user-facing frontend using `npm run build`.
4. **Serve**: Installs `serve` for serving the user frontend.
5. **Expose Port**: Exposes port `5173` for frontend access.
6. **Start**: Runs the user frontend with `serve`.

---

## Troubleshooting

### Common Issues

1. **Ports Conflict**: Ensure that the user frontend is exposed on `5173`, the admin frontend on `5174`, and the backend on `5050`.
2. **Network Issues**: Both containers should be connected to the same Docker network (`demo`), allowing them to communicate using container names.
3. **Failed API Requests**: Ensure that the frontend is making API requests to the backend container using its name (`http://backend:5050/api/endpoint`).

### Inspecting the Network

If you're facing issues with the network, check the network connections using:

```bash
docker network inspect demo
```

This will display details about the containers connected to the `demo` network.

---

## Conclusion

Following the above steps will allow you to set up, build, and run all components (frontend, admin frontend, and backend) of the Shopper application. If you encounter any issues, check the logs and ensure all containers are correctly configured to communicate with each other via the shared Docker network.

---
To stop and delete an existing container in Docker, follow these steps:

### 1. **Stop the Running Container**

You can stop a running container using the `docker stop` command followed by the container name or container ID:

```bash
docker stop <container-name-or-id>
```

For example, if you want to stop the `shopper-backend` container:

```bash
docker stop shopper-backend
```

### 2. **Remove the Stopped Container**

Once the container is stopped, you can remove it using the `docker rm` command:

```bash
docker rm <container-name-or-id>
```

For example, to remove the `shopper-backend` container:

```bash
docker rm shopper-backend
```

### 3. **(Optional) Remove the Docker Image**

If you want to remove the Docker image associated with the container (which you built earlier), you can use the `docker rmi` command:

```bash
docker rmi <image-name-or-id>
```

For example, to remove the `shopper-backend` image:

```bash
docker rmi shopper-backend
```

### 4. **(Optional) Prune Unused Containers, Networks, and Volumes**

If you want to clean up unused containers, networks, volumes, and images, you can run:

```bash
docker system prune
```

This will remove all stopped containers, unused networks, and dangling images. You will be prompted to confirm.

---

### Summary of Commands:

1. **Stop container**:

   ```bash
   docker stop <container-name-or-id>
   ```

2. **Remove container**:

   ```bash
   docker rm <container-name-or-id>
   ```

3. **Remove image** (optional):

   ```bash
   docker rmi <image-name-or-id>
   ```

4. **Prune system** (optional, to clean up unused resources):

   ```bash
   docker system prune
   ```

Let me know if you need further assistance!

