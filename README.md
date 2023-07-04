# A method for evaluating the performance of perception algorithms based on a scenario simulation framework for autonomous driving applications

## A master's thesis in Software Engineering by the Faculty of Engineering of the University of Porto in conjunction with Bosch

### [Francisco Bastos](mailto:up202103393@up.pt), [Nuno Flores](mailto:nflores@fe.up.pt), and [Carlos Silva](mailto:carlos.silva6@pt.bosch.com)

## Table of Contents

1. [Introduction](#introduction)
2. [Related Work](#related_Work)
3. [Analysis and Design of the System](#analysis_and_design)
4. [Preliminary Results](#results)

<div id='introduction'/>

## Introduction

One of the biggest trends currently pushing forward research and development (R&D) in the automotive industry is trying to build the world's first truly autonomous self-driving vehicle. However, the scientists, engineers, and technicians trying to create one of these systems are constantly running into problems never addressed before in the history of mankind.

One of these particular issues is ensuring system safety without skyrocketing development costs. This master's thesis project aims to address this topic by developing a closed-loop, software-in-the-loop (SiL) environment representation verification and validation (V&V) pipeline that ensures a valid quality assurance (Q&A) process for the perception layer of autonomous driving (AD) systems without compromising the overall system's quality and development costs.  

This repository is a work product of that master's thesis. It focuses on the actual evaluation process of the proposed Q&A method. This process was also conducted using a new dissimilarity metric that enables a fast, accurate, robust, automated, and shape-preserving evaluation between two LiDAR point clouds. The metric has already shown potential to other interested stakeholders with easy adaptability to other use cases. Additionally, it has already identified some issues concerning the performance of the Bosch L4 Static World GT Generation tool map reconstruction capabilities.

<div id='related_Work'/>

## Related Work

This section aims to describe this project's related work. One of the goals of this master's thesis was also to find the best key performance indicators (KPIs) to evaluate the performance of a Bosch perception algorithm. The Bosch L4 Static World, GT Generation tool, was chosen among the many available perception algorithms. The tool is SLAM based. Therefore, the analysis points used to evaluate the performance of SLAM algorithms can also be used to evaluate the tool.

It is possible to evaluate the SLAM algorithm on its accuracy, consistency, efficiency, and robustness. Since this algorithm was already developed and tested, some of the possible areas of evaluation were also already conducted (namely efficiency). So the goal of this dissertation shifted from evaluating all of the possible areas to focusing on only evaluate the tool's accuracy, consistency, and robustness. Also, since a previous dissertation on this topic has already evaluated the SLAM's accuracy on its pose estimation capabilities, this dissertation will only focus on the accuracy of the map reconstruction capabilities of the SLAM algorithm - the consistency and robustness areas of evaluations were also addressed during this dissertation via the creation of various corner case (CC) scenarios that allowed the SLAM to be tried on the most diverse driving scenarios. However, those evaluation areas fall outside this repository's scope and will not be addressed here.

In order to evaluate the accuracy of the SLAM map reconstruction capabilities, a new dissimilarity metric was created, which enabled a quick comparison between the two (LiDAR in this particular case but could also be used for Radar, for example) point cloud maps. Regardless, to explain the metrics' inner workings, it is first required to give a little context as to why its creation was required.

At its core, most of the papers in the literature relative to this area orbit around the same general validation method. The first step of this method consists in collecting the reference and the data to be evaluated (from a GT generation algorithm, for example), then that data needs to be synthesized into a virtual testing environment, and lastly, there needs to be a comparison method that enables a fast and accurate comparison of both the reference and generated data. One of the ways in which it is possible to validate the data is by using the following generic validation method proposed by Oberkampf and Trucano, presented in the following figure:

![Example of a generic validation method according to Oberkampf and Trucano](/figures/7995752-fig-1-source-large.png?raw=true "Example of a generic validation method according to Oberkampf and Trucano")

The most critical part of the method is the comparison process itself. The paper *"Validating Simulation Environments for Automated Driving Systems Using 3D Object Comparison Metric"* by Wallace et al. contributes to the LiDAR simulation validation approach, using a *similarity metric* to compare two 3D point cloud maps. One generated through real-world LiDAR scans and another via a virtually simulated LiDAR sensor. This paper inspired the newly proposed metric, which will be explained in the following section.

This process's primary goal is to provide the ability to quantify how similar a map created by a virtual LiDAR sensor model is to the map generated by a real LiDAR. The unique factor of this metric is its generalization feature since it also allows the validation of other 3D objects for other perception sensors, such as cameras and radars.

The paper also summarizes the four main criteria used to select a method for 3D object comparison. The criteria are:

**1. Suitability for the data type:** The object is created from a LiDAR environment scan. This means they are large in scale and complex, are unlikely to be convex, and cannot be described through a single shape. This is generally different from the model created using computer-aided design (CAD) systems.

**2. Automated:** The method should function without manual interference or apriori model knowledge. This will allow the comparison of many pairs quickly and allow validation to be completed without human bias.

**3. Shape-preserving:** Ideally, the method should maintain the shapes' original structure without deformation. This is to avoid loss of relevant information and fidelity with the comparison.

**4. Practicality:** This mainly includes complexity, run-time, and robustness. It will handle large and complex objects quickly and without breaking even if the data differs from what is expected.

Also, according to the paper, the most common LiDAR point cloud comparison processes use one-to-one direct comparison methods, using the mean distance between corresponding points. The point comparison is either done on a point-by-point basis (Chamfer or Hausdorff distances) or by using a local model that takes a least squares average plane of all the points in the other cloud that are located within a fixed sphere around the original point and finds the shortest distance between this plane and the original point. These methods avoid discrepancies caused by individual missing points or calibration errors. However, their most significant disadvantage is requiring manual rotational alignment of the two point clouds to compare the corresponding points, which is an error-prone method.

Another two methods mentioned in the paper allow objects to be aligned automatically by identifying their critical points (using the object's curvatures) or using the inertia tensor as a principal axis. Although these methods show promising results, further work must be done to make them more reliable since a slight alignment discrepancy would significantly affect the similarity score.

Similar methods as those mentioned in the paper use an underlying surface created using a 3D Fisher vector called *DPDist* (Deep Point Cloud Distance). However, this method uses artificial intelligence (a neural network), which requires many sample models to train and is only applicable in some use cases. This method is also more suited to sparse point clouds, which are not the case. Other methods mentioned in the paper suggest decomposing a 3D object into many 2D silhouettes from different perspectives; the most similar silhouette pairs are identified between the two objects, and the dissimilarity is calculated and summed over all pairs. The dissimilarity measure between two 2D silhouettes is given by measuring the distortion of a bijection between the two images. This method works best with convex objects with many distinct silhouettes.

Other methods from the paper suggest point cloud comparisons between skeletal Reeb graphs. This method reduces the 3D object surface to a directed graph based on the distance from the center of mass. The similarity is calculated by finding the number of edges in both objects' most significant common sub-graph. However, this method is much more suited to CAD-generated 3D objects than those generated by point clouds.

Nevertheless, the method described by Wallace et al., which by itself uses a variation of the one proposed in the articles *"An Intelligent Method for Comparing Shapes of Three-Dimensional Objects"* by Muliukha et al., compares the distribution of pairwise distances between a sample of points on the object's surface. The distribution for each shape is found by taking a sample of points on the object's surface, finding the pairwise distances between them, and storing those in a histogram, as shown in the figure below:

![Example of a cube to histogram conversion.](/figures/objetct_to_histogram.png?raw=true "Example of a cube to histogram conversion.")

In this method, the object's alignment is indifferent since the distribution is shape-preserving and inherent to the shape. Another characteristic is its run-time, which can be controlled by the number of points sampled and is unaffected by the object's size. Its main drawback is that the score is only sometimes consistent due to the random nature of the point sampling. However, Wallace et al. made some adaptations to the original algorithm to overcome this shortcomings.

The original algorithm working principle is the following: it takes a sample of points on the surface of each object. Then it calculates the pairwise Euclidean distances for all these points and stores them in a 2D matrix. The distances are then normalized (concerning the maximum distance of each object) and converted into a histogram with a fixed number of evenly spaced bins. Instead of storing the frequency (number of points in the point cloud), the histogram uses the probability density due to random sampling. These discrete probability distributions represent the shapes of the objects in the point cloud. These distributions are then compared using the Minkowski distance - formula shown in equation bellow:

$$\\label{eq:eq_minkowski_distance}
D(X,\\ Y)=\\sum\_{i=1}^{n}\|x\_{i}-y\_{i}\|$$

The Minkowski distance is the sum of the absolute differences between the columns of the histograms - it is also important to mention that other metrics can be used, like the Bhattacharya and Chi-Square Distances.

The two methods diverge in the normalization process. The method proposed by Wallace et al. normalizes the distances for each object by dividing them by the maximum distance found. This means that the total size of the map is irrelevant, which is helpful since it allows a comparison between shapes without manually having to control their volume. Also, it allows the histogram bins to be aligned by simply picking the number of evenly spaced bins. However, since it transforms from a real point cloud to a simulated one, the method used needs to preserve distance; the similarity metric should consider the objective distances instead of the relative distances to the object's size. This procedure uses the largest distance from both objects to normalize the distances. The histogram is then created for the object with the largest distance. The same histogram bins were then used to create the second histogram. This ensures they are aligned and the Minkowski metric can be used safely.

Regarding outlier points (in this case, points that were far from the vehicle route), they were removed from the actual point cloud in a pre-processing stage. The following figure summarizes the general workflow of the pipeline:

![Wallace et al. methodology general flowchart.](/figures/general_comparison_method.png?raw=true "Wallace et al. methodology general flowchart.")

<div id='analysis_and_design'/>

## Analysis and Design of the System

As mentioned in the previous section, to assess the similarity between the reconstructed map and a reference map, it is possible to use several KPIs, namely the Hausdorff and the Chamfer distances. However, due to the disadvantages stated by Wallace et al., a new dissimilarity metric was proposed. The metric is called Distância de Minkowski-Silva (Minkowski-Silva Distance in English) or DMS. The DMS is the newly proposed metric implemented by the PIXEL container - which is the evaluation method of the proposed pipeline. The metric has shown promising results - as shown in the next section. This metric allows a quick, practical, and automatic assessment of the similarity between two maps while preserving the original structures' shapes. The following diagram showcases its proposed workflow:

![DMS Proposed Workflow.](/figures/dms_proposed_workflow.png?raw=true "DMS Proposed Workflow.")

The DMS metric was inspired by the one created by Wallace et al. with the following working principle:

1. Take a sample of points on the surface of each object.

2. Calculates the pairwise Euclidean distances for all these points and stores the distance in a 2D matrix.

3. Normalize the distances (concerning the maximum distance from both objects).

4. Convert the distance matrix into a histogram with a fixed number of evenly spaced bins.

5. Compare the histograms using the Minkowski distance - the formula is shown in the previous equation.

The biggest problem found with this method is the fact that instead of taking all of the points from the sample, Wallace et al. take a sample of points from the surface of each object, which allows them to calculate the pairwise Euclidean distances for all these points more efficiently - since calculating the pairwise Euclidean distances from a large sample of points is a heavy computational task but assures better results. In order to solve this issue and improve the normalization process, the author and Eng. Carlos
Silva made some changes to the original algorithm's inner workings. The modifications were the following:

1. Take all the points of each object.

2. Find the object centroid.

3. Calculates the Euclidean distances from one observed point to the centroid and stores the distance in a 2D matrix.

4. Normalize the distances (concerning the maximum distance from both objects).

5. Convert the distance matrix into a histogram with a fixed number of evenly spaced bins.

6. Compare the histograms using the new Minkowski-Silva distance:
    $$DMS(X,\ Y)=\sum_{i=1}^{N_{bins}}\left [\frac{|x_{i}-y_{i}|}{Counts_{total 1} + Counts_{total 2}}\right ]$$

The modifications allowed the newly proposed method to give a faster, more accurate similarity score between the two maps since it considers **all of the data set points**. If the pairwise Euclidean distance were still used for all the data points, the computer would run out of processing power while evaluating the similarity between the two maps. So instead of using the pairwise Euclidean distance, we opted first to find each of the sample's centroids, then calculate the (Euclidean) distance from an observed point to that centroid (which ensures a better computational performance), and then fill the 2D distance matrix with the outputted results.

Another remark concerns *Step 4*. During the histogram creation function, the process must create and align the two sample representative histograms with one standard normalized axis. Otherwise, it is not possible to safely use the Minkowski-Silva metric. Regarding the Minkowski metric, we changed it to ensure further bin normalization. Therefore, the equation in *Step 6* considers the total number of bin counts, allowing the comparison of point-cloud images with entirely different alignments. In this new distance, a similarity score of $0$ means that the point-cloud images are the same, and a similarity score of $1$ means that the point-cloud distances are entirely different.

<div id='results'/>

## Preliminary Results

After implementing this method in vanilla Python to better understand this new metric, the author and his supervisor (Carlos Silva) devised a three-stage plan. The plan is the following:

1. To verify this method, the author **created a Script in vanilla Python** where he **compared** the following:

    1. **The point cloud of a Cube with a point cloud of a Sphere:**

        1. The result of the DMS should be $1$ because a Cube and a Sphere do not possess common features, and only one bin should represent the Sphere.

    2. **The point clouds of two Cubes with different dimensions:**

        1. The results of the DMS should be $0$ (if they are the same) or close to $0$, and, due to the normalization process, both histograms should be overlapped.

2. **Create a set of scenarios and compare the retrieved point cloud data of the different scenes.**

    1. Find a straight road without many dynamic or static objects, place an ego vehicle on the road with a mounted Bosch B3 simulated LiDAR sensor, and retrieve the point cloud data of different scenarios.

    2. The **main goal** of this stage is to **better understand what the different values of the metric mean.**

3. **Use the DMS to evaluate the SLAM algorithm:**

    1. Create a scenario where an ego vehicle enters an uncongested roadway; this test scenario aims to see if the SLAM algorithm can detect traffic even when entering intersections, label them as dynamic objects, and remove them.

Concerning stage one, after implementing the DMS, generating the point cloud of a cube and of a sphere, and comparing them using the metric, we concluded that the metrics could distinguish between two utterly different point clouds, and even if we feed the algorithm with the point clouds of two cubes with different dimensions, due to the normalization process, the DMS has the expected behavior, as shown in the following figures:

![A point cloud of a cube compared to a point cloud of a sphere. The DMS score is 1, meaning that they are two completely different objects.](/figures/dms_cube_vs_sphere.png?raw=true "A point cloud of a Cube compared to a point cloud of a Sphere. The DMS score is 1, meaning that they are two completely different objects.")

![The point clouds of two cubes with different dimensions. The DMS score is 0, meaning that the DMS considers them the same object even if they present different sizes.](/figures/dms_cube_vs_cube.png?raw=true "The point clouds of two cubes with different dimensions. The DMS score is 0, meaning that the DMS considers them the same object even if they present different sizes.")

After creating a basic understanding of the algorithm, we moved to stage two of the characterization process. This stage took the most time of the three because of the scenario-creation process. From one base map, by variating the number of static and dynamic actors involved and their behaviors, the author created from 1 base map, 108 different scenes ranging from highway scenarios to accidents. This process allowed the author to make 11664 comparisons, which were then summarized on the following heat map:

![The heat map of stage two analysis process. The axis is the variation of the test cases, and the diagonal is the comparison of the axis with a reference map.](/figures/dms_heat_map.png?raw=true "The heat map of stage two analysis process. The axis is the variation of the test cases, and the diagonal is the comparison of the axis with a reference map.")

![Starting from the top left, the first image represents the Base Map, and the next point cloud represents a variation of the Base Map, a Dense Urban scenario. The bottom figure shows the DMS Similarity Score, which is 0.4606. The image on the left is the histogram representative of the two point clouds.](/figures/dms_base_map_dense_urban_scenario.png?raw=true "Starting from the top left, the first image represents the Base Map, and the next point cloud represents a variation of the Base Map, a Dense Urban scenario. The bottom figure shows the DMS Similarity Score, which is 0.4606. The image on the left is the histogram representative of the two point clouds.")

The heat map creation process proceeded as follows: the author went into the simulator and looked for a map that did not possess many embedded static actors, such as houses and trees. After encountering a straight road on a previously scanned Bosch test track - that Applied Intuition had previously digitalized and made available on the simulator. The author then created a series of scenarios where the ego would drive for 20 seconds on that stretch of road and retrieve the simulated data. The map of the world without any assets and interference from the weather conditions would provide the author with the *\"base map\"* (or reference map) for the analysis. Then, by following the scenario variation method also proposed in this dissertation, by variating certain assets of the basic scenario, the author could (from one functional scenario) create 108 different test cases (concrete scenarios) that would evaluate the metric under many different driving scenes. The process can be observed in the following figures:

![An example of a variation of the base map. As can be seen, we have the base map with dense traffic on both sides. The distance from the car in front of the ego is more or less 20 meters, and the timeout is 20 seconds. This process was followed in all test cases to ensure the reproducibility of the test scenarios.](/figures/dense_urban_scenario_example.png?raw=true "An example of a variation of the base map. As can be seen, we have the base map with dense traffic on both sides. The distance from the car in front of the ego is more or less 20 meters, and the timeout is 20 seconds. This process was followed to all test cases to ensure reproducibility of the test scenarios.")

![The base map on ADP scenario editor.](/figures/example_base_map_on_adp.png?raw=true "The base map on ADP scenario editor.")

At this stage, we addressed the following premise: *\"I needed to compare a reconstructed SLAM map with a reference map. The DMS can assess that similarity. However, what does that DMS score of 0.5 means? Is it okay for the complexity of the scene?\"* Those were the new research questions that arose from the development of this new metric. With this study, we can now understand what a similarity score of $0.5$ means, and if it makes sense for the scene under evaluation.

This process was also valuable for evaluating the SLAM algorithm because if the scene under evaluation concerns a dense urban area (with sunny conditions), and the DMS metric gives a score lower than $0.3$, it is excepted. After all, when the author created the reference map, there were not present any dynamic objects, so if the SLAM reconstructed the map correctly - correctly labeling and removing any dynamic object on the scene - the similarity score between the maps should theoretically be $0$ because that would mean that the SLAM could generate a reconstructed map equal to the reference map; however, an acceptable result could also be a value between $[0.0,\ 0.3]$ since it would indicate the presence of trees and houses, which are expected in a dense urban scenario.

We also realized a few of the metrics' peculiarities at this analysis stage. Our first discovery was that the pre-process functions are essential when we conducted the evaluation process on actual point clouds. Otherwise, the DMS scores will not portray the actual similarities of the scenes. The DMS pre-processing stage is required due to one characteristic of the normalization process. Since we normalized the distances concerning the maximum distance from both objects, that distance will inherently be an outlier point. An outlier point can be a point that is to far away from the vehicle course and must not be considered. Therefore, outlier points will harm the metric. So, the following pre-processing stage was created:

* Load the point cloud into an object,
* downsample the point cloud by voxel size,
* and statistically remove any outlier values of the point cloud.

We also found that the sensitivity of the metric on identifying point-cloud features depends on the number of points. The fewer the points, the worst the metrics results will be. So there must be a trade-off between point-cloud down sampling and outlier removal with the computation power to get the best possible results - on the other hand, depending on the number of points, the metric can identify local or global dissimilarities between point clouds.

Also, the smaller and farther away the objects are from the sensor, the worst the distance value will be. Furthermore, bigger objects such as trucks and *\"complex scenarios\"* are more dissimilar than smaller objects (persons and scooters/bicycles) compared to a base map. Another behavior we noticed was that weather conditions impact the metrics result - especially if it is raining or snowing. However, this behavior is more related to the actual LiDAR sensor and the simulator capabilities than the actual metrics functioning.

Corner cases, such as accidents, also impact the metric; for example, when we compared the base map with a snapshot of a point cloud in the moment of impact between the ego and another car, the DMS score was $0.8421$. This score means that the point clouds do not have many common features - which makes sense since the more significant part of the snapshot is occupied with the back car of the other vehicle.

These behaviors enabled us to conclude that this metric is better suited to assess the global similarity of the scenes - especially if we use an aggressive pre-processing stage. Imagine a parking lot; if we compared the similarity of a parking lot on Monday and on a Tuesday of the same week if the main structural blocks of the scenes did not change during those days (for example, the trees and the buildings around the parking lot did not change, and both days maintained the same weather and light conditions). Suppose the same cars are parked in similar spots. In that case, the DMS will tell us that the scenes are very similar because the metric will only look into the global similarity of the scene - and smaller objects, such as cars, will not significantly impact the score. However, if between Monday and Tuesday, it snowed or the cars parked in entirely different spots (forming two completely different structural blocks), the metric will correctly identify the differences between the two pictures - note that this process can also be controlled by the number of points used by the metric; therefore, the pre-processing stage is crucial because it enables us to control if we assess the global or local similarity of the scene.

The last behavior we noticed concerned the computational resources necessary to compute the similarity scores. With some preliminary analyses, we concluded that the algorithm took $2.173$ seconds to compare two point clouds with $10000$ points and $1.2771$ seconds to compare two point clouds with $1000$ points. Also, we noticed that the heaviest task was the histogram creation function (consuming $7.5$ MiB of memory). However, we could not conduct an extensive study on the performance of the metric - this could be interesting future work assessing the performance of the DMS and if it could be improved.

Concerning the third stage of the characterization process, we used the DMS to evaluate the performance of the SLAM's map reconstruction capabilities. In order to do so, we implemented Test Case 12 from a Test Plan that we devised - which will not be analyzed here. For generating the reference map, the author implemented a point cloud concatenation function used as a base for the automatic point cloud alignment method (which will also not be described here). After aligning several point clouds, Open3D has a concatenation operator
that allows the author to combine several aligned point clouds into one
single file.

Despite the somewhat poor results of some automatic alignment methods, this automatic point cloud concatenation method generated a relatively good reference map by only stitching a reduced number of point clouds from the noiseless virtual LiDAR sensor model. The other viable option is to put the LiDAR sensor in a strategic position that enables it to capture most of the scene actors and save that scan as the reference map - since the noiseless virtual LiDAR sensor model has an *\"infinite range,\"* this option is also viable however not very robust if the test course has many turns or slops, which will affect the scan.

![Starting from the top left, the first image represents the Reconstructed SLAM Map, and the next point cloud represents the Reference Map. The bottom image shows a mismatch on the z-axis when we overlap the Reconstructed SLAM Map with the Reference Map. The image on the right shows a mismatch on the x and y-axis when we overlay the two maps.](/figures/overlay_slam_reference_map.png?raw=true "Starting from the top left, the first image represents the Reconstructed SLAM Map, and the next point cloud represents the Reference Map. The bottom image shows a mismatch on the z-axis when we overlap the Reconstructed SLAM Map with the Reference Map. The image on the right shows a mismatch on the x, y-axis when we overlay the two maps.")

As can be seen in the figure above, although ideally, the DMS value should be $0$ - that would indicate that the SLAM map perfectly resembled the reference map - that is impossible despite being similar, the maps possess some misalignment on the
$(x,\ y,\ z,\ pitch,\ yaw,\ roll)$ axis. However, because of stage two of the characterization process, we know that the DMS score should be
between $0$ and $0.3$. If the DMS gives a score between $[0.0,\ 0.3]$, it will prove that the method could also be used to evaluate the performance of the SLAM algorithm.

![Starting from the top left, the first image represents the Reconstructed SLAM Map overlapped with the Reference Map. The bottom image shows the DMS score (of 0.1757), and the image on the left shows the histogram representative of both the Reconstructed SLAM map and the Reference Map.](/figures/dms_test_case_12_vs_reference_map.png?raw=true "Starting from the top left, the first image represents the Reconstructed SLAM Map overlapped with the Reference Map. The bottom image shows the DMS score (of 0.1757), and the image on the left shows the histogram representative of both the Reconstructed SLAM map and the Reference Map.")

As can be seen in the figure above, the result of the DMS between the Reference Map and the Reconstructed SLAM Map for Test Case 12 is $0.1757$; since the value is between the expected range, the tests pass, and it also shows that the DMS could also be used to evaluate the performance of the Map Reconstruction capabilities of the SLAM algorithm.

However, the metric still has room for improvement. To summarize, the proposed point cloud comparison method works properly, with a relatively fast processing time (for which the bottleneck is the histogram creation function), and it dispenses the painful point cloud alignment process. The metrics sensitivity on identifying point cloud features depends on the number of points. The fewer the points, the worst the metrics results will be. Therefore, there must be a trade-off between point cloud down sampling and outlier removal with the computation power to get the best possible results. Also, we noticed that weather conditions and corner cases, such as accidents, impact the metrics result - especially if it is raining or snowing. For future work, it would be interesting to investigate the metric capabilities further and use it in real-world point clouds or adapt it to other use cases, such as medical image analysis.
