# Frontend Dockerfile

> The Dockerfile defines the build process for the **Next.js frontend** of the web application. It sets up the necessary environment to run the frontend in a Docker container, including installing dependencies and starting the development server.

Here’s a brief explanation of the `Dockerfile` for the `mfb_webapp_frontend` service:

1. **Base Image**:
   - `FROM node:23.4.0-alpine`: This sets the base image as a lightweight version of Node.js (version `23.4.0-alpine`) to optimize image size.

2. **Working Directory**:
   - `WORKDIR /app`: Sets `/app` as the working directory inside the container. All subsequent commands will run in this directory.

3. **Copy Dependency Files**:
   - `COPY package.json package-lock.json ./`: Copies `package.json` and `package-lock.json` files into the `/app` directory of the container. These files define the project dependencies.

4. **Install Dependencies**:
   - `RUN npm install`: Installs the project dependencies specified in `package.json`.

5. **Copy Application Files**:
   - `COPY . .`: Copies the rest of the application files (e.g., frontend code) from the local machine into the `/app` directory in the container.

6. **Expose Port**:
   - `EXPOSE 5000`: Exposes port `5000` from the container, which is the port the Next.js development server will run on.

7. **Start the Application**:
   - `CMD ["npm", "run", "dev"]`: Runs the `npm run dev` command to start the development server (for Next.js), enabling the frontend application to serve on port 5000.

This `Dockerfile` sets up the environment and dependencies for the frontend and ensures the Next.js app is ready to run within the container.

## Dockerfile

```yml
# Use an official Node.js image as the base image
FROM node:23.4.0-alpine

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to the working directory
COPY package.json package-lock.json ./

# Install dependencies
RUN npm install

# Copy the rest of the application files
COPY . .

# Expose the application port (5000 as specified in your script)
EXPOSE 5000

# Run the development server
CMD ["npm", "run", "dev"]
```