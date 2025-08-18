
### Mosaicing under canopy GoPro images in Agisoft Metashape  

Author of Tutorial: Arvin Fakhri

Dataset collected by: Miriam Herrmann, Marius Derenthal, Ephraim Schmidt-Riese

#### Overview

This tutorial provides a comprehensive, step-by-step guide for processing under-canopy forest imagery in Agisoft Metashape (v2.0.3 or later). We'll focus on a power-user workflow that emphasizes manual control and iterative refinement to achieve the highest possible accuracy, which is crucial for complex environments like forests.

The tutorial uses under-canopy images collected with a GoPro camera mounted to a telescope stick within a forest stand. Most of the explanations are also valid for other datasets such as those collected by drones or airplanes.

The tutorials deviates from the standard processing chain offered by Metashape itself and summarizes several years of working experience with the software.

#### Overview dataset
In this tutorial we will use a dataset consisting of 449 GoPro images collected in a forest stand in the Czech Republic during the field season in 2024. The GoPro camera is a GoPro HERO 9 Black. The cameras were taken from a height of around 4-5 m by walking through the forest stand with telescope stick to which the GoPro camera was mounted to. The survey was conducted in a field plot sized 30 x 30 m by walking parallel lines with a high overlap in forward direction but also between parallel lines. The surveyor always walked on in one direction on a 30 m line, then turned 90° at the end of the 30 m walked approximately 5 m and then turned another 90° to walk back a parallel line. The example dataset was conducted during overcast conditions, which minimized the effect of different illumination conditions. That is, the dataset is comparably easy to process.

You can find the example dataset here:

Now let's start with the processing.

#### Project setup: loading images and initial settings

First, we need to create a project, load our images, and configure some essential settings before any processing begins.

**Loading the images**

1.  **Launch Agisoft Metashape.** You'll see an empty workspace with a "Chunk 1" panel.
2.  **Save your project.** Before adding any data, it's critical to save your project. Go to **File > Save** and choose a location. This ensures all your project files and processing results are stored properly from the start.
3.  **Add photos.** The easiest way to load your images is to select all of them in your file explorer and drag-and-drop them directly into the "Chunk" panel in the Metashape workspace. Alternatively, you can right-click on the chunk and select **Add > Add Photos...**. After loading, the chunk will update to show the number of images (e.g., "Chunk 1 (464 images)").

**The "Duplicate" strategy**

For complex projects, a highly recommended best practice is to perform each major processing step in a **separate, duplicated chunk**. While this uses more disk space, it provides two huge advantages:

-   **Organization:** Each stage of your project (alignment, optimization, dense cloud, etc.) is clearly separated.
-   **Error recovery:** If you make a mistake or want to change a parameter, you don't need to re-run the entire workflow. You can simply go back to the previous chunk, duplicate it, and start again from that point.

Let's start by renaming our first chunk. Right-click on "Chunk 1" and select **Rename**. Name it **"Images"**. This will be our pristine, unprocessed set of photos.

Figure 1
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_01.png)

**Camera calibration settings**

Before aligning, we must tell Metashape about the specific characteristics of our camera. Since this tutorial uses GoPro data, which has a very wide field of view, we need to specify a fisheye camera model.

1.  Go to the main menu and select **Tools > Camera Calibration...**.
2.  In the "Camera Calibration" window, Metashape will have automatically grouped the images by the camera model detected in the image metadata (EXIF).
3.  Select your camera model. Change the **Camera Type** from "Frame" to **"Fisheye"**.

Figure 2
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_02.png)


Figure 2: Metashape will have pre-filled parameters like focal length (f) and pixel size from the metadata. The other interior orientation parameters (IOPs), such as principal point offset (cx​, cy​) and distortion coefficients (k1​,k2​,k3​,k4​,p1​,p2​,b1​,b2​), will be empty under the "Initial" tab. These will be calculated during the alignment and optimization steps and will appear under the "Adjusted" tab.



**Reference settings**

Here, we define the coordinate systems and accuracy of our input data. This is crucial for georeferenced projects.

1.  Right-click on the **"Images"** chunk and select **Reference Settings...**.

Figure 3: The reference setting provides very important information.
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_03.png)


