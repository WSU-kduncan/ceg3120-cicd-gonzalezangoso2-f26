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


### Publishing the Image to DockerHub

I created a public DockerHub repository to store my container images.

DockerHub repository:
[ignaciooga/ceg3120-site](https://hub.docker.com/r/ignaciooga/ceg3120-site)

First, I tagged my local Docker image using my DockerHub username and repository name:

```bash
docker tag ceg3120-site:local ignaciooga/ceg3120-site:latest
```

I logged in to DockerHub using my username and a Personal Access Token with Read & Write permissions:

```bash
docker login -u ignaciooga
```

The PAT was entered at the password prompt and was not stored in the GitHub repository.

Then I pushed the image to DockerHub:

```bash
docker push ignaciooga/ceg3120-site:latest
```

The image was successfully published with the `latest` tag.

### Pulling and Testing the DockerHub Image

To verify the published image, I used:

```bash
docker pull ignaciooga/ceg3120-site:latest
```

Then I started a new container using the DockerHub image:

```bash
docker run -d --name ceg3120-from-hub -p 8081:80 ignaciooga/ceg3120-site:latest
```

I used port 8081 because my original local container was already running on port 8080.

To verify the running containers, I executed:

```bash
docker ps
```

The following screenshot shows the successful DockerHub push and pull commands and the running container using port 8081.

![DockerHub pull and container test](images/dockerhub-pull-test.png)

### DockerHub Image Validation

**Home page from DockerHub image**

The following screenshot shows the Home page running from the DockerHub image on port 8081 with the CSS applied.

![DockerHub Home page](images/dockerhub-home.png)

**About page from DockerHub image**

This screenshot shows the About page running from the same container, confirming that both HTML pages and the CSS work.

![DockerHub About page](images/dockerhub-about.png)

Both pages worked correctly when running the published DockerHub image.


## Part 2 - GitHub Actions and DockerHub

### GitHub Repository Secrets

I configured two repository secrets in GitHub Actions:

- `DOCKER_USERNAME`: stores my DockerHub username.
- `DOCKER_TOKEN`: stores my DockerHub Personal Access Token.

I created a DockerHub PAT with Read & Write permissions. These secrets allow GitHub Actions to authenticate with DockerHub without exposing credentials in the workflow file.

The following screenshot shows both repository secrets configured in GitHub.

![GitHub Actions repository secrets](images/github-secrets.png)


### GitHub Actions Workflow

I created a GitHub Actions workflow called `docker-publish.yml` to automatically build and push Docker images to DockerHub.

The workflow is triggered when a commit is pushed to the `main` branch.

The workflow performs these steps:

1. Checks out the GitHub repository.
2. Sets up Docker Buildx.
3. Logs in to DockerHub using repository secrets.
4. Builds the Docker image using the Dockerfile.
5. Pushes the image to `ignaciooga/ceg3120-site:latest`.

### Testing the Main Branch Trigger

I pushed commit `19d9fce` to the `main` branch, which automatically triggered the workflow.

After correcting the DockerHub authentication, I re-ran the workflow successfully.

The following screenshot shows the successful execution of GitHub Actions, including the commit SHA and the `main` branch.

![Successful GitHub Actions workflow](images/github-main-success.png)


### Testing a Different Branch

To verify that the workflow only runs on the `main` branch, I created a new branch called `test-no-ci`.

I added a file called `branch-test.txt` and committed the changes.

Then I pushed the new branch to GitHub using:

```bash
git push -u origin test-no-ci
```

The following screenshot shows the successful push to the test branch.

![Test branch push](images/github-branch-push.png)

After pushing the changes, I checked the GitHub Actions page.

No new workflow run was triggered by the `test-no-ci` branch. Only the previous runs from `main` appeared.

![GitHub Actions branch filtering](images/github-branch-filter.png)

This confirms that the workflow is correctly configured to run only when changes are pushed to the `main` branch.

After completing the test, I switched back to `main` to continue working on the project.


### Testing Automatic Docker Image Updates

To test the automatic image publishing process, I updated the `index.html` file by adding the following message:

"Update: This website is automatically published using GitHub Actions!"

I committed the change and pushed it to the `main` branch.

Commit: `7dec80b`

GitHub Actions automatically started the Docker Image CI workflow. The workflow successfully built and pushed the updated Docker image to DockerHub.

The following screenshot shows the successful workflow execution.

![Automatic update workflow](images/github-auto-update-success.png)

### Pulling the Updated Image

After the workflow completed, I downloaded the updated image from DockerHub.

Since my Mac uses ARM64 and the GitHub Actions image was built for AMD64, I specified the platform:

```bash
docker pull --platform linux/amd64 ignaciooga/ceg3120-site:latest
```

Then I started a new container using port 8082:

```bash
docker run -d --platform linux/amd64 --name ceg3120-updated -p 8082:80 ignaciooga/ceg3120-site:latest
```

I used `docker ps` to verify that the container was running.

![Updated Docker container](images/github-auto-update-docker.png)

### Verifying the Website Update

I opened `http://localhost:8082/index.html` in my browser.

The updated message appeared on the Home page, confirming that the image published by GitHub Actions contained my latest website changes.

![Updated website](images/github-auto-update-home.png)

This test confirms that pushing changes to the `main` branch automatically builds and publishes an updated Docker image to DockerHub.


### Part 2 Workflow Permanent Link

The following link points to the GitHub Actions workflow used in Part 2 of this project.

[Part 2 - Docker Image CI Workflow](https://github.com/WSU-kduncan/ceg3120-cicd-gonzalezangoso2-f26/blob/ba43acf5fcf0aca37d5bb0f7a525f3e8c5c3bc9a/.github/workflows/docker-publish.yml)

This version of the workflow builds and pushes the Docker image to DockerHub whenever changes are pushed to the `main` branch.

I saved this permanent link before making changes for Part 3, so the original workflow can still be reviewed.


## Part 3 - Semantic Versioning

### Configuring Semantic Versioning

In this part, I modified the GitHub Actions workflow to use semantic versioning for Docker images.

Instead of running on every push to `main`, the workflow now runs when a version tag matching `v*.*.*` is pushed to GitHub.

I added `docker/metadata-action@v6` to automatically generate Docker image tags based on the Git version.

The workflow uses the following tags:

- `1`: major version.
- `1.0` or `1.1`: major and minor version.
- `latest`: most recently published version.

The generated tags and labels are passed to `docker/build-push-action@v7`, which builds and publishes the Docker image to DockerHub.

### Publishing Version 1.0.0

I created the first version using these commands:

```bash
git tag -a v1.0.0 -m "Release version 1.0.0"
git push origin v1.0.0
```

Pushing this tag automatically triggered the GitHub Actions workflow.

The workflow completed successfully and published the image with the tags `1`, `1.0`, and `latest`.

![GitHub Actions v1.0.0](images/github-semver-v1-success.png)

![DockerHub v1.0.0 tags](images/dockerhub-semver-v1-tags.png)

### Publishing Version 1.1.0

For the second version, I modified `web-content/index.html` to add a new section called "What's New in Version 1.1.0".

After committing and pushing the change to `main`, I created and published the new version:

```bash
git tag -a v1.1.0 -m "Release version 1.1.0"
git push origin v1.1.0
```

GitHub Actions automatically built and published the updated Docker image.

![GitHub Actions v1.1.0](images/github-semver-v1-1-success.png)

![DockerHub v1.1.0 tags](images/dockerhub-semver-v1-1-tags.png)

### Verifying Docker Image Versions

After publishing version 1.1.0, I checked the DockerHub repository.

The tags `latest`, `1`, and `1.1` point to the new version and share the same digest.

The `1.0` tag still points to the previous image with a different digest.

This confirms that semantic versioning works correctly and that older Docker image versions remain available.

### Conclusion

I successfully configured GitHub Actions to automatically build and publish versioned Docker images to DockerHub.

The workflow is now triggered by Git version tags instead of normal pushes to the `main` branch.


### Testing the Tag Trigger

I tested that the workflow only runs when a version tag is pushed.

After pushing commit `1e96073` to `main` without creating a new version tag, I checked GitHub Actions.

No new workflow execution appeared. The latest execution was still the successful `v1.1.0` release.

![GitHub Actions without a new run](images/github-tag-negative-actions.png)

![Commit without a version tag](images/github-tag-negative-commit.png)

### Testing Docker Image Rollback

I tested that both the previous and current Docker image versions could run independently.

First, I downloaded and started version `1.0`:

```bash
docker pull --platform linux/amd64 ignaciooga/ceg3120-site:1.0
docker run -d --platform linux/amd64 --name ceg3120-v1-0 -p 8083:80 ignaciooga/ceg3120-site:1.0
```

The website at `http://localhost:8083/index.html` displayed the original content without the version 1.1.0 section.

![Docker version 1.0](images/docker-rollback-v1-0.png)

Next, I downloaded and started version `1.1`:

```bash
docker pull --platform linux/amd64 ignaciooga/ceg3120-site:1.1
docker run -d --platform linux/amd64 --name ceg3120-v1-1 -p 8084:80 ignaciooga/ceg3120-site:1.1
```

The website at `http://localhost:8084/index.html` included the new "What's New in Version 1.1.0" section.

![Docker version 1.1](images/docker-rollback-v1-1.png)

I also used `docker ps` to verify that both containers were running at the same time on different ports.

This confirms that previous Docker image versions remain available and can be used for rollback.
