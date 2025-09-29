So I've had a space game idea in the back of my head for a while now, so I thought it would be fun to mess around with calculating orbital trajectories. The idea is tOp sEcReT, but I can say that it would involve many hundreds of thousands of active agents managing their own orbital trajectories. That seemed like the perfect excuse to use the massively parallel capabilities of the GPU, and thus began my journey into compute shaders and procedural geometry!

Very naively I figured I could simply simulate the gravity force. So I threw some gravity physics into a simple GPU kernel and gave it a few tens of thousands of massy objects. To dodge the performance cost of transferring the resulting positions back to the CPU, I used the resulting positions as UV coordinates into a texture, so that essentially each massy object just turns on whatever pixel in the texture that best approximates its location. I colored the pixel by the object's three component velocity, and voila, clown barf!

![01_GRAVSIM_QUAD.gif](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/9/01_GRAVSIM_QUAD_d1480a4692.gif)

Okay, so that's pretty cool. But what about making actual trajectories out of this? So long as I'm doing the actual gravity force calculation, I'd be doing things iteratively. That is, later points in the trajectory would have to be calculated only after earlier points have finished. This would mean that each thread in my GPU kernel would have to share position data at every step. Oof, that sounds like thread-blocking hell.

I decided I would try out another approach. Specifically, the [Universal Variable Formulation](https://en.wikipedia.org/wiki/Universal_variable_formulation#:~:text=In%20orbital%20mechanics%2C%20the%20universal,also%20parabolic%20and%20hyperbolic%20orbits)! The Universal Variable Formulation, hereon UVF, is an analytic approach to determining how far around its orbit an object has gone at a given time. Being analytical, every t could be calculated independently, and therefore in parallel. With the UVF, we wouldn't have to do any memory sharing! Instead, every t that we want to calculate could be on its own thread, running in parallel.

By running the kernel without parallel t values, we get the following...

![02_ORDERED_GAME_OBJECTS.gif](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/9/02_ORDERED_GAME_OBJECTS_14f75c9085.gif)

The orderliness of the orbits is due to the orbits being initialized in an orderly fashion. Let's add some randomness...

![03_RANDOM_GAME_OBJECTS.gif](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/9/03_RANDOM_GAME_OBJECTS_784dd89239.gif)

Alright! So now we've got the UVF working. But this still doesn't look like trajectories. We need to form parallel time results into lines.

This is where a line mesh would be perfect! To batch all the trajectory calculations together, we use a single mesh object with each object's path as its own submesh. We then generate line strip indices for each submesh, and one large vertex buffer that is shared between each mesh.

We then pass the vertex buffer into the compute shader as a [UAV](https://docs.microsoft.com/en-us/windows/win32/direct3d11/typed-unordered-access-view-loads) buffer (Unordered Access View), which allows each parallel thread to modify vertices concurrently. And voila:

![04_PROC_LINEMESH_TIMESTEP.gif](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/9/04_PROC_LINEMESH_TIMESTEP_4cd704e80e.gif)

And now UVF gives us some cool features. For example, we could add an eccentricity modifier, and animate it in real time, creating this 'order out of chaos' effect:

![05_PROC_LINEMESH_ECCENTRICITY.gif](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/9/05_PROC_LINEMESH_ECCENTRICITY_8085bbe370.gif)

Or we can even animate the magnitude of the time steps, creating this cool vortex looking thing:

![06_PROC_LINEMESH_TIMESTEPANIM.gif](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/9/06_PROC_LINEMESH_TIMESTEPANIM_6dd63cb147.gif)

The main downside to using the UVF is that its only deals with 2-body interactions. That is, it only models interactions between one very small object and one very massive object. This means that if I want to model interplanetary trajectories, I'm going to have to implement some sort of 'context switcher' that changes which body the small object is being affected by as it passes into and out of their spheres of influence.

This isn't unheard of at all. In fact, it's exactly how Kerbal Space Program works internally. At every point in an object's trajectory, they determine whether that position has entered or left any massive objects' [sphere of influence](https://en.wikipedia.org/wiki/Sphere_of_influence_\(astrodynamics\)). If the object has entered a different sphere of influence, then the algorithm makes the necessary context switch and calculates the rest of the trajectory using the new massive body instead.

That's where I'm at now! I intend to pick this up again when I have a moment to breath.