2.  **Camera Accuracy (m):** The images may contain GPS data. However, GPS signals under a dense forest canopy are notoriously unreliable. The default camera accuracy of 10 meters is a reasonable starting point for this low-quality GPS data.
3.  **Marker Accuracy (m):** This setting defines the accuracy of any Ground Control Points (GCPs) you might use. The default is 0.005 m (5 mm), which is extremely high and typically only achievable with a total station. If you used a survey-grade dGPS/GNSS, you might have an accuracy of 2-5 cm (0.02 - 0.05 m). Always set this value to a realistic estimate of your GCP accuracy. If you don't, Metashape may incorrectly distrust your high-quality GCPs. For now, we will not use markers.
4.   **Coordinate Systems:** Ensure the camera reference and marker reference coordinate systems are correctly set. If your camera geotags are in WGS84 and your markers are in a local UTM zone, you must specify both correctly so Metashape can transform everything into a single project coordinate system.


Figure 4: The reference setting menu
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_04.png)



**2. Aligning the Images**

Image alignment is the foundational step where Metashape finds matching features across photos, calculates each camera's position and orientation (Exterior Orientation Parameters, or EOPs), and generates a preliminary sparse point cloud.

1.  **Duplicate the chunk.** Right-click on the **"Images"** chunk, select **Duplicate**, and rename the new chunk to **"Alignment"**.
2.  Right-click on the **"Alignment"** chunk and navigate to **Process > Align Photos**.


Figure 5: How to align images.
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_05.png)



3.  In the "Align Photos" dialog, we will use specific settings designed for maximum data retention, which we will refine later.

**Accuracy:** We set this to High. This setting determines the image scale for feature detection. "High" uses the original, full-resolution images. While computationally intensive, it provides the most potential tie points for complex scenes. Avoid "Highest" unless necessary.

**Generic preselection:** **Checked**.

**Reference preselection:** **Unchecked** (unless you have highly accurate camera GPS and want to speed up alignment).

**Key point limit:** Set this to 0. This tells Metashape not to cap the number of features it detects on each photo.

**Tie point limit:** Set this to 0. This tells Metashape not to limit the number of matching points it uses to create the sparse cloud.

By setting the limits to 0, we prevent Metashape from prematurely discarding potentially useful points. This gives us full control over the filtering process later, which is key to our high-accuracy workflow.


Figure 6: The suggested settings for the alignment.
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_06.png)


Click **"OK"** and let the process run. This may take a significant amount of time.

**Troubleshooting alignment failures**

If some cameras fail to align, they will appear as unchecked in the "Cameras" list. Try these strategies:

1.  **Realign with different accuracy:** Duplicate your "Images" chunk again and try aligning with different accuracy. Sometimes this helps create a baseline alignment that can be improved upon later.
2.  **Manually place markers:** Identify a few clear, static features that are visible in both an aligned and an unaligned image. Right-click in the image and "Add Marker". Place the marker on the same feature in at least 2-3 aligned images and in the unaligned image. Then, select the unaligned cameras, right-click, and choose "Align Selected Cameras".
3.  **Disable/remove problematic images:** Carefully inspect the unaligned images. Are they blurry from motion or rolling shutter effect? Is there a moving object (like a person or animal or a car) dominating the frame? Sometimes, removing these poor-quality images is the best/only solution.



**3. Optimization**

This is the most critical stage for ensuring the final accuracy of your model. Our goal is to iteratively clean the sparse point cloud, removing low-quality tie points, and then optimizing the camera parameters based on only the high-quality points.

1.  **Duplicate the chunk.** Right-click on **"Alignment"**, select **Duplicate**, and rename the new chunk to **"Optimized"**. All the following steps will be performed on this chunk.

**Step 1: Initial manual cleaning**

Before using automated tools, use your knowledge of the scene to perform a rough cleaning.

1.  **Manual cleaning (visual inspection):** Rotate the sparse cloud and look for obvious errors. Use your knowledge of the surveyed area to identify and remove erroneous points.

-   Are there points floating high above the treetops or deep below the forest floor?
-   Use the selection tools (**Rectangle Selection**, **Free-Form Selection**) to highlight these obviously incorrect points and press the **Delete** key to remove them.
-   **Tip for fisheye imagery:** Points on the very edges of a fisheye image often have high uncertainty. A useful technique is to select the clean, central area of the point cloud, then go to **Edit > Invert Selection**. This highlights the noisy peripheral points, which you can then inspect and delete.


Figure 7: Manual refinement of the sparse point cloud by removing points located in marginal areas.
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_07.png)

