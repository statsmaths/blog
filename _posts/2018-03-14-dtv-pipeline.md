---
layout: post
title: "Distant Viewing: Frame Level Annotations"
categories: distanttv
---

In this post we explore the initial structure of the Python package **dvt**,
with a particular focus on frame-level annotations.

**This post is part of a series about the Distant Viewing TV
project. To see a full list of available posts in the series
see [Distant Viewing TV: Introduction](../dtv-introduction).**

The initial work we did on extracting features from moving images achieved 
good results and some interesting early conclusions. Generalizing this to a
more sustainable code-base was a key first step for us as we took up the 
coding aspect of the Distant Viewing project once again. The early code was
a clunky combination of a dozen scripts written in numerous programming 
languages (R, Bash, Python, and C++). A major bottleneck was that static frames
had to first be extracted to the hard drive using the **ffmpeg** library, and
then read back in by each subsequent step in the process. A more self-contained
approach was needed for building a usable and reasonably fast toolkit.

### Building a Library in C

We initially felt that C was a natural choice for building our toolkit. It is
known to be very fast, cross-platform, and is usually relatively easy to call
from other high-level scripting languages such as R and Python. Furthermore,
most of the libraries we wanted to interact with are already written in C or
C++: ffmpeg, dlib, HandBrake, OpenCV, Tensorflow, and Darknet. We started down
this path by building a distant viewing library that processed raw video files
by calling the C-level ffmpeg libraries APIs. This took a considerable amount
of time given the complexity of video encoding algorithms, but we learned a lot
along the way about how the raw video files are stored digitally. In the end
the library did exactly what we wanted: read in a video file frame by frame,
produced basic summary statistics of each frame, and saved the result. By not
having to save the intermediate frame results, the software ran incredibly
fast. It could process an hour of HD video material at 24 frames per second in
under 2 minutes. 

Our next step was to integrate neural networks into our library. The idea was 
to run a basic object detector written in TensorFlow to a subset of the frames
extracted by ffmpeg. This is where the difficulty started. TensorFlow uses a
build system called Bazel. While it seems to offer some great functionality, it
seems to stop the resulting binaries from being called as a dynamic library. In
other words, if we want to write C code that calls TensorFlow, it will be
difficult to do this without building it within the TensorFlow system. After
spending more than a week trying to get TensorFlow and ffmpeg to work nicely
together, and seeing that integration with DarkNet and OpenCV would be
similarly difficult, it seemed that a new solution was needed.

### Moving to a Python Package

Next up as a choice for building the pipeline was to write everything in
Python. This has the benefit of being easier to program and debug, handling 
most of the dependencies for us, and already having actively maintained
bindings for all of the C or C++ libraries we wanted to interact with. The
two downsides to this approach are a slight decrease in performance (speed)
and the increased difficultly of allowing the library to be called from other
high-level languages such as R. In the end, however, we have felt that the
move to Python has been a uniformly good choice.

