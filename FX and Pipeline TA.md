The text below is a transcript of my [portfolio demo video](https://vimeo.com/1107621546).
## The Job
For the racing film Fuel Up I was responsible for all VFX for the sequence where the main character's boyfriend dies in a violent racecar crash on a rainy night. The director and producers intended to shoot several of these shots against a volume wall, therefore everything I built needed to run in real-time in UE5.
## The Plan
I decided I wanted to take advantage of the incredible FX tooling in Houdini to art-direct the car destruction sim, then combine the baked results with a real-time rain sim in UE5.
## Pipeline
It was very important to me that the car destruction sim had softbody deformations. The clearest path to a Houdini->Unreal pipeline for softbody deformations was by baking the deformations into Vertex Animation Textures, which would be fed into an Unreal Material Function that drives the world position offset and normals on a static mesh.

The first problem I ran into was that my racecar was not a single deforming soft body but was in fact nearly 100 potentially detachable pieces. The Unreal VAT Material Function for soft bodies expects a mesh without discontinuities, therefore each of these deforming pieces would need their own VAT material.

Furthermore, the racecar had 12 different materials using four different UV coordinate systems. Many of my deforming pieces used more than one of these materials, which meant that for the worst case I would need to create 12x100=1200 VAT materials.

In Houdni I created a TOP network that scheduled VAT renders for each piece. For Unreal I wrote a Python script that would go per piece and:
1. Import the VAT and associated mesh
2. Create material instances for all 12 materials for that piece
3. Apply the VATs to those instances
4. Assign the material instances to the static mesh
5. Shove the static mesh into a Blueprint with the others

To drive the deformation animation I added a function to the Blueprint for updating the "display frame" material property across all pieces. I then modified a Sequence Animation Director blueprint to forward sequencer keyframes to this function.

Besides the softbody deforming pieces, each racecar was made up of nearly 2000 tiny rigid bodies, mostly contributed by the window glass. I knew I was occupying a significant portion of VRAM with the softbody VATs, so I opted to bring these rigid bodies to Unreal as an FBX skeletal animation.

Rain was executed as a Niagara system, and the mist particulate was brought in as a Niagara point cache and shaded as circular sprites. I also briefly experimented with using VDB animations for the mist but quality came at too steep a performance cost.
## Mist
I felt that it would dramatically improve the project to simulate the effect of rain aerosolizing into a fine particulate mist upon contact with the vehicle, and between the wheels and the ground.

I started with a simple rain sim, where raindrops die upon collision. Whenever a raindrop lands I spawn a cluster of ejecta points which bounce away for a single frame before dying. I also added ejecta point emitters at the bottom of the wheels.

I rasterize these ejecta points into a high resolution VDB volume, transferring ejecta velocities to the volume. I then advect the VDB with the pyro solver, treating it like a zero buoyancy, quickly dissipating smoke. The result is a cloudy mist look. To get the fine particulate look, I use the resulting VDB as a velocity field to transfer the misty motion to the ejecta points from earlier.
## Lookdev
I took advantage of the slab-based Substrate BSDF in UE5 to enhance the car materials. For the main body I created a dirt layer, a coat layer, and a base layer, and added view-angle dependent sparkles. All glass uses the thin translucent shading model.

The rain and mist are small sprites using the Thin Translucent shading model. They simulate the smearing effect of motion blur by stretching the sprite.