2.  **Filter by image count (structural removal):** Tie points that are only visible in two images are geometrically weak. Their 3D position is calculated with zero degrees of freedom, meaning their accuracy cannot be reliably estimated. While some of these points may be correct, this group also contains many points with large, undetectable errors. It's a standard best practice to remove them early in the process.

Go to **Model > Gradual Selection...**.


Figure 8: Sparse point cloud refinement using gradual selection tool
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_08.png)

Criterion: **Image count**.

Level: Set the value to **2**. This will select all points that are visible in two images or fewer.


Figure 9: Gradual selection menu
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_09.png)

Click **OK**. The selected points will turn red.

Press the **Delete** key to remove the selected points.

After this initial cleaning, we are ready to optimize the camera alignment for the first time. Go to **Optimize Cameras...**. Check the parameters you wish to fit (e.g., **f, cx, cy, k1, k2, k3, p1, p2**) and click **OK**.

Figure 10: Camera optimization
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_10.png)


**Step 2: Iterative filtering and optimization (the refinement loop)**

Now we begin a systematic loop based on statistical quality metrics: **Filter -> Delete -> Optimize**. With each iteration, we will remove the weakest remaining points and then re-optimize the camera alignment to improve the accuracy of the entire solution.

**Iteration 1: remove points with high reprojection error**

1.  **Filter:** Go to **Model > Gradual Selection...**.

Criterion: **Reprojection error**.

Level: Drag the slider to a starting value like **0.8**. This criterion measures how accurately the 3D point projects back onto the original 2D images (a lower value is better). Again, adjust the level to remove a reasonable number of the worst remaining points. Click **OK**.

2.  **Delete:** Press the **Delete** key.
3.  **Optimize:** Go to **Tools > Optimize Cameras...** and click **OK**.

**Iteration 2: remove points with high Reconstruction Uncertainty**

1.  **Filter:** Go to **Model > Gradual Selection...**.

Criterion: **Reconstruction uncertainty**.

Level: Drag the slider to select a starting value, like **50**. This value represents the uncertainty of a point's 3D position. Watch the "Selected points" count and aim to remove a small fraction (e.g., 5-15%) of the remaining points. Adjust the level until you reach your target. Click **OK**.

2.  **Delete:** Press the **Delete** key.
3.  **Optimize:** Go to **Tools > Optimize Cameras...** and click **OK** to re-run the optimization with the default parameters.

**Iteration 3: remove points with low Projection accuracy**

1.  **Filter:** Go to **Model > Gradual Selection...**.

Criterion: **Projection accuracy**.

Level: Drag the slider to select a starting value, like **50** (A high value for this criterion simply means the point was localized on lower-resolution versions of the images, which can make its position less precise), then Click **OK**.

2.  **Delete:** Press the **Delete** key.
3.  **Optimize:** Go to **Tools > Optimize Cameras...** and click **OK** to re-run the optimization with the default parameters.

You can repeat this cycle, until you are satisfied with the cloud's cleanliness and the error metrics reported in the Reference pane.

Since this process can be quite boring to run manually, you can also automate these steps by running a Python script with a GUI, available here: wwwwww


Figure 11: To use the developed GUI, first load it through the "Run Script" menu (a), after which a new tab will appear (b). The program consists of two main parts (c): the first includes checkboxes to select which parameters should be included in the calibration model (IOPs), and the second displays the target value for each criterion.
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_11.png)


**Step 3: Final camera parameter check**

After you have finished cleaning the point cloud, it's wise to check for high correlations between your camera lens distortion parameters. This ensures your camera model is stable.

1.  Go to **Tools > Camera Calibration...**. Click on the **"Adjusted"** tab to see the final computed values.
2.  Right-click on your camera model (e.g., "Hero9 Black") and select **Distortion Plots...**.


Figure 12: distortion details
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_12.png)

3.  Select the **Correlations** tab. You will see a matrix of values.
4.  Look for any pairs with a high correlation, typically shown in red (e.g., > 0.8). For fisheye lenses, it's common to see high correlation between k3 and k4.


Figure 13: correlation matrix
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_13.png)

5.  If you find a high correlation, it means the parameters are redundant. You should disable the one with the higher error estimate. Go back to **Tools > Optimize Cameras...**, uncheck the box for that parameter (e.g., uncheck Fit k4), and run the optimization one final time.

