DeSTIN
=============

Papers about DeSTIN
-------------------
* http://web.eecs.utk.edu/~itamar/Papers/BICA2009.pdf
* http://www.ece.utk.edu/~itamar/Papers/BICA2011T.pdf
* http://web.eecs.utk.edu/~itamar/Papers/AI_MAG_2011.pdf
* http://research.microsoft.com/en-us/um/people/dongyu/nips2009/papers/Arel-DeSTIN_NIPS%20tpk2.pdf
* http://goertzel.org/Goertzel_AAAI11.pdf
* http://goertzel.org/DeSTIN_OpenCog_paper_v1.pdf
* http://goertzel.org/papers/DeSTIN_OpenCog_paper_v2.pdf
* http://www.springerlink.com/content/264p486742666751/fulltext.pdf
* http://goertzel.org/VisualAttention_AGI_11.pdf
* http://goertzel.org/Uniform_DeSTIN_paper.pdf
* http://goertzel.org/papers/Uniform_DeSTIN_paper_v2.pdf
* http://goertzel.org/papers/CogPrime_Overview_Paper.pdf

Current Work
------------
Implementing uniform destin described here:
http://wiki.opencog.org/w/DestinOpenCog

Creating enhanced visualization capabilities.

For more detailed development progress see Diary.md

Building
--------
Instructions below are for Ubuntu. 

See Windows build instructions at https://github.com/opencog/destin/blob/master/WindowsBuild.md

Dependencies:

* CMake (>= 2.8)
* OpenCV 2 (libopencv-dev)

Java Bindings: JDK, ant and SWIG 2.x

Python Bindings: python-dev (tested with python 2.7), python-opencv, and SWIG 2.x

To build DrentheDestin, CUDA SDK is required. 

Individual parts can be skipped by commenting out their ADD_SUBDIRECTORY line in the CMakeLists.txt
Some examples:

* If you dont have CUDA for example, comment out the "ADD_SUBDIRECTORY(DrentheDestin)" in Destin/CMakeLists.txt

* If you dont want to install any language bindings comment out "ADD_SUBDIRECTORY(Bindings)" in Destin/CMakeLists.txt.

* If you want python bindings but not Java, then in Destin/Bindings/CMakeLists.txt comment out "add_subdirectory(Java)"

Building:

    $ git clone http://github.com/opencog/destin.git
    $ cd destin
    $ git submodule init
    $ git submodule update
    $ cd Destin
    $ cmake . 
    $ make
    
Rebuilding:
    
During development sometimes CMake is not able to detect dependencies between the different built libraries. So it is recommened
to do a "make clean" and "make -j4" (j4 lets you use multiple cores) from the Destin directory to make sure everyting is building fresh.

Building Java Bindings:

The native jni bindings lib is built with CMake, the actual java program which uses it is built with java_build.sh:

    $ cd Destin/Bindings/Java
    $ ./java_build.sh
    $ ./java_run.sh
    

Show me something!
------------------

Currently the most interesting things to see are the scripts in the Destin/Bindings/Python directory. 
These scripts are meant to be used interactivly so I recommend installing and using idle for python ( search for and install the idle package).

To see a self organizing map (SOM):

Manually download http://www.cs.toronto.edu/~kriz/cifar-10-binary.tar.gz

    $ tar xf cifar-10-binary.tar.gz
    $ cd cifar-10-batches-bin
    $ pwd

Note the absolute path 

    $ cd Destin/Bindings/Python
    
Edit som.py and edit the cifar_dir variable near the top and change it to the absolute path of the cifar-10-batches-bin directory

    $ idle som.py

Two windows appear, select the som.py window and click Run->Run Module (F5).
Training will begin, after less than 5 minutes the SOM will popup.
    
To see DeSTIN train on an video and see its output:
    
    $ cd Destin/Bindings/Python
    $ idle dostuff.py
    
Two windows should appear.

    - select the dostuff.py window
    - click Run -> Run Modedule ( F5 )

You should see a video of a hand and windows for 8 layers.
    
In the other "*Python Shell*" window you can interact with the code after its done with 500 frames ( about a minute).
To make it continue processing type go(100) to have it process 100 more frames. You can make it train on webcam input instead by
changing the line: 

    vs = pd.VideoSource(False, "hand.m4v")
    
to 

    vs = pd.VideoSource(True, "")
    
    
Here is my final documentation of the GSOC project.

Task:
Implementation of scale invariance in Uniform Destin.

Approach:

The approach I took is largely similiar to the implementation of translational invariance in Uniform Destin. 
However unlike translational invariance I had added centroids at the levels to be rescaled as well.

What I had done before midterm evaluation:

The function AddRescaledCentroids in centroid.c adds the rescaled centroids for each node in the destination layer into the centroid pool for layer. My implementation is similar to the implementation of AddUniformCentroid in centroid.c. My function basically allocates space for extra centroids and their dimensions and appends it to uniform mu (destin->uf_mu) and updates other parameters as well such as sigma,absolute variance etc.
I have managed to implement the deletion of duplicate or redundant centroids in the function doDestinwithRescaling in DestinNetworkAlt.cpp. Basically it removes additional or overlapping centroids with each added iteration. For each iteration an extra rescaled centroid is added to each node in the destination layer and replaced on subsequent iterations.
The function  doDestinwithRescaling acts like a main function and calls the AddRescaledCentroids function for each node and each centroid in the source layer.This function can be modified according to specific needs and constraints.

What I have done after midterms:

•	Implemented rescaling at all levels for all iterations above a certain threshold. Basically this is similar to the way uniform centroids were added at each level.                                              
In the file node.c I have added a function Uniform_addRescale which takes as input the destination layer and source layer and uniformly incorporates the rescaled centroids into the centroid pool at that layer.
This function is called for all layers below the destination layer. The function is similar to add_Uniform function in node.c.

•	I have balanced the adding of Uniform(translation invariant) centroids as well as the rescaled centroids. This required a lot of trial and error based testing to check most optimal arrangement when uniform and rescaled centroids needed to be added so that the centroid pool is cleaner and there won’t be mix-up of features.


Core Functionality:
Function doDestinwithRescale  is the main function in DestinNetworkAlt.cpp which basically calls all the rescaling processes.  It has arguments which specify max number of iterations, number of iterations till which rescaling is applied etc. It calls the function Uniform_addRescaledCentroids which is the function which adds the rescaled centroids to corresponding nodes.

The function Uniform_addRescaledCentroids takes as input the destination and source layers. For each centroid in each node of the destination layer, that centroid is rescaled from source to destination layer and added to the centroid pool for that layer. It adds a centroid for every node in that layer and this extra rescaled centroid is shown in the centroid image as the rescaled image of the source layer.
The function add_Rescaledcentroids provides the functionality for the actual rescaling of centroids which is passed by the Uniform_addRescaledCentroid function mentioned above. For each centroid it calls the rescale_centroid function which returns a vector which corresponds to the value for the rescaled centroid.

Results:
I have tested my entire implementation on letter_experiments.py which essentially enables recognition of handwritten letters. 
The results are promising although despite exhaustive testing I haven’t been able to achieve a perfect accuracy between the rescaled images and the original images. Through various heuristics l have been able to improve upon the similarities between the original and rescaled images considerably compared to during the midterm evaluations mostly due to the addition of centroids during several iterations.

