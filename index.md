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

One approach that the EASE Lab has adopted in the effort to extend access to STEM empowerment has been to enhance activities that young people of diverse backgrounds enjoy, and create connections between those passions and STEM (through computing, primarily).

Accordingly, we imagine that a fully-fleshed out version of this project would involve:
-people noticing computer-controlled characters doing steps among themselves on the screen.
-people approach the Kinect and notice that they join the seen.
-curious participants begin trying out steps of their own, informed by what they see the on-screen characters doing.
-The on-screen characters have been learning from what kinect steppers around the country have been doing in front of the Step Kinect installations.
-The characters on screen begin to follow what one or more participants are doing.
-Participants will feel that stepping is fun and that the new Kinect experience was amazing - and they'll be curious about AI.
-They realize that they'd like to create experiences like Step Kinect and they go into STEM.

<a name="stepkinect"></a>

## The Technical Exploration Sprint

### Adding Azure to Kinect-enhanced steps in a sprint

Our first sprint goal was to integrate different aspects of the Azure Kinect's features that have not been available before in previous Kinect models while introducing compelling contexts.

[the MS Azure Kinect SDK site]() shows the full list of the device's features. Our initial efforts explored only a subset: video sensing and Azure Cognitive Services.

We have developed a system to use the Kinect camera to track the points on a person's leg to determine when a "stomp" is happening or when hands are clapping. While people are (hopefully stepping) in front of the camera, we send full frame images to Cognitive Services face API to receive the likely emotion expressed during the frame.

We elected to use openFrameworks extensively in the first sprint. This gave us the ability to explore making every stomp that is registered trigger a graphical visualization under the stepper's foot. We also play back different sounds for stomps than we do for claps.

We change the background of the scene to red when the Face API informs the program that the stepper is considered to be showing angered face. When happiness is detected, the background is light gold.

<a name="stepsense"></a>

#### Sensing steps with cameras

The ver 0.1 code snippet at the bottom of this subsection gives an example of how we went about detecting when a stepper stomps the ground. The approach is reasonable albeit not super robust at the moment. To get the proof of concept working, we needed to expose timestamp information from the openFrameworks codebase by adding functions to retrieve the private timestamp variable, so that we could determine DP/DT of two samples - only using the Y. We did not use pure distance formula because we were only concerned with a leg going up or down, which the Kinect maps to the Y coordinate. The program considers a stomp to occur When the current velocity is small, but the preceding velocity was large.

We had to modify source files from the openFrameworks Kinect add-on to expose the timestamps.

code is from the ofapp.cpp

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

We drew up a design for creating an effect that triggers when a stomp happens and briefly gives particle effect graphical feedback. The program can inform how mild or wild the effects are expressed based on the speed of the stomp or the last read emotion of the stepper.

The initial approach to emitting graphics involves importing an object such as a sphere and use its one coordinate to guide its path. The number of spheres, their appearance, and their paths are works-in-progress. openFrameworks appears to have a myriad of options for importing objects.


<a name="sounds"></a>
#### Generating sounds with steps

openFrameworks features a straightforward method for playing sound. We currently have short mp3 files of sound effects, one specifically to play when a stomp event is registered, a different one for when a clap event is registered. Currently, we are fine tuning appropriate sound effects and acceptable latency so that the sounds seem responsive to their triggers. We see the soundscape as something that change according to emotion as well.


<a name="collabo"></a>
## Future Directions

Once we have results that we're happy with for one stepper in front of the camera, we will look to sense multiple bodies in the frame. The Cognitive Services Face API returns the bounding box attributes of the face it is classifying an emotion for, which will give us an ability to change a particular stepper's outline or body color instead of the entire scene, for more interactions - can your team take on different personalities or stay in an intense anger face for the entire show (not an easy feat).

We expect to give the microphone array some added functionality through code. We can filter some of the input.


We learned that getting Visual Studio running can take time.
getting environment variables and linkers all working didn't come out of the box. Some steps are not documented. When you open a solution intitially, it defaults to a 32-bit debugger and a 64-bit debugger is needed and the errors do not inform you very well.
Going through solution property pages to add additional include directories such azure kinect and body tracking libraries. The sentence that reminds users to "make sure that you remember to include such and such dll from this bin and library" without saying how to do that and make sure that you have it.
The whole of visual studio works well when a solution file can be opened.

### Seeding collaborations

Should this work continue...
A musician could augment these efforts in the following ways...