The sparse point cloud is now clean, robust, and ready for building the dense cloud.

Figure 14: Refined sparse point cloud
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_14.png)


**4. Assessing model quality**

Before building the dense cloud, check the quality of your refined alignment. In the **Reference** pane, look at the "Error" columns. The overall Root Mean Square Error (RMSE) for your cameras and markers should be low. For a high-quality project without GCPs, a total camera error of **0.3-0.5 pixels** is a good target. Visually inspect the sparse cloud one last time for any "doming" or warping effects.


**5. Create dense point cloud**

The dense cloud adds millions more points, creating the detailed foundation for the mesh.

1.  **Duplicate the Chunk.** Right-click **"Optimized"**, duplicate it, and rename the new chunk **"DenseCloud"**.
2.  Go to **Build Dense Cloud...**.


Figure 15: dense cloud
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_15.png)


**Quality:** **Medium** or **High** is recommended. "High" quality is more time-consuming but produces a more detailed cloud. **Never select Ultra High that in 99% of cases it causes Agisoft to crash, or you’ll be waiting for ages.**

**Depth filtering:** This is crucial for vegetation. **Aggressive** filtering can remove too much foliage, while **Disabled** can leave a lot of noise. **Mild** or **Moderate** is usually the best choice for forest scenes.

**Calculate point colors:** **Checked**.


Figure 16: Point cloud settings
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_16.png)


3.  Click **OK**. This is the most computationally intensive step in the entire process.

Figure 17: Dense point cloud
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_17.png)


**6. Create Mesh and Texture**

The mesh creates a solid, continuous surface from the dense cloud, and the texture wraps it in a realistic skin from the original photos.

1.  **Duplicate the chunk.** Right-click **"DenseCloud"**, duplicate it, and rename it **"MeshTexture"**.
2.  Go to **Build Mesh...**.


Figure 18: Build mesh menu
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_18.png)

**Source data:** **Depth maps**.

**Surface type:** **Arbitrary (3D)**. This is essential for complex, non-terrain objects like forests.

**Face count and Quality:** **High** is a good starting point. This determines the geometric complexity of the mesh.

**Interpolation:** **Enabled (default)**.

Figure 19: Mesh settings
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_19.png)


3.  Click **OK**.
4.  After the mesh is built, go to **Build Texture...**.


Figure 20: Texture generation
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_20.png)


**Mapping mode:** **Generic**.

**Blending mode:** **Mosaic (default)**.

**Texture size/count:** A good balance of quality and performance is a single texture of **8192 x 8192** pixels. You can increase this (e.g., 16384) or use multiple textures for higher resolution, but it will increase file size.


Figure 21: texture settings
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_21.png)



5.  Click **OK**. You can now view your fully textured 3D model!


Figure 22: textured mesh
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_22.png)


**7. Generate project outputs (DSM, DTM, Orthomosaic)**

The final step is to generate standard products from your model.

**Create DSM (Digital Surface Model)**

The DSM is a raster map of elevation that includes the top of all features (canopy, ground, etc.).

1.  Go to **Build DEM...**. (DEM in Metashape can create both DSM and DTM).


Figure 23: DTM/DSM menu
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_23.png)


2.  **Source data:** **Dense cloud** or **Mesh**. Mesh is often faster if already built.
3.  Define your coordinate system and resolution. If classification is not required using **Depth maps** as the source data is a recommended alternative. Click **OK**. This product is your DSM.


Figure 24: DEM settings for DSM generation
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_24.png)

Figure 25: DSM
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_25.png)


**Create DTM (Digital Terrain Model)**

The DTM represents the bare-earth elevation. Creating it requires an extra step to classify the ground points.

1.  First, classify the ground points in your dense cloud. Go to **Tools > Point Cloud > Classify Ground Points...**. Use the default settings first and adjust if needed.

Figure 26: point cloud classification to create DTM
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_26.png)

2.  Now, go to **Build DEM...**.
3.  In the "Build DEM" dialog, under "Point classes", make sure to select **only Ground**. De-select all other classes (like vegetation, buildings, etc.).
4.  Click **OK**. This new product is your DTM.

Figure 27: DSM
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_27.png)

**Create Orthomosaic**

The orthomosaic is a geometrically-correct, top-down view of your model.

1.  Go to **Build Orthomosaic...**.


Figure 28: Build Orthomosaic… menu
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_28.png)

