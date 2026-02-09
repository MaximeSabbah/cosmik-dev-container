# cosmik-dev-container

Devcontainer setup for using with mmpose and ros noetic. (linked rt-cosmik : https://github.com/MaximeSabbah/RT-COSMIK.git)

Befaure launching anything open the ports : `xhost +local:`

Commands to run to launch the devcontainer : 
in .devcontainer do : `docker build -t mmpose_image .`
and `docker images` to get the IMAGE_ID
Then in another shell do : 
- AT LAAS : `docker run -it --net=host --gpus all --privileged --device-cgroup-rule "c 81:* rmw" --device-cgroup-rule "c 189:* rmw" -e DISPLAY=$DISPLAY -v /dev:/dev -v /tmp/.X11-unix:/tmp/.X11-unix IMAGE_ID`
-AT SSI NUS (avoids gazebo lag and crash): `docker run -it --net=host  --gpus all --ulimit nofile=1024:524288 --privileged --device-cgroup-rule "c 81:* rmw" --device-cgroup-rule "c 189:* rmw" -e DISPLAY=$DISPLAY -v /dev:/dev -v /tmp/.X11-unix:/tmp/.X11-unix IMAGE_ID`

# To get started with models 
more to come soon...
