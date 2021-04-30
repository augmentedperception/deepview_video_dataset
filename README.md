# Immersive Light Field Video with a Layered Mesh Representation

This archive contains the raw multi-camera light field video recordings used in our paper,

> *Michael Broxton, John Flynn, Ryan Overbeck, Daniel Erickson, Peter Hedman, Matthew DuVall, Jason Dourgarian, Jay Busch, Matt Whalen, and Paul Debevec.* [Immersive Light Field Video with a Layered Mesh Representation](https://augmentedperception.github.io/deepviewvideo/). *ACM Transactions on Graphics, 39(4), 2020.*

For more information about our paper, please visit our 2020 SIGGARPH technical paper website linked to above. This data was captured using our 46 camera hemispherical light field video rig, which is described in detail in our 2019 SIGGRAPH poster, [A Low Cost Multi-Camera Array
for Panoramic Light Field Video Capture](https://augmentedperception.github.io/lowcost-panoramic-LFV/).

There are 15 scenes, each corresponding to one dataset shown on the website. Each scene is stored a separate zip file containing a collection of 46 videos,although some scenes have a small number of missing cameras. The frame ranges in these videos match those used to produce our web demos. You can download the scenes here:

 * [01_Welder](https://storage.googleapis.com/deepview_video_raw_data/01_Welder.zip)
 * [02_Flames](https://storage.googleapis.com/deepview_video_raw_data/02_Flames.zip)
 * [03_Dog](https://storage.googleapis.com/deepview_video_raw_data/03_Dog.zip)
 * [04_Truck](https://storage.googleapis.com/deepview_video_raw_data/04_Truck.zip)
 * [05_Horse](https://storage.googleapis.com/deepview_video_raw_data/05_Horse.zip)
 * [06_Goats](https://storage.googleapis.com/deepview_video_raw_data/06_Goats.zip)
 * [07_Car](https://storage.googleapis.com/deepview_video_raw_data/07_Car.zip)
 * [08_Pond](https://storage.googleapis.com/deepview_video_raw_data/08_Pond.zip)
 * [09_Alexa_Meade_Exhibit](https://storage.googleapis.com/deepview_video_raw_data/09_Alexa_Meade_Exhibit.zip)
 * [10_Alexa_Meade_Face_Paint_1](https://storage.googleapis.com/deepview_video_raw_data/10_Alexa_Meade_Face_Paint_1.zip)
 * [11_Alexa_Meade_Face_Paint_2](https://storage.googleapis.com/deepview_video_raw_data/11_Alexa_Meade_Face_Paint_2.zip)
 * [12_Cave](https://storage.googleapis.com/deepview_video_raw_data/12_Cave.zip)
 * [13_Birds](https://storage.googleapis.com/deepview_video_raw_data/13_Birds.zip)
 * [14_Puppy](https://storage.googleapis.com/deepview_video_raw_data/14_Puppy.zip)
 * [15_Branches](https://storage.googleapis.com/deepview_video_raw_data/15_Branches.zip)


To make this data easier to work with, each archive also contains a `models.json` file with camera calibration information that was produced using our structure from motion solver. Camera parameters are stored in the following format:

```python
view = {'name': 'camera_0001',
 'position': [0.003311238794086777, 0.000126384684934784, 0.42421246053630285],
 'orientation': [-0.03295944563072383,
  0.02964606619481096,
  -0.03337751007195316],
 'focal_length': 1111.3638715594666,
 'pixel_aspect_ratio': 1.0,
 'principal_point': [1284.7296468363463, 925.4651609728676],
 'height': 1920.0,
 'width': 2560.0,
 'radial_distortion': [0.0970525284520715, -0.01708587111009977, 0.0],
 'projection_type': 'fisheye'}
 ```

The videos were captured using Yi4k action sports cameras with fisheye lenses, therefore it is necessary to include fisheye and radial undistortion when modeling these cameras. This example code shows how to correctly project a world point to an image plane pixel location.

```python
from scipy.spatial.transform import Rotation

def fisheye_to_perspective(point, radial_distortion):
  """Remove fisheye and radial distortion, projecting to a perspective camera."""
  r = np.sqrt(point[0] * point[0] + point[1] * point[1])
  theta = np.arctan2(r, point[2])
  
  r2 = theta * theta
  distortion = 1.0 + r2 * (radial_distortion[0] + r2 * radial_distortion[1])

  return np.array(
      [theta / r * point[0] * distortion, 
       theta / r * point[1] * distortion, 
       1.0])

# Extract camera parameters from the JSON dictionary.
R = Rotation.from_rotvec(view['orientation']).as_dcm()
t = np.array(view['position'])[:, np.newaxis]
extrinsics = np.concatenate((R, -np.dot(R, t)), axis=1)
intrinsics = np.array([[view['focal_length'], 0.0, view['principal_point'][0]],
                       [0.0, view['focal_length'], view['principal_point'][1]],
                       [0.0, 0.0, 1.0]])
radial_distortion = np.array(view['radial_distortion'])

# Project a point in world space into a camera image.
world_point_to_project = np.array([0.5, 0.5, 10.0, 1.0])
local_point = np.dot(extrinsics, world_point_to_project)
undistorted_point = fisheye_to_perspective(local_point, radial_distortion)
px = np.dot(intrinsics, undistorted_point)
# Returns: [1377.85525, 1017.61440, 1.0]
```
