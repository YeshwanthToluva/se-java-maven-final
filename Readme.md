git clone https://github.com/YeshwanthToluva/se-java-maven-final
cd se-java-maven-final
docker build -t spring-app-final:latest .  
docker run -d --name spring-app-run -p 7777:8080 spring-app-final:latest





Thats is 

for aws use this 
#!/bin/bash
#
# EC2 User Data Script for Ubuntu
# Automates Docker installation, cloning a Git repo,
# building a Docker image, and running the container.

# Exit immediately if a command exits with a non-zero status
set -e

# --- Configuration Variables ---
REPO_URL="https://github.com/Edigirala-Neksha/se-lab2-web-maven.git"
APP_DIR="/opt/se-lab2-web-maven"
IMAGE_NAME="se-lab2-web-app"
CONTAINER_NAME="se-lab2-container"
HOST_PORT="9090"
CONTAINER_PORT="8080"

echo "--- Starting Automated Setup Script ---"

# 1. Update package list
echo "Updating package list..."
echo "+ sudo apt-get update -y"
sudo apt-get update -y

# 2. Install Docker and Git (Git is required for cloning the repository)
echo "Installing Docker and Git..."
echo "+ sudo apt install -y docker.io git"
sudo apt install -y docker.io git

# 3. Ensure Docker service is running and enabled
echo "Starting and enabling Docker service..."
echo "+ sudo systemctl start docker"
sudo systemctl start docker
echo "+ sudo systemctl enable docker"
sudo systemctl enable docker

# 4. Clone the Git repository
echo "Cloning the web application repository from $REPO_URL..."
if [ -d "$APP_DIR" ]; then
    echo "Directory $APP_DIR already exists. Removing it to perform a fresh clone."
    echo "+ sudo rm -rf $APP_DIR"
    sudo rm -rf "$APP_DIR"
fi
# FIX: Explicitly using 'sudo' here ensures permission to create the directory in /opt
echo "+ sudo git clone $REPO_URL $APP_DIR"
sudo git clone "$REPO_URL" "$APP_DIR"

# 5. Build the Docker image
echo "Changing directory to $APP_DIR and building Docker image: $IMAGE_NAME..."
# Note: We use the full path to build the image
echo "+ sudo docker build -t $IMAGE_NAME $APP_DIR"
sudo docker build -t "$IMAGE_NAME" "$APP_DIR"

# 6. Run the Docker container
echo "Running the container on port $HOST_PORT (mapped to $CONTAINER_PORT inside container)..."

# Stop and remove any pre-existing container with the same name
if sudo docker ps -a | grep -q "$CONTAINER_NAME"; then
    echo "Stopping and removing existing container: $CONTAINER_NAME"
    echo "+ sudo docker stop $CONTAINER_NAME"
    sudo docker stop "$CONTAINER_NAME" || true
    echo "+ sudo docker rm $CONTAINER_NAME"
    sudo docker rm "$CONTAINER_NAME" || true
fi

# Run the new container
RUN_CMD="sudo docker run -d -p $HOST_PORT:$CONTAINER_PORT --name $CONTAINER_NAME $IMAGE_NAME"
echo "+ $RUN_CMD"
sudo docker run -d \
    -p "$HOST_PORT":"$CONTAINER_PORT" \
    --name "$CONTAINER_NAME" \
    "$IMAGE_NAME"

echo "--- Setup Complete! ---"
echo "Web application container is running. You can check its status with: sudo docker ps"
echo "Access the application via your EC2 public IP on port $HOST_PORT (e.g., http://<EC2-IP>:$HOST_PORT)"
