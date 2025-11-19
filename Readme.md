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




this another one
#!/bin/bash
# Script to automate the full Nagios container lifecycle: Pull, Run, Status, Stop, and Cleanup.

# --- Configuration Variables ---
NAGIOS_IMAGE="jasonrivers/nagios:latest"
NAGIOS_CONTAINER_NAME="nagiosdemo"
HOST_PORT="9091"
CONTAINER_PORT="80"
NAGIOS_URL="http://localhost:$HOST_PORT"
NAGIOS_USER="nagiosadmin"
NAGIOS_PASS="nagios"

echo "========================================="
echo "   🚀 Automated Nagios Deployment Script "
echo "========================================="

# --- 1. Pull Image (Ensures the latest is available) ---
echo "## 1. Pulling Nagios Image"
echo "---"
if docker pull $NAGIOS_IMAGE; then
    echo "Image '$NAGIOS_IMAGE' pulled successfully."
else
    echo "ERROR: Failed to pull Docker image. Check your internet connection or repository name."
    exit 1
fi

# --- 2. Run Container ---
echo ""
echo "## 2. Running Nagios Container on Port $HOST_PORT"
echo "---"

# Check for and remove any existing container named 'nagiosdemo' (stopped or failed)
if docker ps -a --filter name=$NAGIOS_CONTAINER_NAME | grep -q $NAGIOS_CONTAINER_NAME; then
    echo "Found existing container named '$NAGIOS_CONTAINER_NAME'. Removing it for a fresh run..."
    docker rm -f $NAGIOS_CONTAINER_NAME
fi

# Check if the desired port is currently allocated by a non-Docker process
if sudo ss -tuln | grep -q ":$HOST_PORT "; then
    # We allow the script to continue here, as Docker's internal checks are robust, 
    # but give a warning if a non-Docker process is using it.
    echo "WARNING: Local port $HOST_PORT appears to be in use by another application."
fi

# Run the container in detached mode
echo "Starting container '$NAGIOS_CONTAINER_NAME'..."
if docker run -d --name $NAGIOS_CONTAINER_NAME -p $HOST_PORT:$CONTAINER_PORT $NAGIOS_IMAGE; then
    echo "Container started successfully."
else
    echo "FATAL ERROR: Container failed to start. Port $HOST_PORT may still be in use."
    exit 1
fi

# --- 3. Verification and Access ---
echo ""
echo "## 3. Verification and Access"
echo "---"

# Show running containers (as per your checklist)
docker ps

# Access information
echo "Nagios is accessible at: $NAGIOS_URL"
echo "Default Credentials: Username: $NAGIOS_USER, Password: $NAGIOS_PASS"
echo "To access the web UI, you would typically run 'open $NAGIOS_URL' or navigate there manually."

# --- 4. Cleanup/Stop (Instructions) ---
echo ""
echo "## 4. Cleanup Instructions"
echo "---"

echo "To stop the running Nagios container (as per your checklist):"
echo "   $ docker stop $NAGIOS_CONTAINER_NAME"
echo ""

echo "To verify it is stopped (as per your checklist):"
echo "   $ docker ps"
echo ""

echo "To delete the downloaded image and reclaim disk space (as per your checklist):"
echo "   $ docker rmi $NAGIOS_IMAGE"

echo "========================================="
echo "      Script Execution Finished!         "
echo "========================================="