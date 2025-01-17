# Setup & Usage

## Methods:
1. Using `docker-compose` (Recommended)
2. Manual setup


### 1. Using `docker-compose`

`docker-compose` can be used to orchestrate the deployment of all the necessary services for this application. By running a single command, all components, including the MongoDB database, FastAPI backend, and Next.js frontend, are automatically pulled (from Dockerhub), configured, and started.


**Requirements**
Make sure you have **Docker** and `docker-compose` installed on your deployment target

```bash
# For Debian based distros
sudo apt update && sudo apt upgrade && sudo apt install docker.io docker-compose
# For Arch based distros
sudo pacman -Syu && sudo pacman -S docker docker-compose
```

**Orchestration**

- Copy over the following to `docker-compose.yml` on your deployment target.

- If you are serving the App on a cloud instance, then make sure to replace the public IP of that instance in the following `docker-compose.yml` file. You will face CORS issues if this is not done.

- Watch out for the port numbers!

- For testing on localhost, no changes are necessary other than adding your RapidAPI key for the `Latest Mutal Fund Nav` service.

```yml
services:
  mongoose:
    image: mongo:latest
    container_name: mongoose
    ports:
      - "27017:27017"
    restart: unless-stopped
    volumes:
      - mongoose_data:/data/db
    networks:
      - mfb-network

  mfb_webapp_backend:
    image: teleporterx/mfb_webapp_backend:latest
    container_name: mfb_webapp_backend
    ports:
      - "8000:8000"
    restart: unless-stopped
    environment:
      MONGO_URL: mongoose
      # PUBLIC_FRONTEND_URL: "http://36.213.136.122:5000"  # Set the public frontend URL
      PUBLIC_FRONTEND_URL: "http://localhost:5000"  # Set the public frontend URL
      JWT_SECRET_KEY: "mysupersecretjwtkey"
      RAPID_MUT_FUND_KEY: "insertyourapikey"
      RAPID_URL: "https://latest-mutual-fund-nav.p.rapidapi.com"
    depends_on:
      - mongoose
    networks:
      - mfb-network  # Attach to custom network

  mfb_webapp_frontend:
    image: teleporterx/mfb_webapp_frontend:latest
    container_name: mfb_webapp_frontend
    ports:
      - "5000:5000"
    restart: unless-stopped
    environment:
      # NEXT_PUBLIC_BACKEND_URL: "http://36.213.136.122:8000"  # This will work inside the Docker network
      NEXT_PUBLIC_BACKEND_URL: "http://localhost:8000"  # This will work inside the Docker network
    depends_on:
      - mfb_webapp_backend
    networks:
      - mfb-network  # Attach to custom network

volumes:
  mongoose_data:

networks:
  mfb-network:  # Define custom network
    driver: bridge  # Use bridge driver for networking
```

- Run the docker-compose on the same level as the `docker-compose.yml` file

```bash
sudo docker-compose up -d
```

Heres an example of how a successful run would look like
```bash
$ sudo docker-compose up -d
[+] Running 4/4
 ✔ Network bhive_default          Cre...                         0.1s
 ✔ Container mongoose             Starte...                      0.4s
 ✔ Container mfb_webapp_backend   Started                        0.5s
 ✔ Container mfb_webapp_frontend  Started                        0.7s
```

To stop the containers use 

```bash
sudo docker-compose down
```

### 2. Manual Setup

**Mongo**

Local setup
```bash
wget -qO - https://www.mongodb.org/static/pgp/server-6.0.asc | sudo tee /etc/apt/trusted.gpg.d/mongodb.asc
echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/6.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-6.0.list
sudo apt update
sudo apt install -y mongodb-org
sudo systemctl status mongod # verify MongoDB is running
```

Docker
```bash
sudo docker run -d --name mongodb -p 27017:27017 mongo:latest
```

**Backend**

```bash
git clone https://github.com/teleporterx/mfb_webapp_backend.git
cd mfb_webapp_backend
py -m venv .venv
source .venv/bin/activate
pip install poetry
poetry install
uvicorn main:app --host 0.0.0.0
```

**Frontend**

```bash
git clone https://github.com/teleporterx/mfb_webapp_frontend.git
cd mfb_webapp_frontend
npm install
npm run dev
```

**The `.env` file**

For the manual setup this `.env` file containing the MongoDB connection string and the secret key for the backend should be created at the root of the project.

```env
MONGO_URL=localhost:27017
JWT_SECRET_KEY=mysupersecretjwtkey
RAPID_MUT_FUND_KEY=insertyourkeyhere
RAPID_URL=https://latest-mutual-fund-nav.p.rapidapi.com
```