2.  Select the surface you want to drape it on (usually the **DEM/DTM** for terrain or **Mesh** for models).


Figure 29: Orthomosaic settings
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_29.png)

3.  Define the resolution and other parameters. Click **OK**.

Figure 30: Orthomosaic
![Figure 1](https://github.com/fabianfassnacht/Metashape_Mosaic_creation_gradual_selection/blob/main/Metashape_30.png)


**8. Generate Report**

Finally, document your work. A report is essential for quality control and for sharing your methodology.

1.  Go to **File > Export > Generate Report...**.


Figure 30: Creating the report of the project

2.  Choose a file path and click **Save**.

This PDF report contains all the vital statistics of your project: processing parameters, camera calibration results, camera/GCP errors (RMSE), and the number of points at each stage. It serves as the final proof of your model's quality and the rigor of your workflow.


**GCPs and CPs**

Before starting, it's important to know the difference between the two types of points:

-   **Ground Control Points (GCPs):** These are markers with known, high-accuracy real-world coordinates. You use them _during_ the camera optimization process to scale, orient, and position your model correctly in geographic space. They are the "control" for your project.
-   **Check Points (CPs):** These are also markers with known coordinates, but they are _withheld_ from the optimization process. Their purpose is to provide an independent, unbiased validation of the final model's accuracy. By comparing the model-derived coordinates of the CPs to their known real-world coordinates, you can verify how accurate your project truly is.

**Step 1: Place markers in images**

The first step is to tell Metashape where your physical targets are located in the images.

1.  **Create a marker:** In the **Photos** pane, find an image that clearly shows one of your ground targets. Right-click directly on the center of the target and select **Add Marker**. A new marker flag will appear.
2.  **Rename the marker:** In the **Reference** pane on the left, a new marker (e.g., "point 1") will appear. It's good practice to rename it to match the label from your field survey (e.g., "GCP-01").
3.  **Refine placement:** A single placement isn't enough. You need to accurately place the same marker in multiple photos.

-   In the **Reference** pane, right-click on your new marker and select **Filter Photos by Markers**. The Photos pane will now only show images where Metashape thinks the marker is visible.
-   Go through at least 3-5 of these photos. For each one, zoom in and drag the marker flag so its crosshair is precisely on the center of the target. The more photos you place a marker in, and the more precise your placement, the better your results will be.

Repeat this process for all your GCPs and CPs.

**Step 2: Import marker coordinates**

Once all markers are placed, import their known geographic coordinates from your survey. You can either manually edit the default coordinates of each marker or load a text file containing the name and coordinates of each point.

1.  **Open the import tool:** In the **Reference** pane, click the **Import Reference** icon on the toolbar.
2.  **Configure the import:** In the "Import CSV" dialog box:

-   **Coordinate system:** This is critical. Select the exact coordinate system your survey data was collected in (e.g., WGS 84, or a specific UTM zone like UTM zone 32N).
-   **Delimiter:** Choose the character that separates the columns in your file (e.g., Comma, Tab, Space).
-   **Column assignment:** Tell Metashape which column in your file corresponds to which data type. Make sure to assign the **Label**, **Easting/X coordinate**, **Northing/Y coordinate**, and **Altitude/Z coordinate** correctly.
-   Click **OK**. The "Source" coordinate columns in the Reference pane will now be populated with your imported data.

**Step 3: Set marker types (GCP vs. CP)**

This is where you tell Metashape which points to use for control and which to use for checking. You should aim to use about 75% of your points as GCPs and 25% as CPs, spread evenly throughout your project area. When I don’t have enough points, I usually follow the LOOCV strategy.

1.  **In the Reference pane, ensure you can see the "Accuracy" columns.** If not, right-click the header row and check them.
2.  **To define a GCP:** For each marker you want to use as a Ground Control Point, leave the checkbox next to its name **checked**. Then, click into its **Marker Accuracy (m)** cell and enter a realistic accuracy value for your survey equipment (e.g., 0.02 for 2cm accuracy from a dGPS/GNSS).
3.  **To define a CP:** For each marker you want to use as a Check Point, simply **uncheck** the checkbox next to its name. This removes it from the optimization calculations, turning it into a passive verification point.

**Step 4: Re-optimize the camera alignment**

Now that Metashape knows the location of your GCPs, you must re-run the optimization to align the model to them.






