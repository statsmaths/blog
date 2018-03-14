---
layout: post
title: "Distant Viewing TV: Initial Training Set"
categories: distanttv
---

We construct a training set of labeled scene and shot breaks
from a single episode of Bewitched. Using this, we are able to
verify the performance of our shot detector. It also allows us
to sanity-check our face detector.

**This post is part of a series about the Distant Viewing TV
project. To see a full list of available posts in the series
see [Distant Viewing TV: Introduction](../dtv-introduction).**

At this point, we have a prototype for: face detection, face
disambiguation, and shot detection. These were tested and tuned
by randomly applying them over shot snippets of a single episode
of Bewitch. The next logical step to me seemed to be coding each
and every shot break in the episode. This way, we can be more
certain that the shots are all being captured and feel confident
pushing forward. I went through the episode frame by frame, coding
where each shot took place, the scene location, and whether a shot
depicts a close-up on a single character.

Here is the shot detector metric against my original coding over
a range of frames near the end of the episode:

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img19.png)

For many of the breaks there is a clear and obvious signal in our
detection algorithm. There seems to be only one real potential false
signal around 31750; it occurs during a camera panning the set at
a relatively fast rate. However, we can likely remove these anamolies
by not only using a fixed cut-off but also looking to see if the metric
'jumps' at a specific spot. In this negative example the metric smoothly
increases and decreases from a large value, unlike in any of the positive
examples.

More worrying are the five shot cuts that do not seem to be
picked up by the metric. Looking closer at them, these all refer to
fade-ins and fade-outs. Given the nature of a fade, all happen slowly
and do not make the metric spike. These fades need to be handled as
a special case by detecting when the scene is entirely black. Detecting
black scenes required a bit of work because the HSV color model has many
equivalent ways of representing pure black. I then switched out saturation
for chroma; this makes the color space look like an ice-cream cone rather
than a cylinder. Once applied, I then coded each frame with whether it
was completely black or not. Looking at all of the shot breaks, we can
see that nearly every one is detected correctly with the exception of
the cartoon intro between frames 3000 and 4000 (I did not include cartoon
shot breaks in my data):

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img20_1.png)

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img20_2.png)

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img20_3.png)

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img20_4.png)

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img20_5.png)

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img20_6.png)

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img20_7.png)

The spikes in the last plot right at the end of the episode are title cards
that probably should be considered shot breaks but were not included in my
training data (though they are found by the fade-in fade-out detector). The
only values that seems like a legitimately bad point is the false negative around frame
number 4400 and the false positive around 28300. In the first example, Endora magically
appears outside of Darrin's office in a cloud of strobe lights. You can see the bad
data point in all of its 1960s television special effects here:

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img21.png)

In this case I am truly stumped. I think for now treating this as a shot break is
probably not too terrible. The false negative comes from a cross-fade between two
scenes in the Stephens' house. This seems possible to fix, but niche enough to avoid
worrying about too much at the current moment.

Coding just the shots took around 4 hours, and I did not have time to code all of
the characters in each shot. We can get some validation for the face detector though,
keeping in mind that it is already known to be fairly noisy with regard to just a
single shot. For each character, I identified each detected face with a character
if the similarity score exceeded 0.7. Here are the scores for each character for
each detected face. I've colored the points by the identity of the scene.

The secretary is only in the office scenes, and is in fact detected nowhere else:

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img22_6.png)

Endora is similarly only found in the kitchen and outside of Darrin's office:

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img22_5.png)

Larry does not show up until the office scene, is the only character in his
own house, and never enters the Stephens' kitchen:

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img22_4.png)

Darrin is in the majority of scenes, but not in Larry's house nor is he in the
kitchen after the introduction. We see this show up in the dataset:

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img22_2.png)

Samantha is oddly missing from the first scene until Darrin returns home about
halfway through the episode. The character detector makes one errant
identification outside of Darrin's office but otherwise seem to work well:

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img22_1.png)

The worst character is the character I just called 'Lady', the client Darrin
is trying to win over in the episode. She is mistaken for Samantha in a few
opening frames as well as towards the end of the episode:

![values](https://statsmaths.github.io/blog/assets/2017-06-22-dtv-first-training-set/img22_3.png)

Overall these indicate that the classifier is working reasonably well.

*The next post in this series is available at:
[Distant Viewing TV: Building the Distant Viewing Toolkit](../dtv-scope).*

