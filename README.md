# AI Sample Face Detection

![](./resource/face_detection_cam.gif)

## 👋 Overview

The ` sample_face_detection` is a Python-based face detection ROS node that uses QNN for model inference.
The models are sourced from Qualcomm AI-hub.It can detect face and locate facial features from face image
It captures the `face_image.jpg` as input and publishes the result to the `/mediaface_det_image` topic.

For model information, please refer to [MediaPipe-Face-Detection - Qualcomm AI Hub](https://aihub.qualcomm.com/iot/models/mediapipe_face?searchTerm=Media)

## Pipeline Flow For Face Detection

![](./resource/sample_face_detection_pipeline.jpg)

| ROS Node         | Description                                                  |
| ---------------- | ------------------------------------------------------------ |
| qrb_ros_face_detector | qrb_ros_face_detector is a Python-based ros jazzy packages that processes face images. This ROS node subscribes to an image topic, and publishes face image result topic after pre/post processing. ` |
| qrb_ros_nn_inference | qrb_ros_nn_inference is a ROS2 package for performing neural network model, providing AI-based perception for robotics applications. |
| qrb ros camera | Qualcomm ROS 2 package that captures images with parameters and publishes them to ROS topics. |
| image_publisher_node | image_publisher is  a ros jazzy packages, can publish image ros topic with local path. |


## 🔎 Table of contents

  * [Used ROS Topics](#-used-ros-topics)
  * [Supported targets](#-supported-targets)
  * [Usage](#-usage)
  * [Build from source](#-build-from-source)
  * [Contributing](#-contributing)
  * [Contributors](#%EF%B8%8F-contributors)
  * [FAQs](#-faqs)
  * [License](#-license)

## ⚓ Used ROS Topics 

| ROS Topic | Type                         | Published By     |
| --------- | ---------------------------- | ---------------- |
| `/mediaface_det_image`  | `< sensor_msgs.msg.Image > ` | `qrb_ros_face_detector` |
| `/image_raw`                   | `<sensor_msgs.msg.Image> `  | `image_publisher_node, camera_node` |
| `/face_detector_input_tensor ` | `<qrb_ros_tensor_list_msgs.msg.TensorList> ` | `qrb_ros_face_detector`     |
| `/face_detector_output_tensor ` | `<qrb_ros_tensor_list_msgs.msg.TensorList> ` | `qrb_ros_nn_inference`     |
| `/face_landmark_input_tensor ` | `<qrb_ros_tensor_list_msgs.msg.TensorList> ` | `qrb_ros_face_detector`     |
| `/face_landmark_output_tensor ` | `<qrb_ros_tensor_list_msgs.msg.TensorList> ` | `qrb_ros_nn_inference`     |

## 🎯 Supported targets

<table >
  <tr>
    <th>Development Hardware</th>
     <td>Qualcomm Dragonwing™ IQ-9075 EVK</td>
  </tr>
  <tr>
    <th>Hardware Overview</th>
    <th><a href="https://www.qualcomm.com/products/internet-of-things/industrial-processors/iq9-series/iq-9075"><img src="https://s7d1.scene7.com/is/image/dmqualcommprod/dragonwing-IQ-9075-EVK?$QC_Responsive$&fmt=png-alpha" width="160"></a></th>
  </tr>
  <tr>
    <th>GMSL Camera Support</th>
    <td>LI-VENUS-OX03F10-OAX40-GM2A-118H(YUV)</td>
  </tr>
</table>

## 🚀 Usage

  <summary>Usage details</summary>

```bash
# Export the NN inference required variables.
export ADSP_LIBRARY_PATH="/usr/lib/rfsa/adsp;/usr/lib/rfsa/adsp/hexagon-v81"
export CDSP_LIBRARY_PATH="/vendor/dsp/cdsp0;/usr/lib/rfsa/adsp/hexagon-v81"

# setup ros environment
source /opt/ros/jazzy/setup.bash

# You can use defalut face image file
ros2 launch sample_face_detection launch_with_image_publisher.py model_path:=/opt/model/
# You can also replace this with a custom image file
ros2 launch sample_face_detection launch_with_image_publisher.py image_path:=/opt/resource/xxx.jpg model_path:=/opt/model/
or # You can launch with qrb_ros_camera lacunch file
ros2 launch sample_face_detection launch_with_qrb_ros_camera_iq10.py  model_path:=/opt/model/
```

When using this launch script, it will use the default parameters:

```py
DeclareLaunchArgument(
'image_path',
default_value=os.path.join(package_path, 'face_image.jpg'),
description='Path to the image file'
)

# Node for image_publisher
image_publisher_node = Node(
package='image_publisher',  
executable='image_publisher_node', 
namespace=namespace,
name='image_publisher_node', 
output='screen', 
parameters=[
{'filename': image_path},  
{'rate': 10.0},  # Set the publishing rate to 10 Hz
]
)
```

It will send local glasses.jpg file, and outputs image at `10` Hz. 

The output key logs:

```
[image_publisher_node-1] [INFO] [1787105463.881644663] [image_publisher_node]: File name for publishing image is: /opt/ros/jazzy/share/sample_face_detection/face_image.jpg
[component_container-2] [QRB INFO] Loading model from binary file: /opt/model/MediaPipeFaceDetector.bin
[component_container-2] [QRB INFO] /usr/lib/libQnnHtp.so initialize successfully
[component_container-2] [QRB INFO] Qnn device initialize successfully
[component_container-2] [INFO] [1787105464.311213459] [nn_inference_node_face_detector]: Inference init successfully!
[component_container-2] [QRB INFO] Loading model from binary file: /opt/model/MediaPipeFaceLandmarkDetector.bin
[component_container-2] [INFO] [1787105464.332459151] [nn_inference_node_face_landmark]: Inference init successfully!
[qrb_ros_face_detector-3] [INFO] [1787105464.350295832] [mediaface_det_node]: init done~
[component_container-2] [INFO] [1787105464.502121501] [nn_inference_node_face_detector]: Got model input data, start executing inference...
[component_container-2] [INFO] [1787105464.505947393] [nn_inference_node_face_detector]: Inference execute successfully!
[component_container-2] [INFO] [1787105464.505984477] [nn_inference_node_face_detector]: Publish the inference result...

```
Verify Results Then you can check the /mediaface_det_image ROS topic

```
# Open a new terminal
source /opt/ros/jazzy/setup.bash

ros2 topic echo /mediaface_det_image

example output:
    ---
    header:
      stamp:
        sec: 0
        nanosec: 0
      frame_id: ''
    height: 355
    width: 325
    encoding: bgr8
    is_bigendian: 0
    step: 975
    data:
    - 255
    ...

```

The IQ10 device doesn't support a desktop environment, so view the face detection result with rqt from a separate Ubuntu host machine that has desktop support and is on the same ROS network.

```
# Open a new terminal
source /opt/ros/jazzy/setup.bash
rqt

```
Select menu selection: `Plugins --> Visualization --> Image View`, then select the /mediaface_det_image topic to view the detected faces and facial landmarks overlaid on the input image.


## 🤝 Contributing

We love community contributions! Get started by reading our [CONTRIBUTING.md](CONTRIBUTING.md).

Feel free to create an issue for bug report, feature requests or any discussion💡.

## ❤️ Contributors

Thanks to all our contributors who have helped make this project better!

<table>
  <tr>
    <td align="center"><a href="https://github.com/chuhxie"><img src="https://avatars.githubusercontent.com/u/205736874?v=4" size="32"  width="32" height="32" alt="chuhxie"/><br /><sub><b>chuhxie</b></sub></a></td>
  </tr>
</table>


## ❔ FAQs

<summary>Can detect multiple face?</summary><br>
No,it can only support single face detection.


## 📜 License

Project is licensed under the [BSD-3-Clause](https://spdx.org/licenses/BSD-3-Clause.html) License. See [LICENSE](./LICENSE) for the full license text.
