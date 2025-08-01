### Creating image mosaics with AGISOFT Metashape using custom settings
#### Overview
This tutorial explains how to create a orthomosaic from under-canopy images collected with a GoPro camera mounted to a telescope stick within a forest stand. Most of the explanations are also valid for other datasets such as those collected by drones or airplanes.
The tutorials deviates from the standard processing chain offered by Metashape itself and summarizes several years of working experience with the software kindly shared with me by Dr. Arvin Fakhri. Arvin is also the author of the Python script that will be used in the tutorial below. Thanks a lot @Arvin for sharing your knowledge and experiences and for letting me putting this knowledge out there.

#### Overview dataset
In this tutorial we will use a dataset consisting of 449 GoPro images collected in a forest stand in the Czech Republic during the field season in 2024. The GoPro camera is a GoPro HERO 9 Black. The cameras were taken from a height of around 4-5 m by walking through the forest stand with telescope stick to which the GoPro camera was mounted to. The survey was conducted in a field plot sized 30 x 30 m by walking parallel lines with a high overlap in forward direction but also between parallel lines. The surveyor always walked on in one direction on a 30 m line, then turned 90° at the end of the 30 m walked approximately 5 m and then turned another 90° to walk back a parallel line. The example dataset was conducted during overcast conditions, which minimized the effect of different illumination conditions. That is, the dataset is comparably easy to process.

You can find the example dataset here:

Now let's start with the processing.

#### Loading the images and preparing the Metashape project

We first start AGISOFT Metashape from the start menu of the computer which should lead to the situation as shown in Figure 1. The easiest way to load the images is to drag & drop all the images into the "Chunk" area in the workspace as marked in red in Figure 1.
After this, the software should display  "**Chunk 1 (449 images**)" in workspace section.

![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_01.png)

Before we now start with the processing of the images, we need to make one important modification to the camera settings for the current project. For this we select:

Tools ⇒ Camera Calibration

This will open the window shown in Figure 2. As you can see, Metashape automatically detected the camera type from the metadata of the images and filled out some of the settings such as the focal length. Given that the GoPro camera we used to collect the data for this tutorial has a very wide view angle, it is recommended to switch the camera type to "fisheye" as highlighted in red in Figure 2.

![Figure 2](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_02.png)

Another global setting that should be adjusted in some cases at the beginning of the processing work-flow can be reached by performing a right-click on the Chunk 1 section in the workspace and then select "Reference Settings". In the window that pops up you can let Metashape know more about how precise the geolocation information that you provide to the software is. In our case, the GoPro images may have some GPS information included in the images. However, this GPS information below the canopy of a forest may not be very accurate. So the camera accuracy of 10 m which is standard setting seems reasonable. In case you used some markers (targets which are clearly visible in some images and where you know the location with very high reliability) during your survey, you can also set how accurate these are. Be careful, the standard setting is 0.5 cm. This is a very high accuracy that you might not even reach with a dGPS in forests. If you know that you use markers and that your markers have a lower accuracy, you should change this setting. Similarly, the marker accuracy on the right part of the window refers to the how accurate you have set manual markers in the image. Again - here an accuracy of half a pixel is very high and you might need to adjust this value. In our case we will not use any markers for now and hence do not need to make and changes.

![Figure 3](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_03.png)
![Figure 4](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_04.png)

Once we have completed these two steps, we are ready to start with the initial step of the processing pipeline of Metashape, which is aligning the images.


#### Aligning images

To align the images, we perform a right-click on the Chunk 1 section in the workspace and then select "Process ⇒ Align Photos" as shown in Figure 5. This will open up a new window. Here, we will now intentionally deviate from the standard settings of Metashape.

![Figure 5](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_05.png)

As shown in Figure 6, we will set both the **Key point limit** and the **Tie point limit** to 0. This means that Metashape is allowed to search for an unlimited amount of key point and tie points. This will be important for the next step that will be explained with more details below.

![Figure 6](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_06.png)

In short: We try to come up with a very high number of potential points with which we can align the images but will then later gradually select only those points which are of high quality. This normally leads to better alignment results than the automated procedure offered by Metashape.

Another important point to consider here is the field **"Accuracy"**. Accuracy is actually not related to the accuracy of how well tie points are matching or something alike but rather relates to the geometric size of the window in which key-points are being searched. So the higher you set the accuracy, the smaller the search space will be. This is not necessarily leading to a better alignment between images and can even be counter-productive, depending on how your images look like and what kind of objects are found in your image.

In our case we select "very low" as accuracy, mostly to speed up the process in the tutorial. If you want to obtain the best possible results, it would be recommended to also try out other settings.

