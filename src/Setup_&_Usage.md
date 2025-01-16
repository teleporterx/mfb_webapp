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

- Copy over the following to `docker-compose.yml` on your deployment target

```yml
```

- Run the docker-compose on the same level as the `docker-compose.yml` file

```bash
sudo docker-compose up -d
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