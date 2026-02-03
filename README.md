# mmpose-dev-container

Devcontainer setup for using with mmpose and ros noetic. (linked with panda_torque_mpc : https://gitlab.laas.fr/msabbah/panda_torque_mpc and rt-cosmik : https://gitlab.laas.fr/msabbah/rt-cosmik.git )

Befaure launching anything open the ports : `xhost +local:`

Commands to run to launch the devcontainer : 
in .devcontainer do : `docker build -t mmpose_image .`
and `docker images` to get the IMAGE_ID
Then in another shell do : 
- AT LAAS : `docker run -it --net=host --gpus all --privileged --device-cgroup-rule "c 81:* rmw" --device-cgroup-rule "c 189:* rmw" -e DISPLAY=$DISPLAY -v /dev:/dev -v /tmp/.X11-unix:/tmp/.X11-unix IMAGE_ID`
-AT SSI NUS (avoids gazebo lag and crash): `docker run -it --net=host  --gpus all --ulimit nofile=1024:524288 --privileged --device-cgroup-rule "c 81:* rmw" --device-cgroup-rule "c 189:* rmw" -e DISPLAY=$DISPLAY -v /dev:/dev -v /tmp/.X11-unix:/tmp/.X11-unix IMAGE_ID`

# To get started with mmdeploy models 
## FOR GPU USAGE

Go in /root/workspace/mmdeploy
Do : 
```
python3 tools/deploy.py \
configs/mmdet/detection/detection_tensorrt_static-320x320.py \
../mmpose/projects/rtmpose/rtmdet/person/rtmdet_nano_320-8xb32_coco-person.py \
https://download.openmmlab.com/mmpose/v1/projects/rtmpose/rtmdet_nano_8xb32-100e_coco-obj365-person-05d8511e.pth \
demo/resources/human-pose.jpg \
--work-dir rtmpose-trt/rtmdet-nano \
--device cuda:0 \
--show \
--dump-info  # dump sdk info
```

And 

for body 17 : 
```
python3 tools/deploy.py \
configs/mmpose/pose-detection_simcc_tensorrt_dynamic-256x192.py \
../mmpose/projects/rtmpose/rtmpose/body_2d_keypoint/rtmpose-m_8xb256-420e_coco-256x192.py \
https://download.openmmlab.com/mmpose/v1/projects/rtmposev1/rtmpose-m_simcc-aic-coco_pt-aic-coco_420e-256x192-63eb25f7_20230126.pth \
demo/resources/human-pose.jpg \
--work-dir rtmpose-trt/rtmpose-m \
--device cuda:0 \
--show \
--dump-info  # dump sdk info
```

for body 26 : 
```
python3 tools/deploy.py \
configs/mmpose/pose-detection_simcc_tensorrt_dynamic-256x192.py \
../mmpose/projects/rtmpose/rtmpose/body_2d_keypoint/rtmpose-m_8xb512-700e_body8-halpe26-256x192.py \
https://download.openmmlab.com/mmpose/v1/projects/rtmposev1/rtmpose-m_simcc-body7_pt-body7-halpe26_700e-256x192-4d3e73dd_20230605.pth \
demo/resources/human-pose.jpg \
--work-dir rtmpose-trt/rtmpose-m \
--device cuda:0 \
--show \
--dump-info  # dump sdk info
```

in order to create mmpose and mmdet model with tensorrt

## FOR CPU USAGE

Mmdet : 
```
python3 tools/deploy.py \
configs/mmdet/detection/detection_onnxruntime_dynamic.py \
../mmpose/projects/rtmpose/rtmdet/person/rtmdet_nano_320-8xb32_coco-person.py \
https://download.openmmlab.com/mmpose/v1/projects/rtmpose/rtmdet_nano_8xb32-100e_coco-obj365-person-05d8511e.pth \
demo/resources/human-pose.jpg \
--work-dir rtmpose-trt/rtmdet-nano \
--device cpu \
--show \
--dump-info  # dump sdk info
```

MMpose : 
```
python3 tools/deploy.py \
configs/mmpose/pose-detection_simcc_onnxruntime_dynamic.py \
../mmpose/projects/rtmpose/rtmpose/body_2d_keypoint/rtmpose-m_8xb512-700e_body8-halpe26-256x192.py \
https://download.openmmlab.com/mmpose/v1/projects/rtmposev1/rtmpose-m_simcc-body7_pt-body7-halpe26_700e-256x192-4d3e73dd_20230605.pth \
demo/resources/human-pose.jpg \
--work-dir rtmpose-trt/rtmpose-m \
--device cpu \
--show \
--dump-info  # dump sdk info
```

# Ros ws
Source deps_ws and do : `catkin build -j12 panda_torque_mpc`
Then source /ros_ws/devel/setup.bash to use the package

# Then you can normally make mmpose communicate with the ros controller 
