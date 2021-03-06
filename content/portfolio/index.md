+++
slug = ""
date = "2016-10-22T15:52:29+11:00"
title = "Portfolio"
draft = false
comments = false
categories = ["portfolio"]
showpagemeta = false
+++

## Papers

Author ORCID: {{< orcid-link >}}

### 3D human pose estimation with 2D marginal heatmaps

[Publisher link](https://ieeexplore.ieee.org/document/8658906)
|
[Full text](https://arxiv.org/pdf/1806.01484.pdf)

A system for estimating 3D human joint locations from monocular
images.

```bibtex
@inproceedings{nibali2019margipose,
  title={3D Human Pose Estimation with 2D Marginal Heatmaps},
  author={Nibali, Aiden and He, Zhen and Morgan, Stuart and Prendergast, Luke},
  booktitle={2019 IEEE Winter Conference on Applications of Computer Vision (WACV)},
  pages={1477-1485},
  year={2019},
  doi={10.1109/WACV.2019.00162}
}
```

### Numerical coordinate regression with convolutional neural networks

[Full text](https://arxiv.org/pdf/1801.07372.pdf)

A comparison of approaches to predicting keypoint locations from images using
convolutional neural networks.

```bibtex
@article{nibali2018numerical,
  title={Numerical Coordinate Regression with Convolutional Neural Networks},
  author={Nibali, Aiden and He, Zhen and Morgan, Stuart and Prendergast, Luke},
  journal={arXiv preprint arXiv:1801.07372},
  year={2018}
}
```

### Extraction and classification of diving clips from continuous video footage

[Publisher link](http://openaccess.thecvf.com/content_cvpr_2017_workshops/w2/html/Nibali_Extraction_and_Classification_CVPR_2017_paper.html)
|
[Full text](https://arxiv.org/pdf/1705.09003.pdf)

A video analysis system for athletic diving with large deep learning components.
The model is able to extract action clips, track athletes, and classify the
type of dive performed.

<i class="fa fa-trophy" aria-hidden="true"></i> [Best paper award](https://web.archive.org/web/20171101172550/http://www.vap.aau.dk/cvsports/).

```bibtex
@inproceedings{nibali2017extraction,
  author = {Nibali, Aiden and He, Zhen and Morgan, Stuart and Greenwood, Daniel},
  title = {Extraction and Classification of Diving Clips From Continuous Video Footage},
  booktitle = {The IEEE Conference on Computer Vision and Pattern Recognition (CVPR) Workshops},
  pages={38--48},
  month = {July},
  year = {2017}
}
```

### Pulmonary nodule classification with deep residual networks

[Publisher link](https://link.springer.com/article/10.1007%2Fs11548-017-1605-6)
|
[Full text](http://homepage.cs.latrobe.edu.au/zhe/files/Pulmonary_nodule.pdf)

Application of a deep learning classifier to subjective malignancy prediction
of lung cancer nodules. The model is able to approximate the predictive power
of a human radiologist ensemble.

```bibtex
@article{nibali2017pulmonary,
  title={Pulmonary nodule classification with deep residual networks},
  author={Nibali, Aiden and He, Zhen and Wollersheim, Dennis},
  journal={International Journal of Computer Assisted Radiology and Surgery},
  pages={1--10},
  year={2017},
  publisher={Springer International Publishing}
}
```

### Trajic: An effective compression system for trajectory data

[Publisher link](http://ieeexplore.ieee.org/document/7112156/)
|
[Full text](http://homepage.cs.latrobe.edu.au/zhe/files/trajic.pdf)

A compression algorithm designed for GPS trajectory data. The algorithm is able
operate in lossy and lossless modes.

```bibtex
@article{nibali2015trajic,
  title={Trajic: An effective compression system for trajectory data},
  author={Nibali, Aiden and He, Zhen},
  journal={IEEE Transactions on Knowledge and Data Engineering},
  volume={27},
  number={11},
  pages={3138--3151},
  year={2015},
  publisher={IEEE}
}
```

## Projects

### RatTrace

RatTrace is the intelligent rat trap monitoring system which I developed for my
final year undergraduate engineering project. It works by measuring rat bait
levels and wirelessly reporting the results so that pest control experts don't
need to physically check each trap on location.

<div style="float: left; display: block; width: 40%;">
  <a href="/img/rattrace_in_trap.jpg">
    <img style="height: 270px" src="/img/rattrace_in_trap.jpg">
  </a>
</div>
<div class="tablet-frame" style="width: 50%; float: right;">
  <a href="/img/rattrace_web_interface.png">
    <img src="/img/rattrace_web_interface.png">
  </a>
</div>
<div style="clear:both;"></div>

***

### 1000WAT website

[1000WAT](http://1000wat.com.au) is a Thai food restaurant based in Melbourne's
CBD. I created a new website for 1000WAT with a focus on responsive design and
simple content management. Ruby on Rails was used on the server-side, with
jQuery and Twitter Bootstrap providing a good client-side user experience.

<div class="mobile-frame" style="width: 20%; float:left;">
  <a href="/img/1000wat_mobile.jpg">
    <img src="/img/1000wat_mobile.jpg">
  </a>
</div>
<div class="tablet-frame" style="width: 60%; float: right;">
  <a href="/img/1000wat_tablet.jpg">
    <img src="/img/1000wat_tablet.jpg">
  </a>
</div>
<div style="clear:both;"></div>

***

### Self-balancing robot

This robot is able to balance itself on two wheels. The project involved
laying out a PCB using Altium Designer, soldering the manufactured circuit board,
and writing the embedded C code required to make it balance.

<div style="text-align: center">
  <a href="/img/balancing_robot.jpg">
    <img alt="Self-balancing robot" src="/img/balancing_robot.jpg" style="width: 50%;">
  </a>
</div>

***

### Seed-dropping robot

The seed-dropping robot was created as an entrant for the 2014
[National Instruments Autonomous Robotics Commpetition](http://australia.ni.com/ni-arc).
This robot was able to navigate an obstacle course unassisted, using feedback
from a LiDAR and wheel encoders to determine an appropriate course.

This project was developed by a team of students from La Trobe University, and
was a finalist in the 2014 NI ARC competition.

<div style="text-align: center">
  <a href="/img/seed_dropping_robot.jpg">
    <img alt="Self-balancing robot" src="/img/seed_dropping_robot.jpg" style="width: 40%;">
  </a>
</div>

***

### Home Guardian

Home Guardian is a home monitoring system designed for people who live alone and
are at an increased risk of falling victim to an injury or illness in and around
the home that would otherwise affect their ability to seek help on their own.

This project was developed by a team of students from La Trobe University,
and won 1st place in Telstra's inaugral M2M University Challenge in 2013.

<div style="text-align: center">
  <a href="/img/home_guardian.jpg">
    <img alt="Home Guardian device" src="/img/home_guardian.jpg" style="width: 50%;">
  </a>
</div>

***

### MediCom

MediCom is a real-time communication system designed for use in the medical
sector. It makes use of WebRTC for video conferencing capabilities and runs on
both mobile devices and PCs.

This project was developed by a team of students from La Trobe University
according to a brief from Ericsson Australia.

<div style="text-align: center">
  <a href="/img/medicom.jpg">
    <img alt="MediCom app" src="/img/medicom.jpg" style="width: 50%;">
  </a>
</div>