An initial working version of the **dvt** Python package can be found on our
GitHub page: [Distant Viewing Toolkit (DVT) for the Cultural Analysis of
Moving Images](https://github.com/statsmaths/dvt). Though still very much a 
work in progress, it can be installed by running:

```
git clone https://github.com/statsmaths/dvt
cd dvt
python setup.py
```

The design of the package is loosely based on the popular spaCy Python package
for natural language processing. The package takes an input video file and
initially produces frame-level metadata about each frame in the video. This
output data is stored on disk in a JSON file. Rather than a one size fits all
approach, the package provides a number of independent frame annotators (such
as detecting dominant colors, faces, or changes between two frames). Users can
construct a pipeline to process the video file by subsequently adding each
annotator to the pipeline. For example, the following Python code sets up a
pipeline and runs it over the video file `input.mp4`.

```
import dvt

vp = dvt.video.VideoProcessor()
vp.load_annotator(dvt.frame.DiffFrameAnnotator())
vp.load_annotator(dvt.frame.FlowFrameAnnotator())
vp.load_annotator(dvt.frame.ObjectCocoFrameAnnotator())

vp.setup_input(video_path="input.mp4",
               output_path="frame_output.json")
vp.process()
```

And the output will include rows of JSON data, such as the following:

```
{
  "video": "input",
  "type": "frame",
  "frame": 6997,
  "time": 233.5,
  "diff": {
    "decile": [0, 0, 1, 2, 85, 244, 247, 250, 251, 253, 255],
    "grad": [65431, 756182, 215187]
  },
  "oflow": {
    "t2187": [239, 109],
    "t2193": [476, 262],
    "t2194": [277, 139],
    "t2198": [234, 371],
    "t2200": [9, 355],
    "t2202": [528, 261],
    "t2204": [213, 288],
    "t2206": [260, 363],
    "t2207": [10, 277],
    "t2212": [511, 313],
    "t2213": [165, 362],
    "t2219": [175, 355]
  },
  "object": [{
    "box": {
      "top": 241,
      "bottom": 395,
      "left": 428,
      "right": 611
    },
    "score": 0.618,
    "class": "oven"
  }],
}
```

The last element here, for example, shows that the object detector has
identified a possible "oven" in the frame.

### A VideoViewer to Visualize Extracted Metadata

After extracting the metadata, we very quickly wanted to visualize the
extracted metadata. For one thing, this serves as a great visual check that
everything is working well. It is also visually interesting in its own right.
To this end, we built the `dvt.view.VideoViewer` class, which can be used as 
follows:

```
import dvt

vv = dvt.view.VideoViewer()
vv.setup_input(video_path="input.mp4",
               input_path="frame_output.json")
vv.run()
```

It plays the input video along with the identified metadata. Here is screen  
shot showing what this looks like when applied to an episode of *Bewitched*:

![VideoViewer Example 1](https://statsmaths.github.io/blog/assets/2018-03-14-dtv-pipeline/img01.png)

It should be straightforward to save these images as an output video file, but
we have not yet implemented this in the pipeline. When this is available, we 
will include a link on the main Distant Viewing page.

### Frame Annotators

Once we moved to using Python as the language for writing the library, we found
that transferring our various initial frame-level annotation attempts into the
library was relatively easy. Currently available frame annotators are:

- `DiffFrameAnnotator`: determines how different one frame is compared to the
previous frame. Specifically, it down samples the frame to a 32x32 thumbnail
and finds quantiles of the differences in hue, saturation, and value.
- `FlowFrameAnnotator`: follows key-points (points at the corners of detected
edges) across frames using optical flow. Looking at them over time allows for
the analysis of  object and character movement.
- `ObjectCocoFrameAnnotator`: uses the YOLOv2 algorithm to detect and localize
80 classes of objects in the frame. 
- `FaceFrameAnnotator`: detects, localizes, and computes 128-dimensional 
embeddings of faces in the image. It uses a CNN model which is relatively 
robust to scale and 3D-rotations.
- `HistogramFrameAnnotator`: compute histograms of hue, saturation, and values
of the image. Currently it is applied to each sector of a 3x3 grid over the image;
these can be aggregated to have a histogram of the entire image. Also estimates
and outputs the lumosity center of gravity, which can be useful for shot detection.
- `KerasFrameAnnotator`: an annotator that applies any image-based keras model and
output the numeric results. Save the model as an `.h5` model and load it into this
layer. Allows for the inclusion of a preprocessing function and will automatically
resize the image depending on the input shape given in the model.
- `TerminateFrameAnnotator`: a special annotator that can conditionally 
terminate the processing of a given frame. Put this in the pipeline between
fast annotators (e.g., the diff and flow annotators) that need to run on every
frame and slower annotators (e.g., object detection) that can be selectively
applied to only a subset of the frames.
- `PngFrameAnnotator`: another special annotator that saves the current frame
as a PNG file. Pair with the Terminate annotator to only save some of the images.

Currently these annotators have many hard-coded tuning parameters. As the 
toolkit is built out, we plan to document these and allow for tweaking these
at runtime. We also plan to include frame annotators to extract dominant
colors and to embed frames into a space useful for scene classification.

### Next Steps

The next stage of development involves producing video-level annotators. These
will take the `frame_output.json` file as an input and produce higher-level
annotations of the video as a whole. Tasks include:

- shot segmentation and classification
- scene segmentation and clustering
- facial recognition
- camera movement classification
- speaker resolution

This is where the real hard work starts to come in because these models will 
have to be trained and evaluated by us rather than simply calling a library
that already exists (at least, in most cases). We'll follow-up on the
development of these in future posts.








