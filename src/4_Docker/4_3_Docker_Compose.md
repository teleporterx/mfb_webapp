# Docker Compose

> The `docker-compose.yml` file defines a multi-service setup for a web application with a **MongoDB database, FastAPI backend, and NextJs frontend**. 

Also, since we'd be specifying the API keys etc. in this file, we no longer need the internal `.env` (and associated `python-dotenv`) setup. 

Here's a brief breakdown of each service:

**mongoose:**

- Runs the **MongoDB** container (`mongo:latest`).
- Exposes port `27017` to the host.
- Persists data using a named volume (`mongoose_data`).
- Automatically restarts unless manually stopped.

**mfb_webapp_backend:**

- Runs the backend service (`mfb_webapp_backend` image).
- Exposes port `8000` to the host for **Uvicorn (FastAPI server)**.
- Sets environment variables for MongoDB connection and API keys.
- Depends on MongoDB (`mongoose`) to ensure it starts first.

**mfb_webapp_frontend:**

- Runs the frontend service (`mfb_webapp_frontend` image).
- Exposes port `5000` for the **Next.js development server**.
- Sets environment variables to connect to the backend.
- Depends on the backend (`mfb_webapp_backend`) to ensure it starts first.

**Volumes:**

- Defines `mongoose_data` as a **persistent volume** for MongoDB data.

This setup ensures the services are correctly initialized and interconnected, with dependencies managed between MongoDB, backend, and frontend.

## docker-compose.yml

- Please use the updated Dockerfile in the Usage section of this guid. The following Dockerfile is for reference only.

```yml
services:
  mongoose:
    image: mongo:latest
    container_name: mongoose
    ports:
      - "27017:27017"  # MongoDB default port
    restart: unless-stopped
    volumes:
      - mongoose_data:/data/db  # Persist MongoDB data

  mfb_webapp_backend:
    image: mfb_webapp_backend
    container_name: mfb_webapp_backend
    ports:
      - "8000:8000"  # Expose Uvicorn server
    restart: unless-stopped
    environment:
      MONGO_URL: mongoose
      JWT_SECRET_KEY: "mysupersecretjwtkey"
      RAPID_MUT_FUND_KEY: "[REDACTED]"
      RAPID_URL: "https://latest-mutual-fund-nav.p.rapidapi.com"
    depends_on:
      - mongoose  # Ensure MongoDB starts before the backend

  mfb_webapp_frontend:
    # build:
    #   context: ./  # Adjust the context path if the Dockerfile is in a subdirectory
    #   dockerfile: Dockerfile  # Ensure this matches the frontend Dockerfile name
    image: mfb_webapp_frontend
    container_name: mfb_webapp_frontend
    ports:
      - "5000:5000"  # Expose Next.js dev server
    restart: unless-stopped
    environment:
      NEXT_PUBLIC_BACKEND_URL: "http://mfb_webapp_backend:8000"  # Ensure the frontend can access the backend
    depends_on:
      - mfb_webapp_backend  # Ensure backend starts before the frontend

volumes:
  mongoose_data:  # Define the named volume
```