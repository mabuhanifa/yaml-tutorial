Creating a Docker environment for a Node.js application with PostgreSQL and MongoDB involves several steps, including setting up Docker containers, defining ports, environment variables, and networks. In this guide, I'll provide a comprehensive Dockerfile and Docker Compose configuration to achieve this setup.

**Dockerfile for Node.js Application**

Let's start with the Dockerfile for your Node.js application. This file is used to build an image containing your Node.js application and its dependencies.

```Dockerfile
# Use an official Node.js runtime as the base image
FROM node:14

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json to the container
COPY package*.json ./

# Install application dependencies
RUN npm install

# Copy the rest of the application code to the container
COPY . .

# Expose the port your Node.js application will run on
EXPOSE 3000

# Define an environment variable (e.g., for database connection)
ENV DB_URL mongodb://mongo:27017/mydb

# Start your Node.js application
CMD ["node", "app.js"]
```

Explanation:

1. We start with the official Node.js base image (you can choose a different version according to your application's requirements).

2. Set the working directory in the container to `/app`, where your application code will be placed.

3. Copy `package.json` and `package-lock.json` to the container. This allows us to install your application's dependencies before copying the rest of the code. This separation helps to utilize Docker's caching mechanism effectively.

4. Run `npm install` to install your application's dependencies.

5. Copy the remaining application code into the container.

6. Expose port 3000 for your Node.js application. This is the port on which your Node.js application will listen for incoming connections.

7. Define an environment variable `DB_URL` to configure the connection to MongoDB. In this example, we set it to `mongodb://mongo:27017/mydb`. We'll use this environment variable in our Node.js application to connect to MongoDB.

8. Finally, the `CMD` instruction specifies the command to run when the container starts. In this case, it runs `node app.js`, assuming your application's entry point is `app.js`.

**Docker Compose for Multi-Container Setup**

To run both PostgreSQL and MongoDB alongside your Node.js application, we'll use Docker Compose. Create a `docker-compose.yml` file to define the services, networks, and environment variables for your containers.

```yaml
version: "3.8"

services:
  node-app:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      DB_URL: mongodb://mongo:27017/mydb
    depends_on:
      - mongo
      - postgres
    networks:
      - my-network

  mongo:
    image: mongo:latest
    ports:
      - "27017:27017"
    networks:
      - my-network

  postgres:
    image: postgres:latest
    environment:
      POSTGRES_USER: your-postgres-username
      POSTGRES_PASSWORD: your-postgres-password
      POSTGRES_DB: your-postgres-database
    ports:
      - "5432:5432"
    networks:
      - my-network

networks:
  my-network:
```

Explanation:

1. We define three services in the `docker-compose.yml` file: `node-app`, `mongo`, and `postgres`. Each service corresponds to a Docker container.

2. The `node-app` service uses the Dockerfile we defined earlier to build an image for your Node.js application. It also specifies the port mapping for your application (3000:3000) and sets the `DB_URL` environment variable.

3. The `mongo` service uses the official MongoDB image and maps port 27017 from the container to the host.

4. The `postgres` service uses the official PostgreSQL image. You can customize the environment variables (POSTGRES_USER, POSTGRES_PASSWORD, POSTGRES_DB) to match your application's requirements. It also maps port 5432 from the container to the host.

5. `depends_on` ensures that the `mongo` and `postgres` services are started before the `node-app` service. However, keep in mind that it doesn't guarantee that the services are fully ready for connections. You might need to implement retry mechanisms in your application to handle this.

6. We define a custom network called `my-network` that all services are connected to. This enables communication between containers on the same network using their service names as hostnames (e.g., `mongo` and `postgres`).

With this Docker Compose configuration, you can create and manage your multi-container environment with a single command: `docker-compose up`.

**Final Thoughts**

In this guide, we've created a Docker environment for a Node.js application with PostgreSQL and MongoDB. We've provided a Dockerfile for the Node.js application and a Docker Compose configuration to orchestrate the containers, set up networking, and configure environment variables. This setup allows you to run a complete development environment with ease, ensuring that your application can interact with both PostgreSQL and MongoDB while running in Docker containers.
