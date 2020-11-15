# Tiny YOLOv4 TensorFlow Lite model on Jetson Xavier

This sample is an example of running an AI container on the Jetson platform.  This container utilizes the GPU on the Jetson (with NVIDIA drivers, CUDA and cuDNN installed) using an NVIDIA L4T (linux for Tegra) base image with TensorFlow 2 installed.  The Jetson must have been flashed with Jetpack 4.4.

## Xavier Setup and requirements

- Flashed with JetPack 4.4 (L4T R32.4.3) with all ML and CV tools (including `nvidia-docker`)
- Samsung NVMe 512 GB to store docker images
- 16 GB swap file on NVMe mount
- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt#manual-install-instructions) for pushing image to Azure Container Registry
- [Optional] Docker may be configured to run with non-root user as in [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user) allowing the omission of using `sudo` with docker

## Build image

The following instructions will enable you to build a docker container with a [YOLOv4 (tiny)](https://github.com//AlexeyAB/darknet) [TensorFlow Lite](https://www.tensorflow.org/lite) model using [nginx](https://www.nginx.com/), [gunicorn](https://gunicorn.org/), [flask](https://github.com/pallets/flask), and [runit](http://smarden.org/runit/).  The app code is based on the [tensorflow-yolov4-tflite](https://github.com/hunglc007/tensorflow-yolov4-tflite) project.  This project uses TensorFlow v2.

Note: References to third-party software in this repo are for informational and convenience purposes only. Microsoft does not endorse nor provide rights for the third-party software. For more information on third-party software please see the links provided above.

### Prerequisites for building image

1. [Ensure NVIDIA Docker](https://github.com/NVIDIA/nvidia-docker/wiki/NVIDIA-Container-Runtime-on-Jetson) on your Jetson
2. [Install curl](http://curl.haxx.se/)
3. [Install Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli-apt) to be able to push image to Azure Container Registry (ACR)

### Preparing for using Blob Storage on the Edge

1. Create a file in the `app/` folder called `.env` with the following contents:

```
LOCAL_STORAGE_ACCOUNT_NAME=<Name for local blob storage in IoT edge Blob Storage module>
LOCAL_STORAGE_ACCOUNT_KEY=<Key generated for local IoT edge Blob Storage module in double quotes>
```

### Building the docker container

1. Create a new directory on your machine and copy all the files (including the sub-folders) from this GitHub folder to that directory.
2. Build the container image (will take several minutes) by running the following docker command from a terminal window in that directory.

```bash
sudo nvidia-docker build . -t tiny-yolov4-tflite:arm64v8-cuda-cudnn -f arm64v8-gpu-cudnn.dockerfile
```

### Upload docker image to Azure Container Registry

Log in to ACR with the Azure CLI (also may use docker login):

```
az acr login --name <name of your ACR user>
```

Push the image to ACR:

```
docker push <your ACR URL>/tiny-yolov4-tflite:arm64v8-cuda-cudnn
```

Note:
- More instruction at [Push and Pull Docker images - Azure Container Registry](http://docs.microsoft.com/en-us/azure/container-registry/container-registry-get-started-docker-cli) to save your image for later use on another machine.
- IMPORTANT:  Docker may need to be configured to run with non-root user as in [Manage Docker as a non-root user](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user).

    
## Deploy as an edge module for Live Video Analytics

1. Deployment manifest
2. Console app

When the Live Video Analytics on Edge direct methods are invoked on device, images will appear in a folder with the name of your local container e.g. `/media/nvme/blob_storage/BlockBlob/annotatedimageslocal` and with default deployment manifest, will stick around on device for 60 minutes before being deteted after upload to the cloud Blob Storage container (by default called `annotated-images-xavier-yolo4`).

## Troubleshooting

### Troubleshooting a running container

To troubleshoot a running container you may enter it with ssh by using the following command.

```
sudo docker exec -it my_yolo_container /bin/bash
```

For IoT Edge troubleshooting see [Troubleshoot your IoT Edge device](https://docs.microsoft.com/en-us/azure/iot-edge/troubleshoot).

### Azure Media Services

1.  If AMS account has changed, then on device delete and recreate the App Data Directory for AMS:
```
sudo rm -fr /var/lib/azuremediaservices
mdkir -p /var/lib/azuremediaservices
```
   - It is a good idea to then restart lvaEdge module
   ```
   iotedge restart lvaEdge
   ```

### Azure Blob Storage IoT Edge module

1. Check the logs for Permission denied errors for folder on device used as container.
```
iotedge logs <name of your edge container e.g. azureblobstorageoniotedge>
```
   - If there is a "Permission denied" error try changing the owner and group on the folder (see [Granting directory access to container user on Linux](https://docs.microsoft.com/en-us/azure/iot-edge/how-to-store-data-blob?view=iotedge-2018-06#granting-directory-access-to-container-user-on-linux)).
   ```
   sudo chown -R 11000:11000 <local blob directory e.g. /media/nvme/blob_storage>
   ```
   - It is a good idea to then restart lvaEdge module
   ```
   iotedge restart lvaEdge
   ```


## Helpful links

- [`darknet` implementation for YOLOv4](https://github.com/AlexeyAB/darknet)
- [TensorFlow YOLOv4 converters and implementations](https://github.com/hunglc007/tensorflow-yolov4-tflite)