The other settings we leave as is for now and press **"OK**". This process is now likely to last quite a bit of time depending on your computer. If you want to speed up the process, you can try to select the option "low" or "very low" for accuracy.

If some of the images are not aligned after this processing step, you might have to set some tie points (they are then called "markers") manually. In my case I had to do this and it worked for some images but in other images it did not work and I simply dropped them in the end. 

How to set markers manually will be explained in the "Additional advice section" at the end of this tutorial. If you want to follow this process, you can jump to this section now and then return here afterwards. 

In my case, I ended up with successfully aligning 383 of the 449 images and I will not continue with the next steps using these aligned  images. I dropped the remaining ones which I did not manage to align.

#### Optimize camera alignment
The next step we now conduct is to optimize the camera alignment. This is again a step that is typically not done during the standard Metashape workflow. 

To get started we right-click on the Chunk 1 section in the Workspace and select "Process ⇒ Optimize Camera Alignment" as shown in Figure 7. Then a new window open as shown in Figure 8. Here, we can now define which parameters we try to optimize. Generally, the settings shown here at the moment work quite well but to explain the complete work-flow we for now select all parameters, that is, we also check the currently empty boxed **"Fitk4"**, **"Fitb1**" and **"Fitb2"**. We then press **"ok".**

![Figure 7](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_07.png)
![Figure 8](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_08.png)

Now we have to verify whether it was actually necessary to activate these additional parameter or whether they show high co-correlation with the other paramters. To verify this, we select:

Tools ⇒ Camera Calibration

the same window we already know from the first part of this Tutorial when we adjusted the setting to a fisheye camera pops up. If we click on the "Adjusted" section as shown in Figure 9, we can see that after running the camera calibration, these fields have been filled and all the parameters now have values. This would have not been the case at the beginning of the tutorial.

![Figure 9](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_09.png)

However, we now want to check whether tis ireally necessary to activate all of the parameters.

For this we now perform a right click on the "Hero9 Black (3mm)" camera on the top left and select "Distortion plots" as shown in Figure 10. In the new window that pops up as shown in Figure 11, we then we select the section "Correlations" which will show a window that looks as shown in Figure 12. 

![Figure 10](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_10.png)
![Figure 11](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_11.png)
![Figure 12](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_12.png)

If all parameters would contribute to the camera optimization, the correlations between the parameters should be below a certain user-defined threshold. In our case we can see that the l4 and k3 parameters are highly correlated (also with k2) and hence it makes no sense to keep them.

We therefore re-run the steps above but deactivate k4 and then after rechecking, I decided to also deactivate k3. which leads me to the final correlations view after the camera optimization as shown in Figure 13.

![Figure 13](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_13.png)

Once we have found a good subset of parameters that we want to use during the camera optimization procedure, we now start gradually decreasing the tiepoints that we have identified during the alignment step to just remain the highest quality time points before we create the dense point cloud.

#### Gradual selection of high quality tiepoints
The next step is to use various options (criteria) of Metashape to assess the quality of tie points and then select tiepoints of low quality and delete them. Everytime we deleted some tie points we re-run the camera optimization procedure and then follow up with the next criterion, select another subset of points, delete them and rerun the camera optimization.

In short, the procedure follows the following steps:

1. Select "Model ⇒ Gradual selection" this will open a new window as shown in Figure 14.
2. Select the option "Image count" which indicates in how many images the tie point was found - a value of 1 indicates in 1 additional image (in addition to the image where the tie point was originally set). The recommended value here is 2. Then we check how many tie points have been selected when selecting all points that have at least a value of 2 (see red marked area in Figure 14)
3. We re run the camera optimization by right-clicking Chunk 1 ⇒ Process ⇒ Camera optimization
4. We again select "Model ⇒ Gradual selection" but this time we use the criterion "Reprojection error" and we should select a value between 0.3 and 0.8 depending on how many tiepoints we still have left and how many we lose.
5. We re run the camera optimization by right-clicking Chunk 1 ⇒ Process ⇒ Camera optimization
6. We again select "Model ⇒ Gradual selection" but this time we use the criterion "Maximal uncertainty" and select a value between 5 and 20.
7. We re run the camera optimization by right-clicking Chunk 1 ⇒ Process ⇒ Camera optimization
8. We again select "Model ⇒ Gradual selection" but this time we use the criterion "maximal projection error" and select a value between 10 and 100.

If many tiepoints are left after this step, we can re-run the whole steps 1-6 several times.

Given that this is quite a bit of work to run manually, it is also possible to automatize this steps running a Python-script which you can access here:

ADD LINK TO PYTHON SCRIPT

To run the script...

#### Create dense point cloud

xxxxx

#### Create Orthomosaic
xxxxx

#### Additional advice

##### Setting manual markers

