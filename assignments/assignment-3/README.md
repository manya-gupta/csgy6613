--------------DELIVERABLES--------------

Assignment notebook: AIhw3_objTracking.ipynb

Hugging Face Repo: https://huggingface.co/datasets/mg5879/AI-HW3-DroneDetections

Output Video 1: https://youtube.com/shorts/a4if95Fuw_I?feature=share

Output Video 2: https://youtube.com/shorts/ED3xoLuL2-Y?feature=share

The upload quality is very bad on YouTube likely because of the aspect ratio and how I saved the frames,
so I have uploaded the .mp4 files as well

--------------------DATASET---------------------

Dataset chosen: https://www.kaggle.com/datasets/dasmehdixtr/drone-dataset-uav

This dataset was chose due to its compatibility with YOLO and popularity on kagglehub. It has 1359 photos with bounding box labels. These photos are diverse in augmentation and quality. 
Several users have trained YOLO models using this dataset, so I used one of their pre-trained models to extract frames. credit: Leonardo Cofone. His python notebook was very thorough and easy to follow. It resulted in the following stats:

- Precision: 0.980
- Recall: 0.961
- mAP50: 0.990
- mAP50-95: 0.723

This can be seen in Copy_of_Drone_detection.ipynb

---------------KALMAN FILTER DESIGN-----------------

State vector (dim_x=6): [cx, cy, vx, vy, w, h] — center x/y position, x/y velocity, and bounding box width/height. 

Measurements (dim_z=4): The filter observes [cx, cy, w, h] directly from the YOLO detector each frame. Velocity is inferred.

Motion model (F): Assumes constant velocity — position is updated each timestep by adding velocity × dt (0.2s). Width and height are assumed static between frames.

Noise metrics:

- P (initial state covariance): Set to 100 × I, reflecting high initial uncertainty about the state.
- R (measurement noise): Trusts position measurements (cx, cy) more than size measurements (w, h), which are noisier.
- Q (process noise): Adds extra uncertainty to velocity (vx, vy) since acceleration is possible and the constant-velocity assumption isn't perfect.

Initialization: The filter is seeded from the first frame's YOLO detection. If no drone is detected in frame 1, all values default to zero.


-----------FAILURE CASES/MISSING DRONE--------------

My YOLO model only feeds the Kalman filter if it detects a drone with over 40% confidence. If this criteria is not met, the update step is skipped. This allows the filter to coast through frames where the drone is obscured or missed by the detector, using the velocity estimate to maintain a plausible position rather than dropping the track.

I implemented the confidence threshold after working on video 2, as the drone is not only sparse but also very small in that video. The 40% threshold was decided on after testing 50%, which contained many false detections, and 30%, which had too many missed detections. 