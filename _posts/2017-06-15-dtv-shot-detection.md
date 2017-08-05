---
layout: post
title: "Distant Viewing TV: Shot Detection"
categories: distanttv
---

This post explores how we built a shot detection algorithm
from scratch (after evaluating several existing libraries and
finding them deficient)

**This post is part of a series about the Distant Viewing TV
project. To see a full list of available posts in the series
see [Distant Viewing TV: Introduction](../dtv-introduction).**

The testing with face detection showed a need to incorporate
scene detection into our algorithm as well. There is a relatively
low recall but high precision, which we can use to combine over
still images in a given seen to detect where characters are. I found
a number of libraries that attempt to find shot or scene breaks,
including [PySceneDetect](https://github.com/Breakthrough/PySceneDetect)
and [ShotLogger](https://github.com/jgbutler/Shot-Logger). In testing,
both seemed too high-level for our needs. Testing the python
library directly on our movie file found only a small subset of the
total available shot breaks (despite the name, their software is
actually looking for shot breaks rather than scene breaks).
The ShotLogger software has great results on its website, but I was
not able to get it running on my own in a reasonable amount of time.
At any rate, it will be very useful to write our own scene detector.

Ideally, I want a function that takes two images and compares how
'close' they are to one another. A shot break can then be classified
as an abrupt change in this metric between adjacent frames. This metric
may then also be useful in telling us when the actual scene has ended
or when a camera angle has been returned too from a previous break.
As a starting point, I converted each frame to a 100x100 pixel image
and then compared two images using Euclidean distance in the HSV color
space. I used HSV coordinates; I have found in many applications
that it works better than the standard RGB coordinate system and rarely
if ever perform worse. Down-sampling to a smaller grid allows the
metric to stay small even when the camera or characters are moving
within a shot.

Here is an example of the metric applied to adjacent shots, indexed
by the frame number. I added the red dot to show where their is a true
shot break.

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img09.png)

In this example the algorithm does a good job of highlighting an abnormally
large change at the actual scene break. The values around 3695, which are
not as large as the shot break, are large because the characters are moving
around the set. I manually tested this algorithm over randomly choosen runs
of about 150 frames. While most worked well, it did not take long to find an
anomalous example. Here is one from a scene at Darrin's office with the three
real shot breaks denoted by red dots:

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img10.png)

There is no way to use a threshold that captures the real breaks but avoids all
of the non-real breaks. What is going on here? I took a look at the transitions
with large values at 4139 and 4164. Around both frames we have very fast movement
with Larry darting across the scene:

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img16.png)

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img17.png)

As I thought about what is happening to the Euclidean distance in HSV space, it
began to make sense that fast movement could be a problem. I converted the metric
to measure instead the median absolute value of the differences between the images.
Because character movement show not change the majority to the 100x100 pixel bins,
this should be much more robust to movement. Looking at the results, it certainly
helps smooth things out and to clarify where the real scene breaks are:

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img11.png)

We still seem to have a problem with frame 4155. Although there is a scene break,
the prior image is interlaced with the next so the change is too gradual to
detect with a high signal:

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img15.png)

At first, I tried changing grid size from 100x100 to 25x25, but this
was not particularly helpful (I ultimately kept the 25x25 grid as it helped
slightly in other examples):

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img12.png)

After searching around on the internet, I learned that the interlacing in
the video is a known problem when converting from formats meant for older
TVs. There is also a quick way to fix this when calling ffmpeg. Using the
deinterlace option, the still images were now de-interlaced:

``` sh
ffmpeg -i input.VOB -vf fps=6 -deinterlace img/out%06d.png
```

Using these new images along with the median absolute deviation (in
truth, I'm using the 40% of the absolute deviations as it seemed to
be slightly more stable) fixes the problem in this case. Real
transitions are all above non-real transitions:

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img13.png)

The difference between the real break and the non-real one in this example,
however, is still not very large. It seemed unlikely that this would work
for an universal cut-off to denote shot breaks. I needed a way of making
the metric less extreme in the presence of movement. To do that, I re-ran
ffmpeg once again with the frame rate set to 24 frames per second. With
this data, we see a true separation between the true and false scene breaks
(note that the indicies have changes because there are now 4 times as many
frames):

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img14.png)

In this example there is a clear range of cut-off values that would work (e.g.,
0.4) and that seem reasonably accurate for indicating scene breaks. It may
appear that we could have just use the higher frame-rate to begin with and
skip all of the other steps. For example, here is the mean difference metric
on the deinterlaced 24fps data:

![image](https://statsmaths.github.io/blog/assets/2017-06-15-dtv-shot-detection/img18.png)

Even at the higher frame rate, the original metric does a poor job of
differentiating movement and shot breaks.

*The next post in this series is available at:
[Distant Viewing TV: Introduction](../dtv-first-training-set).*
