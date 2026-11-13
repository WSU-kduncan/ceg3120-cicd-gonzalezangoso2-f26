# Project 6 - Continuous Integration

## Project Overview

The goal of this project is to containerize a website using Docker and automate the process of building and publishing Docker images with GitHub Actions.

The main tools used are:

- Git and GitHub: manage the project files and version history.
- HTML and CSS: create the website.
- Docker: build and run the website inside a container.
- DockerHub: store the container images.
- GitHub Actions: automate image builds and publishing.

The website has been created and tested locally using Docker. The GitHub Actions workflow and DockerHub integration will be configured in the next parts of the project.

## Part 1 - Docker Container Image

### Website Content

I created a simple website about CEG3120 Project 6.

The website contains two HTML pages and one CSS file:

- [index.html](web-content/index.html): main page of the website.
- [about.html](web-content/about.html): information about the project and technologies used.
- [style.css](web-content/style.css): styling for both pages.

### Dockerfile

I created a [Dockerfile](Dockerfile) in the root directory of the repository.

The Dockerfile contains:

```dockerfile
FROM httpd:2.4

COPY web-content/ /usr/local/apache2/htdocs/
```

The `FROM` instruction uses Apache HTTP Server as the base image.

The `COPY` instruction copies the website files into the default Apache web directory.

### Building the Docker Image

To build the image from the root directory of the repository, I used:

```bash
docker build -t ceg3120-site:local .
```

The image was built successfully.

For DockerHub, images must be tagged using the DockerHub username, repository name, and image tag.

### Running the Container

I started the container using:

```bash
docker run -d --name ceg3120-web -p 8080:80 ceg3120-site:local
```

The command runs the container in the background and maps port 8080 on my computer to port 80 inside the container.

To check the container status, I used:

```bash
docker ps
```

The website is available at:

- http://localhost:8080/index.html
- http://localhost:8080/about.html

### Testing and Validation

**Docker container running**

The following screenshot shows the container running with the correct image and port mapping.

![Docker container status](images/docker-local-ps.png)

**Home page**

This screenshot shows the Home page served by the Docker container with CSS applied.

![Home page running in Docker](images/docker-local-home.png)

**About page**

This screenshot shows the About page served by the Docker container, confirming that both HTML pages and the CSS work correctly.

![About page running in Docker](images/docker-local-about.png)

## Resources

- [CEG3120 Project 6 Instructions](https://github.com/pattonsgirl/CEG3120/tree/main/Projects/Project6) - Used to understand the requirements and project structure.
