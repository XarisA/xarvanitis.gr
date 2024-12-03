# Self-Hosting Stable Diffusion Models in Linux

Stable diffusion, also known as stable diffusion models or SDMs for short, are a type of generative model that has gained significant attention in the field of artificial intelligence and computer vision. The main idea behind stable diffusion is to generate new images, videos, or other types of data by iteratively refining an input noise signal until it converges to a specific output.

This guide provides a step-by-step instructions to self-host Stable Diffusion models on linux. You can choose between two installation methods: using Docker-Compose or installing manually. 

## Install and run Stable Diffussion WebUI (docker-compose)

Clone the repository from [stable-diffusion-webui-docker](https://github.com/AbdBarho/stable-diffusion-webui-docker)

```bash
git clone https://github.com/AbdBarho/stable-diffusion-webui-docker.git
cd stable-diffusion-webui-docker
```
Then build the essential:

```bash
docker compose --profile download up --build
# wait until its done, then:
docker compose --profile [ui] up --build
# where [ui] is one of: invoke | auto | auto-cpu | comfy | comfy-cpu
# if you don't know which ui to choose, invoke or auto are a good start.
# eg. docker compose --profile auto up --build
```

Open in browser the server address eg
[localhost:7860](localhost:7860)

You can now move to the Web UI section.

## Install and run Stable Diffussion WebUI (manually)

Install required dependecies

```bash
sudo pacman -S wget git python3
```

*For other distributions you can alternatively run the following commands*
```bash
# Debian-based:
sudo apt install wget git python3 python3-venv libgl1 libglib2.0-0
# Red Hat-based:
sudo dnf install wget git python3 gperftools-libs libglvnd-glx 
# openSUSE-based:
sudo zypper install wget git python3 libtcmalloc4 libglvnd
# Arch-based:
sudo pacman -S wget git python3
```

Navigate to the directory you would like the webui to be installed and execute the following command to clone the repository

```bash
git clone https://github.com/AUTOMATIC1111/stable-diffusion-webui.git
```
create a python virtual environment and activate it

```bash
cd stable-diffusion-webui
python -m venv .
source ./bin/activate
```

make the script executable and run it
```bash
chmod +x webui.sh
# The --api command line argument is used for using stable diffusion in ollama web ui
./webui.sh --api
```

You can now move to the Web UI section.

## Web UI
Open in browser the server address eg [localhost:7860](localhost:7860) or through your network ip.

Prompt and fire Generate! 

![stable diffusion web ui](assets/images/178c91cc63af4824b8a7217f52e11833.png)

Enjoy 😁!


## Use Stable Diffusion with Ollama

Linking Stable Diffusion with ollama is pretty straight forward.

Change the following settings in ollama web ui.

![link stable diffusion in ollama](assets/images/86683f98c41141ec9e14e9f66be101fc.png)

Now you can press the new image button to create an image from the given prompt.

A good tip here is to ask the model to create a prompt for you or use a model specialized in creating prompts like ***brxce/stable-diffusion-prompt-generator***.

![ollama generate images](assets/images/7ccfbf070c074b599a1df38fd752f992.png)

As you can see using the stable-diffusion api and overriding the parameters, results in much better generations.

## Troubleshooting

Most of the issues that occured, are described in the troubleshooting section of the [AUTOMATIC1111/stable-diffusion-webui/wiki/Troubleshooting](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Troubleshooting) so feel free to check there.

### NansException

```text
NansException: A tensor with all NaNs was produced in Unet. This could be either because 
there's not enough precision to represent the picture, or because your video card does not 
support half type. Try setting the "Upcast cross attention layer to float32" option in 
Settings > Stable Diffusion or using the --no-half commandline argument to fix this. 
Use --disable-nan-check commandline argument to disable this check.
```

This is known [Bug: NansException](https://github.com/AUTOMATIC1111/stable-diffusion-webui/issues/12921#top) and we have two different approaches to fix this.

1. Use these extra command line arguments:

	`set COMMANDLINE_ARGS=--no-half-vae --disable-nan-check --opt-split-attention --medvram`
	
	- while running the webui.sh 
	eg. 
	```bash
	./webui.sh --api --no-half-vae --disable-nan-check --xformers --opt-split-attention --medvram
	```
	
	- or if you used docker-compose inside the yaml you can create a new profile 
	
	```yaml
	  auto-xaris: &automatic
    <<: *base_service
    profiles: ["auto-xaris"]
    build: ./services/AUTOMATIC1111
    image: sd-auto:76
    environment:
      - CLI_ARGS=--allow-code --medvram --xformers --enable-insecure-extension-access 
                 --api --disable-nan-check  --no-half-vae --opt-split-attention
	```

	and use this when building the container
	
	```bash
	docker compose --profile auto-xaris up --build
	```

	Although disabling nan-check is not a good idea, because you should get blank outputs instead of getting error messages about NaNs, we still can use it until a proper fix comes out.

2.  Use cpu instead of gpu

	```bash
	docker compose --profile auto-cpu up --build
	```

 	But keep in mind that this will be much slower almost class size.

### CUDA out of memory

When using nvidia GPU and trying to create images with pretty large resolution you need to be sure to use the following command line arguments, the same way described in NansException.

`--medvram --xformers`

If you get an out of memory error, change the command line arguments as described in [AUTOMATIC1111/stable-diffusion-webui/wiki/Troubleshooting#low-vram-video-cards](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Troubleshooting#low-vram-video-cards).

If you want to manually manage the memory you can set the PYTORCH_CUDA_ALLOC_CONF environment variable. Check the answer from this redditor [Memory Allocation for CUDA](https://www.reddit.com/r/StableDiffusion/comments/yck182/comment/jj96bgy/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button)

Additionally for older gpu's you can use the `--lowvram --always-batch-cond-uncond` command line arguments.

for example 

```yaml
export 'PYTORCH_CUDA_ALLOC_CONF=garbage_collection_threshold:0.6,max_split_size_mb:512'
CLI_ARGS=--allow-code --enable-insecure-extension-access --api --disable-nan-check  
		 --precision full --no-half --lowvram --always-batch-cond-uncond --xformers
```


### References

- [AUTOMATIC1111](https://github.com/AUTOMATIC1111/stable-diffusion-webui)
- [stable-diffusion-webui-docker](https://github.com/AbdBarho/stable-diffusion-webui-docker/wiki/Setup)
- [AUTOMATIC1111/stable-diffusion-webui/wiki/Troubleshooting](https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki/Troubleshooting)
