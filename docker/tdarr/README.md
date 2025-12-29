# Custom Tdarr Docker Image

This image uses the official Tdarr container as a base and adds:
- AMD/Intel/NVIDIA hardware support (VAAPI, etc)
- HandBrake, Jellyfin FFmpeg, CCExtractor
- .NET 9 and PgsToSrt
- Custom HW transcoding init script

## Build

    docker build -t my-tdarr-custom -f docker/tdarr/Dockerfile .

## Usage

Run as you would the official Tdarr image, with your desired environment variables and volume mounts.

## GitHub Actions Workflow

This repo includes a workflow to build and push the custom Tdarr image to Docker Hub automatically.

### Usage
- Push changes to the `main` branch, or run the workflow manually from the Actions tab.
- Set the following repository secrets:
  - `DOCKERHUB_USERNAME`: Your Docker Hub username
  - `DOCKERHUB_TOKEN`: A Docker Hub access token with push permissions

The image will be built and pushed as `DOCKERHUB_USERNAME/tdarr:latest` for both amd64 and arm64 platforms.

## Customizations
- See Dockerfile for all custom layers and logic.
- Custom scripts are in `docker/root/etc/cont-init.d/`.
