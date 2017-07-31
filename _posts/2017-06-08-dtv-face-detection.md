---
layout: post
title: Face Detection with OpenFace
categories: distanttv
---


We would eventually like to extract a sizeable collection
of both high and low-level features from moving images. The
feature we are most excited about is faceial detection.
Detecting faces is fairly novel for DH applications, which
have focused almost entirely on the analysis of shot breaks.
The location of faces in a shot have the potential give a much
richer and complete description of what is happening in a shot
both from the narrative structure and in terms of the visual
form. If faces could also be tagged with specific character
names, that even further give quantitiative data about the
structure of each television show. Given the potential
benefits, our initial excitement, and the presense of several
open-source libraries for this exact task, face detection
seemed like the most obvious starting point for our analysis.

Some research into face-detection libraries suggested
that the OpenFace would be a good place to start.
Developed at CMU by Brandon Amos, Bartosz Ludwiczuk,
and Mahadev Satyanarayanan, OpenFace utilizes neural networks
trained over a large corpus of images and provides read-made
tools for fine-tuning their model on new training data. The
library is being continually maintained and updated. It is
written in Torch (a well-maintained Neural Network library)
and Python (the language we have tentatively decided to wrap
our toolkit in anyway). All of these features make OpenFace
particularly promising.

OpenFace recommends running inside of a pre-built Docker container.
Docker is software for wrapping up applications inside of a
pre-built filesystem; all of the dependencies and settings are
pre-set making software run the same (in theory) across multiple
platforms and systems. If you are not familiar with docker, think
of it being a "light" version of a virtual machine. I have previously
used Docker on my mac, so there was no initial set-up for me to do
(it used to be the Mac and Docker were terribly difficult to set up,
but now with a custom-built Mac Docker system it should usually be
very easy). Running these two commands from a terminal started by
the Docker Quickstart App was all that was needed to get started:

```
docker pull bamos/openface
docker run -p 9000:9000 -p 8000:8000 -t -i bamos/openface /bin/bash
```

The examples ran out of the box without a problem. Next, I wanted to
run a face detector on my extracted images. Previously when using
Docker to process data on my local file system, I set-up an API
service on Docker using the python Flask module. This made sense
because the software would eventually be deployed in production and
the added infastructure would be useful. In this case, however, all
of that communication seemed needlessly complicated. The eventual
plan for the DTV toolkit is to have the entire thing run inside a
Docker container, so there should be no need to communicate with
the file system. Some quick searching showed that it is possible
to mirror the local file system on the docker file system with a
simple change to the start-up command:

```
docker run -v /Users:/Users -t -i bamos/openface /bin/bash
```

Now for running the python code that calles the OpenFace library.
Doing this revealed two disappointing (at least at the time)
observations. First of all, the python library was written for
Python 2.7 and does not seem to work with any of the Python 3.x
series. I'll save my long personal thoughts about the whole Python 2/3
thing for another venue, but had originally hoped to keep the
DTV toolkit written entirely in Python 3. Secondly, the python
example code reveals that the neural networks are used only for
face disambiguation; the face detection is done with a call to
dlib:

```{python}
import openface

align = openface.AlignDlib(args.dlibFacePredictor)
net = openface.TorchNeuralNet(args.networkModel, args.imgDim, cuda=args.cuda)
```

dlib is a C++ library that has a fantastic track record,
but uses some older techniques such as HOG detectors
rather than neural networks. It would be interesting to
see how well it stacks up to the propretary algorithms used
by Facebook, Apple, and Google.

Excited to see the results, I ran the OpenFace library over a few still
images. The output just gives the coordinates of an image's bounding box,
so it was hard to really visualize how well it is doing. It seemed to find
some faces, but not nearly as many as were in the dozen or so that I tried.
In order to really see how well the algorithm was working, I needed to
get python to output a visualization of all of the detected boxes. After
an hour or so of coding, I had a nice function that took an input image
such as this:

![search results](https://statsmaths.github.io/blog/assets/2017-06-08-dtv-face-detection/img05.png)

Drew a box around any detected faces, and saved the output as this:

![search results](img/img07.png)

With only a slight tweak of the code, I also decided to save a version
of just the face into a seperate file. In our example, the extracted
face is this:

![search results](https://statsmaths.github.io/blog/assets/2017-06-08-dtv-face-detection/img06.png)

I then began going through the 1500 output images from this one episode.
Opening and closing all of these image files was beginning to upset my
computer, and Previous crashed a few times. Thankfully Lauren showed me
a great trick with the spacebar on macOS that allowed me to almost play
the images as if they were a movie. This was a realy time saver and is
what finally made it possible to see enough images to qualitatively
evaluate how well it was working. Across the entire episode, we noticed
three main things about the dlib face detector:

- it has very high precision; in only two cases did we see it detect a
face that was not in fact a face.

- the recall is quite low; in particular it rarely finds faces when a
character is not looking directly forward. Additionally, it also seems
to miss many faces that are looking straight forward.

- when playing the files like a movie, most characters in a given shot
eventually get picked up by the face detector in at least one, and
sometime many, frames.

From this last observation, it seemed like a good idea to try to increase
the frame-rate of extracted images. I re-ran the ffmpeg command, setting
`fps=6`, and then re-ran the OpenFace script over the new images. As
hoped, the increased framerate made it so that nearly every character in
a shot eventually had their face detected in at least one frame.

It now seemed that dlib would be an acceptable library to use, but would
need to somehow take advantage of the moving image aspects of our data.
For example: if Samantha is in a certain part of the shot in the most
previous frame and the shot has not changed she is probably in nearly
the same exact location now (Bewitched is a particularly interesting
example, however, as the "magic" in the show does sometimes make
characters appear and disappear without the shot changing otherwise).
Using this type of logic requires being able to detect shot changes,
the topic of our next entry.

In a rush to see how well the face detection worked, I completely forgot
to calculate the results of the face recognition part of the OpenFace
library. The neural network for face detection takes a detected face
image and maps it into a 128-dimensional vector. In theory, pictures of
the same person should be close in this space and two pictures of different
people should be far apart. The open face library does not supply or
recommend any particular clustering or cut-off values for how to explicitly
use the 128-dimension embeddings. Forging ahead anyway, I calculated the
face embeddings for all of the faced detected by dlib.

I generally feel comfortable running libraries and building pipelines in
python, but for fast exploratory data analysis I always come back to R.
This is probably due to my own comfort in the language than anything else.
Therefore, I saved the results of the embeddings as a csv and read these
into R. I wanted to understand how well the embedding was describing each
face, but this is difficult because I currently have not labelled training
data to test against. So, I instead computed the first two principal
components of the embeddings and plotted all of the faces in two-dimensions.
Luckily I had the code for doing this ready to go from a demonstration I
used in a class from Fall 2015. The output is a fun visualization but also
quite insightful:

![search results](https://statsmaths.github.io/blog/assets/2017-06-08-dtv-face-detection/img08.jpg)

We see that most of the Larry images are in the upper right-hand corner,
Darrin is in in th lower right-hand corner, and Samantha and Endora are
on the left hand-side. Samantha and Endora overlap a bit, but Samantha is
mostly in the lower half of the bubble and Endora in the upper half. There
are also several one-off characters sprinkled around the plot, such as
the man with glasses in the lower middle. For the most part, at least,
the embeddings at least seem reasonable. A more quantitative analysis will
have to wait until we have a bit more data to play with. Interestingly,
we also see one cartoon face recognized (Darrin, but plotted near Endora)
as well as a strange point right in the middle of the plot that we
eventually determined was photograph or painting on the wall of Larry's
office.












