# Gradient Domain Fusion
This project explores gradient-domain processing, a simple technique with a broad set of applications, including blending, tone-mapping, and non-photorealistic rendering developed in the 2003 [paper](https://www.cs.jhu.edu/~misha/Fall07/Papers/Perez03.pdf) by Peréz, Gangnet, and Blake. This project is based on this [assignment](https://yxw.cs.illinois.edu/course/CS445/Content/projects/gradient/ComputationalPhotography_ProjectGradient.html) in computational photography at UIUC in spring 2023.

## Table of Contents
1. [Poisson Blending](#poisson-blending)
1. [Mixed Gradient Blending](#mixed-gradient-blending)
1. [Color2Gray](#color2gray)
1. [Laplacian Pyramid Blending](#laplacian-pyramid-blending)
1. [Non-Photo Realistic Rendering](#non-photo-realistic-rendering)
1. [Poisson Sharpening](#poisson-sharpening)
1. [Acknowledgments/Attributions](#acknowledgmentsattributions)

## Poisson Blending
Poisson image blending is a gradient-domain processing technique to seamlessly blend a source image into a target image. The idea is to preserve the gradient of the source image without changing the background pixels. For better or worse, this technique changes the color of the object, but it can still work well since people often care more about the gradient of an image then the overall intensity. 

The objective function can be formulated as a least squares problem. Given the pixel intensities of the source image s and the target image t, solve for the new intensity values v within the source region S.

$$
v=argmin_v \sum_{i\in S, j \in N_i \cap S} \bigl( (v_i - v_j) - (s_i - s_j) \bigr)^2  + \sum_{i\in S, j \in N_i \cap \neg S} \bigl( (v_i - t_j) - (s_i - s_j) \bigr)^2,
$$

where each $i$ is a pixel in the source region $S$, and each $j$ is a 4-neighbor of $i$. The first summation maintains the gradient within the source region, and the second summation targets similar gradients between the bordering target background and source region.


![""](output/blend-panel_penguin_snow.png "title")
![""](output/poissonBlend_penguin_snow.png "title")

![""](output/blend-panel_penguin-snow2.png "title")
![""](output/poissonBlend_penguin-snow2.png "title")

![""](output/blend-panel_moon_fireworks.png "title")
![""](output/poissonBlend_moon_fireworks.png "title")

![""](output/blend-panel_redpanda_beach.png "title")
![""](output/poissonBlend_redpanda_beach.png "title")


![""](output/blend-panel_xwing_tikal.png "title")
![""](output/poissonBlend_xwing_tikal.png "title")

![""](output/blend-panel_houseUp_mountains.png "title")
![""](output/poissonBlend_houseUp_mountains.png "title")

This blend did not work well since the object image became too dark. The sky background in the object image is lighter than in the background image, so Poisson blending darkens the object image to seamlessly blend the sky.

[Back to top](#table-of-contents)
<br>
<br>





## Mixed Gradient Blending
Mixed gradient blending is similar to Poisson blending above, but it uses the gradient in the source or target image with the larger magnitude. Mixed gradient blending is useful for objects with transparent backgrounds. 

$$
v=argmin_v \sum_{i \in S, j \in N_i \cap S} \bigl( (v_i - v_j) - d_{ij} \bigr)^2  + \sum_{i \in S, j \in N_i \cap \neg S} \bigl( (v_i - t_j) - d_{ij} \bigr)^2,
$$

where $d_{ij} = argmax\lbrace{|d| : d \in {s_i - s_j, t_i - t_j}\rbrace}$

![""](output/blend-panel_batman_wall.png "title")
![""](output/poissonBlend_batman_wall.png "title")

![""](output/blend-panel_charizard_magma.png "title")
![""](output/poissonBlend_charizard_magma.png "title")

![""](output/blend-panel_link_green.png "title")
![""](output/poissonBlend_link_green.png "title")

This example does not work well, since it produces a halo effect around the object, which may require a better segmentation to remedy it.

[Back to top](#table-of-contents)
<br>
<br>



## Color2Gray
Sometimes when converting a color image to grayscale (e.g., when printing to a laser printer), important contrast information is lost, making the image difficult to understand. Using gradient-domain processing, one can create a gray image that has similar intensity to the rgb2gray output but has similar gradients to the original RGB image. The constraints are similar to the mixed gradient problem except it maximizes the magnitude in gradients across RGB channels at each pixel location.

![""](output/color2gray_colorBlind4.png "title")
![""](output/color2gray_colorBlind8.png "title")

The middle image is the result of converting the colorblind image (left) to grayscale using the using the OpenCV function color2gray. The number completely disappears. Poisson image editing (right) better maintains the contrast information, so the number remains visible after converting to grayscale.


![""](output/color2gray_red_tulips.png "title")
The increased contrast, however, can be excessive when applying this technique to a natural image as seen in the tulips.


[Back to top](#table-of-contents)
<br>
<br>



## Laplacian Pyramid Blending
Laplacian pyramids are constructed for both the source and target image. The source and target image can be blended together by combining each level of the pyramid using alpha mattes and then collapsing the blended pyramid into a single image. The key idea is that there is smooth blending at low frequencies and sharp blending at high frequencies. One advantage of Laplacian pyramid blending is that it will leave the source object color unchanged, however, it can lead to halo effects.

![""](output/object-panel_houseUp_mountains.png "title")
![""](output/laplacian_houseUp_mountains.png "title")
![""](output/blends-panel_houseUp_mountains.png "title")

The Laplacian pyramid blend works much better than the Poisson blend since the color of the object image is unchanged.

![""](output/object-panel_penguin_snow.png "title")
![""](output/laplacian_penguin_snow.png "title")
![""](output/blends-panel_penguin_snow.png "title")

The color of the penguin in the Laplacian pyramid blend did not change, but there are some slight halo effects around it. The Poisson blend better blends the target image and the background of the object image together. Also, since the overall intensity of background image is dark, darkening the penguin in the Poisson blend look more natural while the penguin sticks out in the Laplacian pyramid blend.


![""](output/object-panel_redpanda_beach.png "title")
![""](output/laplacian_redpanda_beach.png "title")
![""](output/blends-panel_redpanda_beach.png "title")

Overall, the Poisson and Laplacian pyramid blend give similar results in this example.


[Back to top](#table-of-contents)
<br>
<br>




## Non-Photo Realistic Rendering
Non-photo realistic rendering smooths images by retaining only the gradients along edges. This gives the image a plastic, toy-like effect. The constraints are similar to those in Poisson blending, except it incorporates a canny edge detector. Also, a scale factor of alpha is added, so that the colors are not too dull.

![""](output/npr_freya.png "title")
![""](output/npr_red-tulips.png "title")


[Back to top](#table-of-contents)
<br>
<br>




## Poisson Sharpening
In image sharpening, the goal is to increase changes in pixel values around edges. Instead of incorporating an edge filter directly, Poisson image editing scales up the image gradient.

![""](output/sharp_sharp-sea.png "title")
![""](output/sharp_sharp-link.png "title")


[Back to top](#table-of-contents)
<br>
<br>




## Acknowledgments/Attributions
Images Sources:
- [Batman logo](https://www.google.com/url?sa=i&url=https%3A%2F%2Fwww.supercoloring.com%2Fes%2Fmanualidades-de-papel%2Fplantilla-imprimible-del-logo-de-batman&psig=AOvVaw0riA2CN2QQOiDC_FMsLWop&ust=1677894143565000&source=images&cd=vfe&ved=0CBAQjRxqFwoTCLCQh--1wP0CFQAAAAAdAAAAABAD)
- [Beach](https://calmatters.org/health/coronavirus/2020/04/orange-county-coronavirus-beaches-closed-california-state-parks/)
- [Charizard](http://clipart-library.com/clipart/BigAAneBT.htm)
- [Concrete Wall](https://www.etsy.com/listing/949270109/concrete-wall-peel-and-stick-mural)
- [Fireworks](https://www.bbc.co.uk/programmes/articles/4gGSvzclrSpt07tRzhrmykB/eight-fizzling-facts-about-fireworks)
- [Green Background](https://www.impactplus.com/blog/the-psychology-of-design-the-color-green)
- [House from Up](https://www.artstation.com/artwork/bKzZDa)
- [Link](https://www.drawingskill.com/art/82828)
- [Magma](https://scitechdaily.com/magma-beneath-a-long-dormant-volcano-has-been-observed-moving-upwards/)
- [Moon](https://www.calcalistech.com/ctechnews/article/91yk29jmp)
- [Mountains](https://peakvisor.com/adm/colorado.html)
- [Ocean](https://www.thoughtco.com/why-is-the-ocean-blue-609420)
- [Red Panda](https://socksforanimals.com/blogs/wildlife-projects/red-panda-network)
- [Red tulips](https://www.wttee.com/?category_id=2064633)
- [X-Wing](https://starwars.fandom.com/wiki/X-wing_starfighter?file=Xwing-SWB.jpg)


[Back to top](#table-of-contents)
<br>
<br>
