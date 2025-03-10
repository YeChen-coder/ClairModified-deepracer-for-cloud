# Introduction
This note provides detailed steps for modifying the simulation environment, primarily focusing on textures.
Additionally, an explanation of training-world-related files is included.

# Part 1: Implementation Tips
Note: AWS instances do not actually reboot! Whether you use the `reboot` command or restart the instance from the AWS console, it won’t apply `init.sh` automatically. You must run it manually.

Once you see a "Done" file under `/deepracer-for-cloud`, you're good to go. The system will automatically generate `system.env` and `run.env`, so no need to worry about them.

# Part 2: Modifying the Training Background  
As **Albert Einstein** said, engineers should keep things as simple as possible. 
Instead of modifying the entire race world, I chose to modify only the textures. 
Find the right point, switch it, done.

**Before:**  
![Original car view](/PicturesForMyNote/OriginalCarView.png)  

**After:**  
![Modified whole view](/PicturesForMyNote/ModifiedWholeView.png)  
![Modified car view](/PicturesForMyNote/ModifiedCarView.png)  

**Changed Texture Images:**  
![Original textures list](/PicturesForMyNote/OriginalTexturesList.png)  
![Modified textures list](/PicturesForMyNote/ModifiedTexturesList.png)  

## Steps
### Locate Texture File Paths
The Robomaker code is available at:  
[GitHub: AWS DeepRacer SimApp](https://github.com/aws-deepracer-community/deepracer-simapp)  

Information is wrapped inside, so first, enter the Robomaker container.

1. Find the container ID:
   ```sh
   docker ps
   ```
2. Enter the container:
   ```sh
   docker exec -it <container_id> /bin/bash
   ```
3. Search for `.png` files:
   ```sh
   find / -iname "*China*" 2>/dev/null
   find / -iname "*.png" 2>/dev/null | grep -i "China"
   ```

Normally, textures are located at:
```
/opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/**your-world-name**/textures/
```
In my case, I'm using **China_track**, so my full path is:
```
/opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/China_track/textures/
```
![China_track textures](/PicturesForMyNote/ChinaTrackTextureList.png)

### Replace Original Textures with Custom Ones
Use the simplest painting tool available. The only requirement is that the custom image must have the same **size and name** as the original one!

### Create a Docker Compose File
We use the simplest approach—replacing only specific files while keeping everything else intact.

Create a `docker-compose-mytextures.yml` file under `./Deepracer-for-Cloud/Docker/`:
```sh
nano docker-compose-mytextures.yml
```
Paste the following content:
```yaml
version: '3.7'
services:
  robomaker:
    volumes:
      - '/root/deepracer-for-cloud/textures/China_track_center_yellow_T_01.png:/opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/China_track/textures/China_track_center_yellow_T_01.png'
      - '/root/deepracer-for-cloud/textures/China_track_concrete_T_01.png:/opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/China_track/textures/China_track_concrete_T_01.png'
      - '/root/deepracer-for-cloud/textures/China_track_edge_white_T_01.png:/opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/China_track/textures/China_track_edge_white_T_01.png'
      - '/root/deepracer-for-cloud/textures/China_track_field_grass_T_01.png:/opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/China_track/textures/China_track_field_grass_T_01.png'
      - '/root/deepracer-for-cloud/textures/China_track_road_T_01.png:/opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/China_track/textures/China_track_road_T_01.png'
      - '/root/deepracer-for-cloud/textures/China_track_sea_T_01.png:/opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/China_track/textures/China_track_sea_T_01.png'
      - '/root/deepracer-for-cloud/textures/China_track_side_edge_T_01.png:/opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/China_track/textures/China_track_side_edge_T_01.png'
```
Save and exit:
```sh
Ctrl+S  # Save
Ctrl+X  # Exit
```

### Mount `docker-compose-mytextures.yml`
To understand how this project functions, locate the training script:

Inside `bin/scripts_wrapper.sh`, find the function:
```sh
function dr-start-training {
  dr-update-env
  $DR_DIR/scripts/training/start.sh "$@"
}
```
Since it references `scripts/training/start.sh`, locate this file.

Modify it by adding:
```sh
COMPOSE_FILES="$COMPOSE_FILES $DR_DOCKER_FILE_SEP /root/deepracer-for-cloud/docker/docker-compose-mytextures.yml"
```
Your updated script should look like this:
```sh
if [ ${DR_ROBOMAKER_MOUNT_LOGS,,} = "true" ]; then
  COMPOSE_FILES="$DR_TRAIN_COMPOSE_FILE $DR_DOCKER_FILE_SEP $DR_DIR/docker/docker-compose-mount.yml"
  export DR_MOUNT_DIR="$DR_DIR/data/logs/robomaker/$DR_LOCAL_S3_MODEL_PREFIX"
  mkdir -p $DR_MOUNT_DIR
else
  COMPOSE_FILES="$DR_TRAIN_COMPOSE_FILE"
fi
COMPOSE_FILES="$COMPOSE_FILES $DR_DOCKER_FILE_SEP /root/deepracer-for-cloud/docker/docker-compose-mytextures.yml"
```

## Update and Start Training
```sh
dr-update
dr-start-training
```
View car:
```sh
dr-start-viewer
```
For cloud instances, view the simulation at:
```
http://public-ip-of-instance:8100
```
Note:
- **Must be HTTP!**
- **Ensure inbound rules allow port 8100.**
- **If images don’t load, try incognito mode!**
