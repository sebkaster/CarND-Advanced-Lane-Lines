## Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)
![Lanes Image](./examples/example_output.jpg)

<!-- TABLE OF CONTENTS -->
## Table of Contents

* [About the Project](#about-the-project)
* [Getting Started](#getting-started)
  * [Prerequisites](#prerequisites)
  * [Installation](#installation)
* [Usage](#usage)
* [Contributing](#contributing)
* [License](#license)
* [Contact](#contact)


## About the Project
---

When we drive, we use our eyes to decide where to go. The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle.  Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

In this project lane lines in images are detected using Python and OpenCV. OpenCV means "Open-Source Computer Vision", which is a package that has many useful tools for analyzing images.

<img src="examples/laneLines_thirdPass.jpg" width="480" alt="Combined Image" />

The steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

<!-- GETTING STARTED -->
## Getting Started

The software is written in Python 3.7 and tested on Linux. The usage of the Miniconda Python distribution is strongly recommended.

### Prerequisites

* Miniconda (https://docs.conda.io/en/latest/miniconda.html)

### Installation

1. Clone this repo
```sh
git clone https://github.com/sebkaster/CarND-Advanced-Lane-Lines.git
```

2. Create Anaconda environemnt
```sh
conda create -n lane_lines_env anaconda python=3
```

3. Actiavate environment
```sh
conda activate lane_lines_env
```

4. Install pip package manager
```sh
conda install pip
```

5. Install required python modules
```
python -m pip install -r requirements.txt
```

<!-- USAGE EXAMPLES -->
## Usage

In order to use this project start jupyter by typing `jupyter notebook` in your Anaconda environment. The notebook _P1.ipynb_ shows the implementation of the whole pipeline for single road images and videos. Run the cells to see how this pipeline works. \\

Moreover, you can use the pipeline to process your own images or videos in the notebook as shown below. Tune the configuration parameters to optimize the pipeline on your data samples.

### Single Image

```
img_file_name = "<your_image_file_name>"
img_folder_name = "<your_image_folder_name>"

process_image(img_file_name, img_folder_name)
```

### Videos
```
pv_class = ProcessVideo()

input_video_file_name = "<your_inp_name>"
output_video_file_name = "<your_out_name>"

clip1 = VideoFileClip(input_video_file_name)
white_clip = clip1.fl_image(pv_class.process_image)
white_clip.write_videofile(output_video_file_name, audio=False)
```

<!-- CONTRIBUTING -->
## Contributing

Contributions are what make the open source community such an amazing place to be learn, inspire, and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<!-- LICENSE -->
## License

Distributed under the MIT License.

<!-- CONTACT -->
## Contact

Sebastian Kaster - sebastiankaster@googlemail.com

Project Link: [https://github.com/sebkaster/CarND-Advanced-Lane-Lines](https://github.com/sebkaster/CarND-Advanced-Lane-Lines)


