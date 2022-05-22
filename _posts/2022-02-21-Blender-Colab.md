
---
title: How to Run Blender on Google Colab with a Free GPU
image: 
  path: /assets/images/GoogleBlenderPost.png
  thumbnail: /assets/images/GoogleBlenderPost.png
  caption: "Google Colab and Blender"
---



This is a more comprehensive set of instructions on how to render your Blender scenes on Google Colab. All of these tutorials can be found on my [Youtube Page here](https://www.youtube.com/c/MicroSingularity) with the playlist to all Colab tutorials [here](https://youtube.com/playlist?list=PLV0Zsi5ZYUasqIKTYo7et4iJuaSIArSy5). If you would like a more detailed explanation about each tutorial before watching, I have a blog post [here](https://microsingularity.github.io/blog/Google-Colab) summarising each video.
The notebook files for this code can be [found here](https://github.com/microSingularity/blenderGoogleGPU)

### Check you have a GPU on Google Colab

To make sure you are rendering on Google Colab with a GPU and not just a CPU, run the following line of code. It will display details about the GPU you are using. In particualar, pay attention to the name of the GPU. It should be something like Tesla K80 or simmilar. 

```
!nvidia-smi
```


### Link Google Drive to Google Colab

First, run the following two commands in Google Colab to connect your Google Drive. This will allow you to save your Blender files and renders on Google Drive so they can be retained after your Colab instance closes. 

```
from google.colab import drive
drive.mount('/content/drive')
```
### Download Blender to Google Colab

If you need to use another version of Blender, go to the [Blender website here](https://download.blender.org/release/). Choose the folder of the version you want to download

![List of all Blender Releases](/assets/images/BlenderVersions.png)

Then make sure you find the linux version, and then right click and copy link. This link can then be copied into the code below (if you don't want to use the Blender 3.0 release)

![List of all Blender Releases](/assets/images/Blender3.png)

This line of code dowloads Blender from the official Blender website and saves it in your working directory of Google Colab (not google drive)

```
!wget https://download.blender.org/release/Blender3.0/blender-3.0.0-linux-x64.tar.xz
```
### Save Blender to Google Drive (Optional)
Use these commands to move the Blender zip file you just downloaded to Google Drive so that you don't need to download Blender each time your run Google Colab
The first line of code copies the Blender tar file to Google Drive to a folder called 'Blender'. Make sure you change the filename to match your downloaded version of Blender.

For example, change ```blender-2.93.5-linux-x64.tar.xz``` to ```blender-3.00.0-linux-x64.tar.xz```, for example


This line of code copies blender from Google Colab to Google Drive so that it can be used again next time without downloading again. This only needs to be done once
```
!cp /content/blender-2.93.5-linux-x64.tar.xz /content/drive/MyDrive/Blender/blender-2.93.5-linux-x64.tar.xz
```

This line of code copies the Blender zip file back from Google Drive to Google Colab.
```
!cp /content/drive/MyDrive/Blender/blender-2.93.5-linux-x64.tar.xz /content/blender-2.93.5-linux-x64.tar.xz
```

### Unzip Blender

Once you have downloaded the right version of Blender, you need to unzip the tar file so that it can be executed. You can do this by typing the following command. 
Make sure you change ```blender-2.93.5-linux-x64.tar.xz``` to the correct version of Blender you downloaded earlier

```
!tar xf blender-2.93.5-linux-x64.tar.xz
```

### Fixing Library Errors
In 2021, Google appeared to have updated the default library files that come installed on a Google Colab instance. As a result, Blender failed to run on Google Colab.
This set of code below uninstalls the standard libraries and re-installs the versions needed for Blender

```
#Deletes the Default libtcmalloc-minimal4 version and installs the Ubuntu default version
import os

os.environ["LD_PRELOAD"] = ""

#Deletes wrong Version of libtcmalloc-minimal4
!apt remove libtcmalloc-minimal4
#Installs correct version of libtcmalloc-minimal4
!apt install libtcmalloc-minimal4

#Adds this library to the user environment
os.environ["LD_PRELOAD"] = "/usr/lib/x86_64-linux-gnu/libtcmalloc_minimal.so.4.3.0"
```
### Choosing which blender scene to Render. 

Before uploading your Blender scene to Google Colab, make sure you have configured all the render settings. For example, set the number of samples and resolution before uploading.
If you are using any HDRIs or textures in your Blender scene, be sure to package them into the Blender file.

![Package Textures](/assets/images/ExternalData.png)

Then upload the Blender scene to Google Drive to a folder of your choice. Then copy the path to the Blender scene (.blend file) into the line of code below. Here, my Blender file is in the root Google Drive directory.
```
filename = '/content/drive/MyDrive/BlenderColab.blend'
```

### Render your Blender Scene. 

Now that your Blender files have been uploaded to Google Drive and Blender has been downloaded, you are ready to render your Blender scene on Google Colab.

The first line of code here will render a single frame of your Blender scene using CUDA and save the file to ```/content/drive/MyDrive/Blender```, specified using the ```-o``` command. You can change this filename to whatever you like.
The ```debug-all``` comamnd is optional and just gives you extra infromation in case it doesn't render. The ```-f 1``` command tells Blender to render frame 1, but you can change the ```1``` to any frame you like. The ```-F 'PNG'``` tells Blender to output your render as a PNG file.

```
#!./blender-2.93.5-linux-x64/blender -b $filename -noaudio -E 'CYCLES' --debug-all -o "/content/drive/MyDrive/Blender" -f 1 -F 'PNG' -- --cycles-device CUDA
```

Optionally, you can change ```CUDA``` to ```OPTIX``` at the end of the line if you are given a GPU with OPTIX support. OPTIX is not supported on the Tesla K80 GPU.
```
!./blender-2.93.5-linux-x64/blender -b $filename -noaudio -E 'CYCLES' -o "/content/drive/MyDrive/Blender" -f 1 -F 'PNG' -- --cycles-device OPTIX
```
If you want to render an animation, use the following line of code.The animation will be saved in whatever format was chosen in blender file. i.e. MP4
The ```-s 1``` and ```-e 250``` commands tell Blender to start rendering the animation on frame one and end on frame 250.

```
!./blender-2.93.5-linux-x64/blender -b $filename -noaudio -E 'CYCLES' -o "/content/drive/MyDrive/Blender" -s 1 -e 250 -a -- --cycles-device CUDA
```


