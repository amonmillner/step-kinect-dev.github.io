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

<iframe width="640" height="480" src="https://www.youtube.com/embed/WfPc8Ia7E20?start=38" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>">


<a name="usc"></a>

### Stepping on campus

As a computer science major in college, Amon also participated in step shows.

![](/photos/usc1.jpg)

Amon is also old, so we have not yet transferred VHS footage of him stepping into digital images so the placeholder above will have to do.

<a name="nmaahc"></a>

## Experiencing Steps + Kinect tech + kids

Amon has had a close encounter with an interactive stepping experience that made use of older Kinect sensors. See his kids at the National Museum of African American History and Culture following the lead of on-screen step instructors while they see point cloud-esque likenesses of themselves on the screen alongside the virtual performers.

![](/photos/nmaahc0.jpg)

<iframe src="https://drive.google.com/file/d/197w6iOkJ9QfrqRP9aMSpM0E64h61uHtl/preview" width="640" height="480"></iframe>

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

Our first sprint goal was to integrate different aspects of the Azure Kinect's features that have not been available before in previous Kinect models while introducing compelling contexts.

[the MS Azure Kinect SDK site]() shows the full list of the device's features. Our initial efforts explored only a subset: video sensing and Azure Cognitive Services.

We have developed a system to use the Kinect camera to track the points on a person's leg to determine when a "stomp" is happening or when hands are clapping. While people are (hopefully stepping) in front of the camera, we send full frame images to Cognitive Services face API to receive the likely emotion expressed during the frame.

We elected to use Open Frameworks extensively in the first sprint. This gave us the ability to explore making every stomp that is registered trigger a graphical visualization under the stepper's foot. We also play back different sounds for stomps than we do for claps.

We change the background of the scene to red when the Face API informs the program that the stepper is considered to be showing angered face. When happiness is detected, the background is light gold.

<a name="stepsense"></a>

#### Sensing steps with cameras

The ver 0.1 code snippet at the bottom of this subsection gives an example of how we went about detecting when a stepper stomps the ground. The approach is reasonable albeit not super robust at the moment. To get the proof of concept working, we needed to expose timestamp information from the Open Frameworks codebase by adding functions to retrieve the private timestamp variable, so that we could determine DP/DT of two samples - only using the Y. We did not use pure distance formula because we were only concerned with a leg going up or down, which the Kinect maps to the Y coordinate. The program considers a stomp to occur When the current velocity is small, but the preceding velocity was large.



```c++
//TODO: Refactor into MOVE Detection Class
double ofApp::calculateDistance(k4a_float3_t pos1, k4a_float3_t pos2)
{
	auto xDistance = pow(pos2.xyz.x - pos1.xyz.x, 2);
	auto yDistance = pow(pos2.xyz.y - pos1.xyz.y, 2);
	auto zDistance = pow(pos2.xyz.z - pos1.xyz.z, 2);
	return sqrt(xDistance + yDistance + zDistance);
}

double ofApp::calculateYDist(k4a_float3_t pos1, k4a_float3_t pos2)
{
	double dp = pos2.xyz.y - pos1.xyz.y;
	return abs(dp);
}

// TODO: Refactor into Stomp/Clap Detection class
void ofApp::saveJointsAndVel(const k4abt_joint_t joints[32],
	double leftVel,
	double rightVel,
	uint64_t timeStamp)
{
	//STOMP DETECTION
	m_leftAnkle = joints[K4ABT_JOINT_ANKLE_LEFT].position;
	m_rightAnkle = joints[K4ABT_JOINT_ANKLE_RIGHT].position;
	m_lastLeftVel = leftVel;
	m_lastRightVel = rightVel;
	m_lastTimeStamp = timeStamp;
}

//TODO: Refactor into Stomp Detection class
double ofApp::calculateVelocity(const k4abt_joint_t joints[32], int leg, uint64_t timeStamp)
{
	//KNEE_LEFT 19, ANKLE_LEFT 20, KNEE_RIGHT 23, ANKLE_RIGHT 24
	double dp = 0;
	if (leg == LEFT) {
		dp = calculateYDist(joints[K4ABT_JOINT_ANKLE_LEFT].position, m_leftAnkle);
	}
	if (leg == RIGHT) {
		dp = calculateYDist(joints[K4ABT_JOINT_ANKLE_RIGHT].position, m_rightAnkle);
	}
	double dt = double(timeStamp - m_lastTimeStamp) / 1000000;
	return dp / dt;
}

//TODO: Refactor into Stomp Detection Class
int ofApp::didStomp(double velocity, double lastVelocity)
{
	if (velocity <= 700.0) {
		if (lastVelocity > 1500.0) {
			return 1;
		}
	}
	return 0;
}
```

