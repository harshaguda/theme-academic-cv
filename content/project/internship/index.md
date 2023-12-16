---
title: Learning to Manipulate Deformable Objects - Plastic Bags
summary: An example of using the in-built project page.
tags:
  - Reinforcement Learning
  - Robotics
date: '2023-09-01T00:00:00Z'

# Optional external URL for project (replaces project detail page).
external_link: ''

url_code: ''
url_pdf: ''
url_slides: ''
url_video: ''

# Slides (optional).
#   Associate this project with Markdown slides.
#   Simply enter your slide deck's filename without extension.
#   E.g. `slides = "example-slides"` references `content/slides/example-slides.md`.
#   Otherwise, set `slides = ""`.
slides: uploads/intership_ppt_handout.pdf
---

# Introduction

- Learning to manipulate a deformable bag using a manipulator.
- Build an environment using Isaac Sim physics engine. 
- Learn to lift and open the bag using a visual policy.



# Bag

- Isaac Sim from Nvidia is the physics engine used to simulate the dynamics of a bag.
- The Bag in the simulator is modified in a way to have a different colored rim.
- Properties of the simulation bag are that of a cloth to achieve a close-to-realistic simulation of the bag.
	
%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

# Environment

- Environment is created using Isaac Orbit\footnote{Mittal, Mayank, et al. "Orbit: A unified simulation framework for interactive robot learning environments." IEEE Robotics and Automation Letters (2023).} with a manipulator and a bag.
- Various bag properties can be tweaked, and multiple manipulators can be added.
- Multiple environments can be trained in parallel.



# Experiments: BagLift

- A visual policy\footnote{Wu, Yilin, et al. "Learning to manipulate deformable objects without demonstrations." arXiv preprint arXiv:1910.13439 (2019).} is trained to get the pick point of the bag to lift the bag above the ground.
- A positive reward is given when the manipulator successfully lifts the bag and a negative reward for an unsuccessful attempt.
- UNet is pre-trained to get the segmentation of the bag. 
# Results: BagLift

# Experiments: BagOpening

- A visual policy\footnote{Blanco-Mulero, David, et al. "QDP: Learning to Sequentially Optimise Quasi-Static and Dynamic Manipulation Primitives for Robotic Cloth Manipulation." arXiv preprint arXiv:2303.13320 (2023)} is trained to get the pick point and a place point to open the bag to maximum opening.
- A pick position is selected from the network, based on the pick position a place position is selected, and a drag from pick $\rightarrow$ place is performed.
- Reward is given when the area of the bag rim is increased from the previous configuration after performing an action. 

# Results: BagOpening

- A pick point is predicted from the UNet to give a pick point, using the pick point and crop-centering around the point a place point is predicted.
# Results: BagOpening
- The visual policy is able to learn the pick and place positions as the simulation environment is perfect in a sense (the bag and the background can be easily distinguished)

# Shortcomings
-Manipulator approaches pick/place point in only one orientation.
- The Inverse Kinematics controller used is not able to solve in some instances.
- Bag slips out of manipulator's grip.
- The task at hand is to increase the opening area of the rim at each step, the maximal opening goal set is not reached.
# Miscellaneous

- A docker image with Orbit support is built and hosted on Dockerhub.\footnote{DockerHub \href{https://hub.docker.com/layers/harshavguda/orbit-isaacsim/2022.2.1/images/sha256-4ea1a60c11b7b82c510a892441cb16d66c9a95874ef54af6ad9287ca9bb56632?context=repo}{link}}
- Isaac Orbit Environments are set to run on Triton HPC using the docker image from above.\footnote{More details can be found at \href{https://harshaguda.notion.site/Isaac-Orbit-on-Triton-60762305bbc244eba68d07bed0c715f6?pvs=25}{documentation.}}
- Synthetic camera data generation from the simulator environment.
# Future Work

- Use more manipulation primitives for the BagOpening task.
- Compare with already existing techniques and benchmarks.
- Realisation on the Franka.


