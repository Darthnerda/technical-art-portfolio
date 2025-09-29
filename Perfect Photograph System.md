For the story-action game Abandon by Dual X studios I owned and developed the photography gameplay mechanic. The idea was that when the player character takes a photo of an object relevant to their childhood, the object transforms into a surreal form, and they write a diary entry about the photo that advances the narrative.

(gif of the effect in action)

The four implementation milestones were:
- A way to author photograph trigger zones/orientations
- Photograph controls and UI
- Photograph capture and persistence
- Object transformation
### The Dance of Capture and Transformation
It's straightforward enough to capture a screengrab from the player camera. But the challenge was in squaring two competing requirements: 
1. The diary entry image ought to show the object in its post-transformation form
2. Taking the photograph causes the object to begin its transformation animation.

(timing diagram)

The problem was simple: at the moment of taking the photograph the object won't be transformed yet, but the resulting image needs it to be. My solution: at photo time I make the camera FLASH, and while the player's screen its whited-out, I swap the object to its post-transformation state for a frame, take the photo with a second camera, then reset it back in time for the white-out to dissipate. The transformation then plays out over time as normal.
### Trigger Setup
To trigger the effect, the player should be looking at the object, however there are many possible compositions in which the object is in frame while not actually being the obvious subject of the image. To solve this I constrain the trigger to satisfactory angles by requiring two triggers to be tripped concurrently: the look trigger and at least one camera position trigger.

(image of the triggers in the editor)

**The look trigger** is a sphere around the object, and is tripped when a raycast from the center of the player camera intersects it. **A position trigger** can be any shape, but by default is a cone with its narrow end near the object and its wide end far from it. It is tripped when the player camera is inside the shape. Used together a level designer can constrain the possible camera positions and orientations according to whatever compositions they want to evoke, while still giving the player a lot of wiggle room.

The tool allows designers to set an upper and lower bound on a valid 'shot-size' for the object in frame, expressed as a percentage: with 0% being smaller than a pixel and 100% being the frame height. The current shot size of the object is calculated using its bounding box and the camera intrinsics using the formula `(cam_distance * sensor_height) / (bound_height * focal_length)` with all values in millimeters.

### Communicating With The Artists
I had many conversations with the design and programming leads to align the system's design with the vision for the game and how our other game systems were working. Then once I had built the system I knew it would be helpful to make a short video as a reference and to teach the level designers how to author correct triggers, and how to ensure that objects can work with the transformation effect. That was the origin of the video below:

(the explainer video)