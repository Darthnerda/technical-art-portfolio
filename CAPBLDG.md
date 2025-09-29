CAPBLDG is an interactive mosaic made of conservative memes posted by users of the social media Parler during the January 6th storming of the capitol building. It was designed to work both on the 15,360 x 4,320 pixel Vroom wall at UC San Diego and on the web in a crossplay fashion.

Visitors to the Vroom wall could visit a webpage on their smartphone that allowed them to control a cursor for enlarging images in the mosaic. You can simulate the experience by pulling up the vroom page on your laptop or desktop and the controls page on your phone. Try moving the vroom cursor around by swiping on the controls page.

[Vroom page](https://data.r-shief.org/CAPBLDG-canvas/?vroom=true)

[Controls page](https://data.r-shief.org/CAPBLDG-controls)

The web version uses the same machinery, but let's users control their cursor directly with their mouse. If multiple people were checking out CAPBLDG at the same time as you, you would each be able to see the other's cursor on the same page. Users can drag a slider to fluidly swap mosaics to a night theme and a second bank of images.

![CAPBLDG-03.webp](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/8/CAPBLDG_03_59020140cf.webp)

The installation's technical implementation is in two parts: client and server. To achieve the mosaic swapping effect in a performant manner, the client code swaps individual photos in the mosaic in a WebGL canvas. The pattern in which the images swap is controlled by a static fractal brownian motion noise shown below. Each noise pixel correlates exactly to one sub-image in the mosaic. Each noise pixel's luminance is compared against a threshold scalar to determine whether the associated mosaic image should swap. When you move the slider, you're updating the threshold scalar and all the pixels are checked again, resulting in a smooth emergence of the noise pattern as images swap.

![static_fbm_bigger.jpg](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/8/static_fbm_bigger_bd2388ff67.jpg)

To achieve the multiplayer and phone control capabilities, a Node.js websocket server keeps track of every opened Canvas and Control, and coordinates their communication. The basic interaction between Canvas, Server and Control is summarized by the graph below. To prevent crowding on busy days, Canvases are instanced with a cap of 18 simultaneous Controls, and stale connections are occasionally culled.

![CAPBDLG_network_sequence.png](https://pub-568840b43a02402fa8b7f4b45571f13c.r2.dev/8/CAPBDLG_network_sequence_5e9faf9516.png)

CAPBLDG is one part of the larger Capitol Glitch installation series that continues to be hosted on-site at the QI gallery in San Diego. Read more about the show at [galleryqi.ucsd.edu](https://galleryqi.ucsd.edu/capital-glitch/).