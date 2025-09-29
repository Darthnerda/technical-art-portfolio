For the indie game Abandon by Dual X Studios, one of the several monstrous antagonists was a slime creature called Beiruo - the 3D artists had created a great geo and mat, but because of its slimy nature I was to work out how to actually get the creature moving.

(The slime geo / mat)
### Assessing the Options
It seems to me that there were three main paths I could take:
1. Simulate softbody slime movements offline in Houdini then phase between them in realtime with a blend tree
2. Simulate sobody slime movements in real-time in-engine
3. Approximate softbody movement with a non-simulation trick in real-time in-engine

Option 1 could reach a high degree of realism in ideal conditions or with further constraint on the slime's movement style, but to take advantage of the higher fidelity I would need higher poly on the geo or animate baked bump maps, both of which posed different issues - and the dynamics on one movement would not blend very gracefully with other movements in the blend tree.

Option 2 could also reach a decent degree of realism, but after a few tests with a few softbody simulation packages, the team determined together that we didn't have the CPU budget for it - and devising a softbody sim with compute shaders would put us off schedule.

Option 3 thus seemed to be the most appropriate choice. So now the question was how to achieve it.
### Targets, Curves, POM
The slime movement script is assigned a gameobject towards which it will crawl. First we run A* to determine a `slime_path` to the target. Then the slime moves by reaching out then pulling itself along. It follows this program:
1. Set a `reach_point` a short distance along the `slime_path`
2. Deform the slime so that the nearest edge stretches out 150% past the `reach_point`
3. Move the root bone to the `reach_point` while proportionally undoing the deformation
4. Repeat

(moving the target around and the slime following)

My first version of evaluating the progress of the slime's stretch every frame made us of Vector3.SmoothDamp to create an ease-in-ease-out, but I felt that I wanted more granular control of the feeling of the movement, so I decided to do the progress evaluation as a linear evaluation of an AnimationCurve provided in the editor. I had a curve for stretching and a curve for de-stretching.

While setting up the deformation step (2), I suspected that the geo wouldn't have enough definition to make world position offset look correct but a quick test put that worry to bed. I also wanted to give the slime a good wiggle to it, and this in fact did prove too much for the available geo.

So to achieve a wiggle I turned at first to an animated bump map, but that still felt pretty surface-level - I finally found what I was looking for by animating the heightmap on a parallax occlusion mapping effect.

### Slime trail as procedural decal projection
As slimes are wont to do, Beiruo was to leave a slimy trail wherever it went. At first I decided I would adapt Sebastian Lague's excellent Unity bezier-based [path creator](https://discussions.unity.com/t/bezier-path-creator-free/730975) tool to achieve this. The path tool was meant to generate meshes from beziers in the editor, but I adapted it so that I could procedurally add new points along the bezier and re-generate in realtime. At the start of the deformation stage of the slime move cycle it would expand the slime trail mesh, setting the uv to exactly match the length of the movement so the trail never appears to be stretched or squished.

(the slime trail in action)

Decal projection became important when it was clear that Beiruo needed to be able to up and down stairs. With my mesh-based path method, the trail would not know to fold to the outline of the stairs. So I moved my slime trail path to an unused part of the map, captured it with an orthographic camera, and decal projected it onto the world around Beiruo. This way the trail would mold to any surface it went along.