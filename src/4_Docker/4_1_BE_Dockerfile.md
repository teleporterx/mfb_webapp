# Backend Dockerfile

> The Dockerfile defines the build process for the **FastAPI backend** of the web application. It sets up the necessary environment, installs dependencies via Poetry, and runs the backend application using Uvicorn.

Since the backend connects to services like MongoDB (via the MONGO_URL environment variable) and other API keys are handled in the docker-compose.yml file, there is no need for a separate .env file in this Dockerfile.

Here’s the breakdown of the `Dockerfile` for the **FastAPI backend** (`mfb_webapp_backend`):

1. **Base Image**:
   - `FROM python:3.13-slim`: Uses the official Python 3.13 slim image as the base to create a lightweight Python environment.

2. **Set Working Directory**:
   - `WORKDIR /app`: Sets `/app` as the working directory inside the container. All subsequent commands will execute relative to this directory.

3. **Copy Dependency Files**:
   - `COPY pyproject.toml poetry.lock /app/`: Copies `pyproject.toml` and `poetry.lock` files to the container to define project dependencies.

4. **Install Poetry and Dependencies**:
   - `RUN python -m venv .venv && \`: Creates a Python virtual environment inside the container.
   - `. .venv/bin/activate && \`: Activates the virtual environment.
   - `pip install --upgrade pip && \`: Upgrades `pip` to the latest version.
   - `pip install poetry && \`: Installs Poetry, a Python dependency management tool.
   - `poetry install`: Installs the dependencies defined in the `pyproject.toml` file using Poetry.

5. **Set Virtual Environment Variables**:
   - `ENV VIRTUAL_ENV=/app/.venv`: Sets the `VIRTUAL_ENV` environment variable to the virtual environment directory.
   - `ENV PATH="$VIRTUAL_ENV/bin:$PATH"`: Adds the virtual environment's `bin` directory to the `PATH`, ensuring the correct Python and `pip` executables are used.

6. **Set Default Environment Variables**:
   - `ENV MONGO_URL=mongoose`: Sets a default environment variable for MongoDB connection (`mongoose`), which can be overridden in `docker-compose.yml`.

7. **Copy Application Files**:
   - `COPY api /app/api`: Copies the `api` directory into the container, containing the FastAPI app code.
   - `COPY main.py /app/main.py`: Copies the `main.py` file, which contains the FastAPI application instance.

8. **Expose Application Port**:
   - `EXPOSE 8000`: Exposes port `8000` for the FastAPI backend, which will be used by the Uvicorn server.

9. **Run the Application**:
   - `CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]`: Starts the application using Uvicorn, binding to `0.0.0.0` so it can accept requests from outside the container, and runs the app on port `8000`.

This `Dockerfile` builds the backend environment, installs dependencies, and starts the FastAPI application ready to serve requests.

## Dockerfile

```yml
# Use an official Python runtime as a parent image
FROM python:3.13-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the pyproject.toml and poetry.lock files first to leverage Docker cache
COPY pyproject.toml poetry.lock /app/

# Install Poetry and create a virtual environment, then install dependencies
RUN python -m venv .venv && \
    . .venv/bin/activate && \
    pip install --upgrade pip && \
    pip install poetry && \
    poetry install

# Set environment variables for the virtual environment
ENV VIRTUAL_ENV=/app/.venv
ENV PATH="$VIRTUAL_ENV/bin:$PATH"

# Set default values for RabbitMQ and MongoDB host (this can be overridden in docker-compose or environment files)
ENV MONGO_URL=mongoose

# Copy the 'api' directory into the container
COPY api /app/api
COPY main.py /app/main.py

# Expose the port the app will run on
EXPOSE 8000

# Run the application using Uvicorn, inside the virtual environment
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8000"]
```