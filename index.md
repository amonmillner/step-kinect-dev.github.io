## EASE Lab Explorations with Azure Kinect

Dr. Amon Millner, Hwei-Shin Harriman, Richard Gao of the Olin College of Engineering Extending Access to STEM Empowerment (EASE) Lab are engaging with the Microsoft Azure Kinect Development Kit to leverage its unique features in a context related to step performances.

Through this page, we are sharing early experiments in adding new dimensions to step performances.

Use this map to the page to follow our progress.

### The Inspiration
1. [Seeing a Step Show on stage and on screen](#movie)
2. [Stepping on campus](#usc)
3. [Experiencing steps + Kinect tech + kids](#nmaahc)

### The Vision
4. [Relating to EASE Lab work](#ease)

### The Technical Exploration Sprint
5. [Adding Azure services to Kinect-enhanced steps](#stepkinect)
   * [sensing steps with cameras](#stepsense)
   * [recognizing emotion with azure](#face)
   * [making responsive graphic visualizations](#viz)
   * [generating sounds with steps](#sounds)

### Future Directions
6. [Seeding collaborations](#collabo)



<a name="movie"></a>
## The Inspiration
### Seeing a Step Show on stage and on screen

As a middle schooler, Amon was intrigued by a few aspects of college, the computer science major and step shows. He saw college through the lens of a family friend, who invited him to watch her step show. He experience the electric atmosphere of the synchronized sound in person and later enjoyed how step shows were portrayed as a symbol of simultaneous brotherhood/sisterhood and sibling competition at the same time - such as the following clip from the 1980's "School Daze" film:

<iframe width="640" height="480" src="https://www.youtube.com/watch?v=WfPc8Ia7E20"></iframe>


<a name="usc"></a>

### Stepping on campus

As a computer science major in college, Amon also participated in step shows.

![](/photos/usc1.jpg)

Amon is also old, so we have not yet transferred VHS footage of him stepping into digital images so the placeholder above will have to do.

<a name="nmaahc"></a>

## Experiencing Steps + Kinect tech + kids

Amon has had a close encounter with an interactive stepping experience that made use of older Kinect sensors. See his kids at the National Museum of African American History and Culture following the lead of on-screen step instructors while they see point cloud-esque likenesses of themselves on the screen alongside the virtual performers.

![](/photos/nmaahc0.jpg)

<iframe src="https://drive.google.com/file/d/197w6iOkJ9QfrqRP9aMSpM0E64h61uHtl/view?usp=sharing" width="640" height="480"></iframe>

Observing how engaging the exhibit was made Amon leave wondering what other types of interactions could be supported with technologies such as Kinects.

Amon was excited to get his hands on the Azure Kinect DK, because he could engage students in his Lab in a very ambitious exploration that connects to his heritage. The art of stepping has a rich past with roots in gumboot dancing, shown in this image from the NMAAHC museum. The art also has great potential to evolve in ways that leverage technological advances. This exploration is an early attempt to experiment in the space of stepping.


<a name="ease"></a>

## The Vision

### Relating to EASE Lab work

I'll explain here how this work can exhibit cultures that are needing more representation at the forefront of advanced technologies.

Making interests come to life in new ways through computing.
Our team has experience with culturally-relevant performances and how they intersect with code.

### The Vision in full

People would...
and they would see code

image of the snake face

<a name="stepkinect"></a>

## The Technical Exploration Sprint

### Adding Azure to Kinect-enhanced steps in a sprint

Our first sprint goal was to integrate different aspects of the Azure Kinect's features that have not been available before in previous Kinect models through compelling contexts.

The features of the device can be found at the MS Azure Kinect SDK site.

We use the camera to track the points on a person's leg to determine when a "stomp" is happening or when hands are clapping.

We send full frame images to Cognitive Services face API to receive the likely emotion expressed during the frame.

we made use of Open Frameworks extensively.

Every stomp that is registered triggers a graphical visualization under the particular stepper's foot.

Every clap sensed triggers a sound.

The background of the scene changes to red when the stepper is considered to be showing angered or intense faces. When happiness is detected, the background is light gold.

<a name="stepsense"></a>

#### Sensing steps with cameras

code and video of detecting steps with the movement delta hack.

<a name="face"></a>
#### Recognizing emotion with azure

sending faces to the cloud.

<a name="viz"></a>
#### Making responsive graphic visualizations

Underfoot animations.

<a name="sounds"></a>
#### Generating sounds with steps

Different sounds for different facial expressions.


<a name="collabo"></a>
## Future Directions

### Seeding collaborations

Should this work continue...
A musician could augment these efforts in the following ways...