<a name="face"></a>
#### Recognizing emotion with azure

To detect emotions of a person in front of the Kinect. The code below shows
how we grab an image which came from a Kinect for Azure format. We used the SDK's function to access the image buffer (a pointer to the RAW image data). We decoded the buffer to convert the raw buffer to a JPEG format using OpenCV.

We used the MS CPP REST SDK that has been around for a while to send image to the Cognitive Services Face API. We sent the binary file input to get back one of the 8 emotions that the Face API classifies (with % of confidence). We had to open the istream with the proper formatting to avoid the Face API failing to return an emotion with no error. We initially did not have the proper format for the anonymous function and got hung there for a while.


<a name="viz"></a>
#### Making responsive graphic visualizations

The background color of the scene changes when Face API returns an emotion. We did not use all of the classification possible: Anger, contempt, disgust, fear, happiness, neutral, sadness, or surprise.

 This code sets the background color.

 ```c++
 bool Device::updateColorInDepthFrame(const k4a::image& depthImg, const k4a::image& colorImg)
 {
   const auto depthDims = glm::ivec2(depthImg.get_width_pixels(), depthImg.get_height_pixels());

   k4a::image transformedColorImg;
   try
   {
     transformedColorImg = k4a::image::create(K4A_IMAGE_FORMAT_COLOR_BGRA32,
       depthDims.x, depthDims.y,
       depthDims.x * 4 * static_cast<int>(sizeof(uint8_t)));

     this->transformation.color_image_to_depth_camera(depthImg, colorImg, &transformedColorImg);
   }
   catch (const k4a::error& e)
   {
     ofLogError(__FUNCTION__) << e.what();
     return false;
   }

   const auto transformedColorData = reinterpret_cast<uint8_t*>(transformedColorImg.get_buffer());

   if (!this->colorInDepthPix.isAllocated())
   {
     this->colorInDepthPix.allocate(depthDims.x, depthDims.y, OF_PIXELS_BGRA);
     this->colorInDepthTex.allocate(depthDims.x, depthDims.y, GL_RGBA8, ofGetUsingArbTex(), GL_BGRA, GL_UNSIGNED_BYTE);
     this->colorInDepthTex.setTextureMinMagFilter(GL_NEAREST, GL_NEAREST);
     this->colorInDepthTex.bind();
     {
       glTexParameteri(this->colorInDepthTex.texData.textureTarget, GL_TEXTURE_SWIZZLE_R, GL_BLUE);
       glTexParameteri(this->colorInDepthTex.texData.textureTarget, GL_TEXTURE_SWIZZLE_B, GL_RED);
     }
     this->colorInDepthTex.unbind();
   }

   this->colorInDepthPix.setFromPixels(transformedColorData, depthDims.x, depthDims.y, 4);
   this->colorInDepthTex.loadData(this->colorInDepthPix);

   ofLogVerbose(__FUNCTION__) << "Color in Depth " << depthDims.x << "x" << depthDims.y << " stride: " << transformedColorImg.get_stride_bytes() << ".";

   transformedColorImg.reset();

   return true;
 }
 ```

Underfoot animations.



<a name="sounds"></a>
#### Generating sounds with steps

Different sounds for different facial expressions.


<a name="collabo"></a>
## Future Directions

### Seeding collaborations

Should this work continue...
A musician could augment these efforts in the following ways...
