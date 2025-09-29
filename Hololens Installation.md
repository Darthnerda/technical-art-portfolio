In collaboration with multimedia artist VJ Um Amel I created a series of digital installations about the January 6th storming of the US capitol building. The project involved work in Unity, Blender, Kubernetes, NGINX, MongoDB, and with NLP techniques such as LDA, T-SNE, OR ratios, and BFS associations.

![Procession-03.webp](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/8/Procession_03_37946ad61d.webp)

The principal artwork was an AR installation displayed in Microsoft's Hololens AR goggles. The experience presented a scale model of the 'capitol corridor' in Washington DC, and an interactive timeline that allowed users to scrub through thousands of geolocated and timestamped videos taken on the day, starting hours before Trump's rally near the Washington monument, through the crowd's procession to the capitol building, confrontations with law enforcement, breaking into the structure, and the fatal shooting of Ashli Babbitt by capitol security as she attempted to break into the Congress chambers.

![Procession-01.webp](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/8/Procession_01_f571d727ea.webp)

The main challenge of the project was optimization for smooth performance on the Hololens, which is not a particularly powerful system. The two angles to this challenge were the scale of the geometry and the multitude of videos.
### Geo Optimization
My scale model had hundreds of detailed structures, thousands of trees and bushes, and dozens of distinct materials. And in the naive setup framerate was severely impacted. The first step as always was profiling. I would begin a debug build on the hololens, and through my solution in visual studio I could connect the Unity profiler. As expected, we were severely GPU bottlenecked. The largest offender was draw calls, which I could trace back to the abundance of distinct geometries and materials. My solution was:
1. Divide the city area into large sections
2. Bake all the geos in a section into a series of LOD meshes and a single material
This quite resolved the performance issue.
### Video Optimization
We had tens of thousands of videos in our dataset, and at any given time there could be up to 200 videos overlapping in time. The videos were not large - a few mb at the largest - and were almost all in the efficient HEVC format - but the entire collection ran about 40gb, which would not fit comfortably on the Hololens. So I decided to host the videos on a separate server and stream the videos into the Hololens.

After a false start with the DASH streaming protocol, I created a custom Docker image that built NGINX with the HLS extension, then deployed that on my Kubernetes cluster with the appropriate NGINX configuration. It took some tweaking of the buffer and asynchronous IO settings on the HLS extension to get streaming to begin without delay and to support many concurrent streams.

Even with this setup I was seeing that upon moving the timeline, using sim-time to drive the videos playtime would overload the streaming server with relatively heavy seek operations. My solution was to disable seeking, and instead to play each video from its beginning and on a loop so long as our sim time intersected the video's duration `start_time < t < start_time + duration`. With this method we do lose exact synchronization between the videos, but this was a fair tradeoff as there were very few times where multiple videos captured the same action.

