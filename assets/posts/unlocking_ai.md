# Unlocking the Power of AI - Running Large Language Models locally in Arch Linux

In the fast-evolving landscape of artificial intelligence, local deployment of AI models has become increasingly crucial for anyone seeking enhanced privacy, and greater control over their data.

In this article, I will present two distinct methodologies for deploying AI models locally, catering to diverse preferences and requirements:

- Normal Installation (on Arch linux)
- Utilizing Docker (+using Docker Compose)

In both aproaches we will use [Ollama](https://ollama.com/) for runnning the model and [open-webui](https://docs.openwebui.com/) as a web UI frontend.

## Installing on Arch Linux

We'll explore the traditional approach of directly installing AI frameworks and libraries on Arch Linux. This method offers a hands-on experience, allowing users to configure and optimize their AI environment according to specific project needs.

We always start by updating the system:

```sh
sudo pacman -Syu
```

### Installing Ollama

Ollama is in the Arch Linux official repositories so we can install it by using pacman:

```sh
sudo pacman -Sy ollama
```

The ollama package has been installed plus a systemd service.

Start the service:

```sh
sudo systemctl start ollama
```

To enable the service so that auto-starts after reboot, use the following command:

```sh
sudo systemctl enable ollama
```

Verify that the service is up and running:

```sh
sudo systemctl status ollama
```

### Downloading a Model

You can see the available modes in the [ollama-library](https://ollama.com/library) and choose the one you like.

In this example we will use **llama3**.

```sh
ollama run llama3
```

This command will pull the model and the 

You can now start chating with llama3!

![ollama run llama3](assets/images/baf9a1f5537a4416b5f7779abd17e0e7.png)

### Using a Web UI

There are multiple web and desktop integrations that you can choose from [ollama-community-integrations](https://github.com/ollama/ollama?tab=readme-ov-file#community-integrations) but in this tutorial we will use the [open-webui](https://docs.openwebui.com/) which I am sure you will be familiar with from the ChatGPT interface.

#### Build and Install

Verify that we have the required packages and install them if not.

```sh
sudo pacman -Sy curl base-devel git npm python-pip
```

Run the following commands to download the package:

```sh
git clone https://github.com/open-webui/open-webui.git
cd open-webui/

# Copying required .env file
cp -RPp .env.example .env
```

Building Frontend Using Node

```sh
npm i
npm run build
```

Now we need to install the required python packages.
To do this we would normally execute the floowing command:

```sh
pip install -r requirements.txt -U
```

but this will result in the following error:

![venv](assets/images/313619a43e634c71be2331bb11e2640f.png)

This error provides an opportunity for users to install Python-specific packages in a virtual environment, isolating them from other packages on their operating system. This helps prevent potential issues
with package conflicts that may be difficult or impossible to recover from. According to the Python Enhancement Protocol website, this error is not intended to create a security barrier, but rather to 
prevent unintended changes from accidentally disrupting a user's working environment."

So we need to create a virtual environment and then install the dependencies.

```sh
cd ./backend

# Create the virtual environment
python -m venv .
source ./bin/activate

# Install the required packages
pip install -r requirements.txt -U

# Serving Frontend with the Backend
bash start.sh
```

![open-webui-running](assets/images/01c5c8e3e51f4d129608aa6ad447fd47.png)

Now you can move to the *Access the Open WebUI* section.

## Utilizing Docker

Alternatively, we'll delve into the utilization of Docker containers to deploy the ollama server and the WebUI frontend.
This is a versatile solution applicable across various operating systems supported by Docker.
By encapsulating the API server dependencies within Docker containers, we achieve portability, reproducibility, and isolation. 
By using docker containers we facilitate seamless deployment and management of the workflows irrespective of the host OS.

### Run Ollama docker

The following commands downloads and runs the latest ollama docker image and maps the container port 11434 to the host's port 11434.

```sh
docker run -d -v ollama:/root/.ollama -p 11434:11434 --name ollama ollama/ollama:latest
```

### Run Open Web-UI docker

Same goes for open-webui

```sh
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

As stated in the openwebui docs:

>If you're experiencing connection issues, it’s often due to the WebUI docker container not being able to reach the Ollama server at 127.0.0.1:11434 (host.docker.internal:11434) inside the container . Use the --network=host flag in your docker command to resolve this. Note that the port changes from 3000 to 8080, resulting in the link: http://localhost:8080.

```sh
docker run -d --network=host -v open-webui:/app/backend/data -e OLLAMA_BASE_URL=http://127.0.0.1:11434 --name open-webui --restart always ghcr.io/open-webui/open-webui:main
```

Now you can move to the *Access the Open WebUI* section.

## Using Docker Compose (easy way)

Using docker-compose for easy installation is the way to go

Just create a `docker-compose.yaml` configuration file and paste the following:

```yaml
version: '3.8'

services:
  ollama:
    volumes:
      - ollama:/root/.ollama
    container_name: ollama
    pull_policy: always
    tty: true
    restart: unless-stopped
    image: ollama/ollama:${OLLAMA_DOCKER_TAG-latest}

  open-webui:
    build:
      context: .
      args:
        OLLAMA_BASE_URL: '/ollama'
      dockerfile: Dockerfile
    image: ghcr.io/open-webui/open-webui:${WEBUI_DOCKER_TAG-main}
    container_name: open-webui
    volumes:
      - open-webui:/app/backend/data
    depends_on:
      - ollama
    ports:
      - ${OPEN_WEBUI_PORT-3000}:8080
    environment:
      - 'OLLAMA_BASE_URL=http://ollama:11434'
      - 'WEBUI_SECRET_KEY='
    extra_hosts:
      - host.docker.internal:host-gateway
    restart: unless-stopped

volumes:
  ollama: {}
  open-webui: {}
```

This Compose file defines both ollama and open-webui services.
Now with a single command, you create and start all the services from your configuration file.

`docker compose up -d`

## Access the Open WebUI

You should have Open WebUI up and running at [http://localhost:8080/](http://localhost:8080/) .

Open a web browser and navigate to [http://localhost:8080](http://localhost:8080)

![Open WebUI](assets/images/1711bed76b4449e8afad2ac56e4e36d9.png)

Inside your local network you can use your ip to access the webui.

e.g. `http://192.168.1.31:8080`

![Open WebUI](assets/images/283cd1c6578d4e62aa24a45f8bd64686.png)

Now you can use the UI to pull the preferred models from the Settings menu.

Enjoy! 😁



## Resources:

- [ollama](https://github.com/ollama/ollama)
- [ollama-docker-image](https://hub.docker.com/r/ollama/ollama)
- [open-webui-getting-started](https://docs.openwebui.com/getting-started/)
- [open-webui](https://github.com/open-webui/open-webui)
