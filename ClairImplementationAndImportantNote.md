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

### Finding Texture Files from the `.world` File
To determine which textures are being used in the simulation, start by locating the `.world` file. These files define the world environment, including texture paths.

1. Locate the `.world` file:
   ```sh
   find /opt/simapp/ -name "*.world"
   ```
2. Open the `.world` file corresponding to your track (e.g., `China_track.world`):
   ```sh
   cat /opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/worlds/China_track.world
   ```
3. Look for lines referencing mesh or texture files:
   ```xml
   <uri>model://China_track/meshes/China_track.obj</uri>
   <texture>meshes/China_track/textures/China_track_road_T_01.png</texture>
   ```
4. From these references, you can determine where the texture files are stored. Normally, textures are located in:
   ```
   /opt/simapp/deepracer_simulation_environment/share/deepracer_simulation_environment/meshes/**your-world-name**/textures/
   ```
5. In my case, using **China_track**, the full texture path is:
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
