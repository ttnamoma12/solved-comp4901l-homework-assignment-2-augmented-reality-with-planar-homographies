Download Link: https://assignmentchef.com/product/solved-comp4901l-homework-assignment-2-augmented-reality-with-planar-homographies
<br>
In this assignment, you will be implementing an AR application step by step using planar homographies. Before we step into the implementation, we will walk you through the theory of planar homographies. In the programming section, you will first learn to find point correspondences between two images and use these to estimate the homography between them. Using this homography you will then warp images and finally implement your own AR application.

<h1>Instructions</h1>

<ol>

 <li><strong>Integrity and collaboration: </strong>Students are encouraged to work in groups but each student must submit their own work. If you work as a group, include the names of your collaborators in your write up. Code should NOT be shared or copied. Please DO NOT use external code unless permitted. Plagiarism is strongly prohibited and may lead to failure of this course.</li>

 <li><strong>Start early! </strong>Especially those not familiar with Matlab.</li>

 <li><strong>Write-up: </strong>Your write-up should mainly consist of three parts, your answers to theory questions, resulting images of each step, and the discussions for experiments. Please note that we DO NOT accept handwritten scans for your write-up in this assignment. Please type your answers to theory questions and discussions for experiments electronically.</li>

 <li>Please stick to the function prototypes mentioned in the handout. This makes verifying code easier for the TAs.</li>

 <li><strong>Submission: </strong>Create a zip file, &lt;ust-login-id&gt;.zip, composed of your write-up, your Matlab implementations (including helper functions), and your implementations, results for extra credits (optional). Please make sure to remove the data /folder, m, warpH.m and any other temporary files you’ve generated. Your final upload should have the files arranged in this layout:</li>

</ol>

&lt;ust login id&gt;.zip

<ul>

 <li>&lt;ust-login-id&gt;/

  <ul>

   <li>&lt;ust-login-id&gt;.pdf</li>

   <li><u>matlab/</u></li>

  </ul></li>

</ul>

∗ MatchPics.m

∗ briefRotTest.m

∗ computeH.m

∗ computeH norm.m

∗ computeH ransac.m

∗ compositeH.m

∗ HarryPotterize auto.m

∗ ar.m

∗ yourHelperFunctions.m <em>(optional)</em>

<ul>

 <li><u>result/</u></li>

</ul>

Figure 1: A homography <strong>H </strong>links all points <strong>x</strong><em><sub>π </sub></em>lying in plane <em>π </em>between two camera views <strong>x </strong>and <strong>x</strong><sup>0 </sup>in cameras <em>C </em>and <em>C</em><sup>0 </sup>respectively such that <strong>x</strong><sup>0 </sup>= <strong>Hx</strong>. [From Hartley and Zisserman]

∗ ar.avi

<ul>

 <li><u>ec/ </u>(optional for extra credit)</li>

</ul>

∗ ar ec.m

∗ panorama.m

∗ the images required for generating the results.

<strong>Please make sure you do follow the submission rules mentioned above before uploading your zip file to CASS</strong>. Assignments that violate this submission rule will be <strong>penalized by up to 10% of the total score</strong>.

<ol start="6">

 <li><strong>File paths: </strong>Please make sure that any file paths that you use are relative and not absolute.</li>

</ol>

Not imread(’/name/Documents/subdirectory/hw1/data/xyz.jpg’) but imread(’../data/xyz.jpg’).

<h1>1           Homographies</h1>

<h2>Planar Homographies as a Warp</h2>

Recall that a planar homography is an warp operation (which is a mapping from pixel coordinates from one camera frame to another) that makes a fundamental assumption of the points lying on a plane in the real world. Under this particular assumption, pixel coordinates in one view of the points on the plane can be <em>directly </em>mapped to pixel coordinates in another camera view of the same points.

<strong>Q1.1 Homography                                                                                                                                                   </strong>(5 points)

Prove that there exists a homography <strong>H </strong>that satisfies equation 1 given two 3 × 4 camera projection matrices <strong>P</strong><sub>1 </sub>and <strong>P</strong><sub>2 </sub>corresponding to the two cameras and a plane <sup>Q</sup>. You do not need to produce an actual algebraic expression for <strong>H</strong>. All we are asking for is a proof of the existence of <strong>H</strong>.

<strong>x</strong><sub>1 </sub>≡ <strong>Hx</strong><sub>2                                                                                                                                     </sub>(1)

The ≡ symbol stands for identical to. The points <strong>x</strong><sub>1 </sub>and <strong>x</strong><sub>2 </sub>are in homogenous coordinates, which means they have an additional dimension. If <strong>x</strong><sub>1 </sub>is a 3D vector [<em>x<sub>i </sub>y<sub>i </sub>z<sub>i</sub></em>]<em><sup>T</sup></em>, it represents the 2D point [ (called inhomogenous coordinates). This additional dimension is a mathematical convenience to represent transformations (like translation, rotation, scaling, etc) in a concise matrix form. The ≡ means that the equation is correct to a scaling factor.

<strong>Note</strong>: A degenerate case happens when the plane <sup>Q </sup>contains both cameras’ centers, in which case there are infinite choices of H satisfying equation 1. You can ignore this special case in your answer.

<h2>The Direct Linear Transform</h2>

A very common problem in projective geometry is often of the form <em>x </em>≡ <strong>Ay</strong>, where <em>x </em>and <em>y </em>are known vectors, and <strong>A </strong>is a matrix which contains unknowns to be solved. Given matching points in two images, our homography relationship clearly is an instance of such a problem. Note that the equality holds only <em>up to scale </em>(which means that the set of equations are of the form <em>x </em>= <em>λ</em><strong>Hx</strong><sup>0</sup>), which is why we cannot use an ordinary least squares solution such as what you may have used in the past to solve simultaneous equations. A standard approach to solve these kinds of problems is called the Direct Linear Transform, where we rewrite the equation as proper homogeneous equations which are then solved in the standard least squares sense. Since this process involves disentangling the structure of the <strong>H </strong>matrix, it’s a <em>transform </em>of the problem into a set of <em>linear </em>equations, thus giving it its name.

<strong>Q1.2 Correspondences                                                                                                                             </strong>(15 points)

Let <strong>x</strong><sub>1 </sub>be a set of points in an image and <strong>x</strong><sub>2 </sub>be the set of corresponding points in an image taken by another camera. Suppose there exists a homography <strong>H </strong>such that:

<strong>x</strong>                   (<em>i </em>∈ {1···<em>N</em>})

where <strong>x</strong>are in homogenous coordinates,  and <strong>H </strong>is a 3 × 3 matrix. For each point pair, this relation can be rewritten as

<strong>A</strong><em><sub>i</sub></em><strong>h </strong>= 0

where <strong>h </strong>is a column vector reshaped from <strong>H</strong>, and <strong>A</strong><em><sub>i </sub></em>is a matrix with elements derived from the points  and . This can help calculate <strong>H </strong>from the given point correspondences.

<ol>

 <li>How many degrees of freedom does <strong>h </strong>have? (3 points)</li>

 <li>How many point pairs are required to solve <strong>h</strong>? (2 points)</li>

 <li>Derive <strong>A</strong><em><sub>i</sub></em>. (5 points)</li>

 <li>When solving <strong>Ah </strong>= 0, in essence you’re trying to find the <strong>h </strong>that exists in the null space of <strong>A</strong>. What that means is that there would be some non-trivial solution for <strong>h </strong>such that that product <strong>Ah </strong>turns out to be 0.</li>

</ol>

What will be a trivial solution for <strong>h</strong>? Is the matrix <strong>A </strong>full rank? Why/Why not?

What impact will it have on the eigen values? What impact will it have on the eigen vectors? (5 points)

<h2>Using Matrix Decompositions to calculate the homography</h2>

A homography <strong>H </strong>transforms one set of points (in homogenous coordinates) to another set of points. In this project, we will obtain the corresponding point coordinates using feature matches and will then need to calculate the homography. You have already derived that <strong>Ax </strong>= <strong>0 </strong>in Question 1. In this section, we will look at how to solve such equations using two approaches, either of which can be used in the subsequent assignment questions.

<h2>Eigenvalue Decomposition</h2>

One way to solve <strong>Ax </strong>= <strong>0 </strong>is to calculate the eigenvalues and eigenvectors of <strong>A</strong>. The eigenvector corresponding to <strong>0 </strong>is the answer for this. Consider this example:

                        <sup></sup>

3    6     −8

<strong>A </strong>=  0      0        6 

                        

0    0      2

Using the Matlab function eig, we get the following eigenvalues and eigenvectors:

                                                             <sup></sup>

1<em>.</em>0000    −0<em>.</em>8944     −0<em>.</em>9535




<strong>V </strong>=             0           0<em>.</em>4472        0<em>.</em>2860 

                                                             

0                 0             0<em>.</em>0953




h                     i

<strong>D </strong>=           3    0    2

Here, the columns of <strong>V </strong>are the eigenvectors and each corresponding element in <strong>D </strong>its eigenvalue. We notice that there is an eigenvalue of 0. The eigenvector corresponding to this is the solution for the equation <strong>Ax </strong>= <strong>0</strong>.

<table width="274">

 <tbody>

  <tr>

   <td width="77">3<strong>Ax </strong>=  00</td>

   <td width="21">6 00</td>

   <td width="177">−8 <sup> </sup>−0<em>.</em>8944 <sup>               </sup>0 <sup></sup>6  0<em>.</em>4472  =  0                                    2                      0                        0</td>

  </tr>

 </tbody>

</table>

<h2>Singular Value Decomposition</h2>

The Singular Value Decomposition (SVD) of a matrix A is expressed as:

<strong>A </strong>= <em>U</em>Σ<em>T<sup>T</sup></em>

Here, <em>U </em>is a matrix of column vectors called the “left singular vectors”. Similarly, <em>V </em>is called the “right singular vectors”. The matrix Σ is a diagonal matrix. Each diagonal element <em>σ<sub>i </sub></em>is called the “singular value” and these are sorted in order of magnitude. In our case, it is a 9 × 9 matrix.

<ul>

 <li>If <em>σ</em><sub>9 </sub>= 0, the system is <em>exactly-determined</em>, a homography exists and all points fit exactly.</li>

 <li>If <em>σ</em><sub>9 </sub>≥ 0, the system is <em>over-determined</em>. A homography exists but not all points fit exactly (they fit in the least-squares error sense). This value represents the goodness of fit.</li>

 <li>Usually, you will have at least four correspondences. If not, the system is <em>underdetermined</em>. We will not deal with those here.</li>

</ul>

The columns of <em>U </em>are eigenvectors of <strong>AA</strong><em><sup>T</sup></em>. The columns of <em>V </em>are the eigenvectors of <strong>A</strong><em><sup>T</sup></em><strong>A</strong>. We can use this fact to solve for <strong>h </strong>in the equation <strong>Ah </strong>= <strong>0</strong>. Using this knowledge, let us reformulate our problem of solving <strong>Ax </strong>= <strong>0</strong>. We want to minimize the error in solution in the least-squares sense. Ideally, the product <strong>Ah </strong>should be 0. Thus the sum-squared error can be written as:

(<strong>Ah </strong>− <strong>0</strong>)<em><sup>T</sup></em>(<strong>Ah </strong>− <strong>0</strong>)

<strong>Ah</strong>

Minimizing this error with respect to <strong>h</strong>, we get:

<em>d</em>

<em>f      </em>=     0

<em>d</em><strong>h</strong>

<strong>A</strong><em><sup>T</sup></em><strong>Ah </strong>=        0

This implies that the value of <strong>h </strong>equals the eigenvector corresponding to the zero eigenvalue (or closest to zero in case of noise). Thus, we choose the smallest eigenvalue of <strong>A</strong><em><sup>T</sup></em><strong>A</strong>, which is <em>σ</em><sub>9 </sub>in Σ and the least-squares solution to <strong>Ah </strong>= <strong>0 </strong>is the the corresponding eigenvector (in column 9 of the matrix <strong>V</strong>).

<h1>Theory Questions</h1>

<strong>Q1.3 Homography under rotation                                                                                                                 </strong>(5 points)

Prove that there exists a homography <strong>H </strong>that satisfies <strong>x</strong><sub>1 </sub>≡ <strong>Hx</strong><sub>2 </sub>, given two cameras separated by a pure rotation. That is, for camera 1, <strong>x</strong><sub>1 </sub>= <strong>K</strong><sub>1</sub>[<strong>I 0</strong>]<strong>X </strong>and for camera 2, <strong>x</strong><sub>2 </sub>= <strong>K</strong><sub>2</sub>[<strong>R 0</strong>]<strong>X</strong>. Note that <strong>K</strong><sub>1 </sub>and <strong>K</strong><sub>2 </sub>are the 3 × 3 intrinsic matrices of the two cameras and are different. <strong>I </strong>is 3 × 3 identity matrix, <strong>0 </strong>is a 3×1 zero vector and <strong>X </strong>is a point in 3D space. <strong>R </strong>is the 3×3 rotation matrix of the camera.

<strong>Q1.4 Understanding homographies under rotation                                                                           </strong>(5 points)

Suppose that a camera is rotating about its center <strong>C</strong>, keeping the intrinsic parameters <strong>K </strong>constant. Let <strong>H </strong>be the homography that maps the view from one camera orientation to the view at a second orientation. Let <em>θ </em>be the angle of rotation between the two. Show that <strong>H</strong><sup>2 </sup>is the homography corresponding to a rotation of 2<em>θ</em>. Please limit your answer within a couple of lines. A lengthy proof indicates that you’re doing something too complicated (or wrong).

<strong>Q1.5 Limitations of the planar homography                                                                                           </strong>(5 points)

Why is the planar homography not completely sufficient to map any arbitrary scene image to another viewpoint? State your answer concisely in one or two sentences.

<strong>Q1.6 Behavior of lines under perspective projections                                                                      </strong>(5 points)

We stated in class that perspective projection preserves lines (a line in 3D is projected to a line in 2D). Verify algebraically that this is the case, i.e., verify that the projection <strong>P </strong>in <strong>x </strong>= <strong>PX </strong>preserves lines.

<h1>2           Computing Planar Homographies</h1>

<h2>Feature Detection and Matching</h2>

Before finding the homography between an image pair, we need to find corresponding point pairs between two images. But how do we get these points? One way is to select them manually (using cpselect), which is tedious and inefficient. The CV way is to find interest points in the image pair and automatically match them. In the interest of being able to do cool stuff, we will not reimplement a feature detector or descriptor here, but use built-in MATLAB methods. The purpose of an interest point detector (e.g. Harris, SIFT, SURF, etc.) is to find particular salient points in the images around which we extract feature descriptors (e.g. MOPS, etc.). These descriptors try to summarize the content of the image around the feature points in as succint yet descriptive manner possible (there is often a tradeoff between representational and computational complexity for many computer vision tasks; you can have a very high dimensional feature descriptor that would ensure that you get good matches, but computing it could be prohibitively expensive). Matching, then, is a task of trying to find a descriptor in the list of descriptors obtained after computing them on a new image that best matches the current descriptor. This could be something as simple as the Euclidean distance between the two descriptors, or something more complicated, depending on how the descriptor is composed. For the purpose of this exercise, we shall use the widely used FAST detector in concert with the BRIEF descriptor.

Figure 2: A few matched FAST feature points with the BRIEF descriptor.

<strong>Q2.1.1 FAST Detector                                                                                                                                            </strong>(5 points)

How is the FAST detector different from the Harris corner detector that you’ve seen in the lectures? (You will probably need to look up the FAST detector online.) Can you comment on its computational performance vis-`a-vis the Harris corner detector?

<strong>Q2.1.2 BRIEF Descriptor                                                                                                                                     </strong>(5 points)

How is the BRIEF descriptor different from the filterbanks you’ve seen in the lectures? Could you use any one of those filter banks as a descriptor?

<strong>Q2.1.3 Matching Methods                                                                                                                                   </strong>(5 points)

The BRIEF descriptor belongs to a category called binary descriptors. In such descriptors the image region corresponding to the detected feature point is represented as a binary string of 1s and 0s. A commonly used metric used for such descriptors is called the <em>Hamming distance</em>. Please search online to learn about Hamming distance and <em>Nearest Neighbor</em>, and describe how they can be used to match interest points with BRIEF descriptors. What benefits does the Hamming distance distance have over a more conventional Euclidean distance measure in our setting?

<strong>Q2.1.4 Feature Matching                                                                                                                                  </strong>(10 points)

Please implement a function:

[locs1, locs2] = matchPics(I1, I2)

where I1 and I2 are the images you want to match. locs1 and locs2 are <em>N </em>×2 matrices containing the <em>x </em>and <em>y </em>coordinates of the matched point pairs. Use the provided function fast corner detect 9( im, threshold) to compute the features (using a higher threshold gives you less points), then build descriptors using the provided computeBrief function and finally compare them using the provided method customMatchFeatures.

Use the function customShowMatchedFeatures(im1, im2, locs1, locs2) to visualize your matched points and include the result image in your write-up. An example is shown in Figure 2. We provide you with the function:

[desc, locs] = computeBrief(img, locs in)

which computes the BRIEF descriptor for img. locs in is an <em>N </em>× 2 matrix in which each row represents the location (<em>x,y</em>) of a feature point. Please note that the number of valid output feature points could be less than the number of input feature points. desc is the corresponding matrix of BRIEF descriptors for the interest points.

<strong>Q2.1.5 BRIEF and Rotations                                                                                                                            </strong>(10 points)

Let’s investigate how BRIEF works with rotations. Write a script briefRotTest.m that:

<ul>

 <li>Takes the cv cover.jpg and matches it to itself rotated [Hint: use imrotate] in increments of 10 degrees.</li>

 <li>Stores a histogram of the count of matches for each orientation.</li>

 <li>Plots the histogram using plot.</li>

</ul>

Visualize the feature matching result at three different orientations and include them in your writeup. Explain why you think the BRIEF descriptor behaves this way. What happens when you switch to a different feature detector? [Hint: For instance, SURF features]. Would the plot change significantly? Why/Why not?

<h2>Homography Computation</h2>

<strong>Q2.2.1 Computing the Homography                                                                                                          </strong>(15 points)

Write a function computeH that estimates the planar homography from a set of matched point pairs.

function [H2to1] = computeH(x1, x2)

x1 and x2 are <em>N </em>× 2 matrices containing the coordinates (<em>x,y</em>) of point pairs between the two images. H2to1 should be a 3 × 3 matrix for the best homography from image 2 to image 1 in the least-square sense. You can use eig or svd to get the eigenvectors (see Section 1 of this handout for details).

<h2>Homography Normalization</h2>

Normalization improves numerical stability of the solution and you should always normalize your coordinate data. Normalization has two steps:

<ol>

 <li>Translate the mean of the points to the origin.</li>

</ol>

√

<ol start="2">

 <li>Scale the points so that the largest distance to the origin is</li>

</ol>

This is a linear transformation and can be written as follows:

<strong> T</strong>1<strong>x</strong>1

<strong> T</strong>2<strong>x</strong>2

where and are the normalized homogeneous coordinates of <strong>x</strong><sub>1 </sub>and <strong>x</strong><sub>2</sub>. <strong>T</strong><sub>1 </sub>and <strong>T</strong><sub>2 </sub>are 3 × 3 matrices. The homography H from <strong>x</strong>f<sub>2 </sub>and <strong>x</strong><sub>f</sub>1 computed by computeH satisfies:

<strong>x</strong>f1 = <strong>H</strong><strong><sup>x</sup></strong><sub>f</sub>2

By substituting and with <strong>T</strong><sub>1</sub><strong>x</strong><sub>1 </sub>and <strong>T</strong><sub>2</sub><strong>x</strong><sub>2</sub>, we have:

<strong>T</strong><strong> HT</strong>

<strong>x</strong>1 = <strong>T</strong> 2 2

<strong>Q2.2.2 Homography with normalization                                                                                                </strong>(10 points)

Implement the function computeH norm:

function [H2to1] = computeH norm(x1, x2).

This function should normalize the coordinates in x1 and x2 and call computeH(x1, x2) as described above.

<h2>RANSAC</h2>

The RANSAC algorithm can generally fit any model to noisy data. You will implement it for (planar) homographies between images. Remember that 4 point-pairs are required at a minimum to compute a homography.

<strong>Q2.2.3 Implement RANSAC for computing a homography                                                           </strong>(25 points)

Write a function:

function [bestH2to1, inliers] = computeH ransac(locs1, locs2)

where bestH2to1 should be the homography <strong>H </strong>with most inliers found during RANSAC. <strong>H </strong>will be a homography such that if <strong>x</strong><sub>2 </sub>is a point in locs2 and <strong>x</strong><sub>1 </sub>is a corresponding point in locs1, then <strong>x</strong><sub>1 </sub>≡ <strong>Hx</strong><sub>2</sub>. locs1 and locs2 are <em>N </em>×2 matrices containing the matched points. inliers is a vector of length <em>N </em>with a 1 at those matches that are part of the consensus set, and 0 elsewhere. Use computeH norm to compute the homography.

<h2>Automated Homography Estimation and Warping</h2>

<strong>Q2.2.4 Putting it together                                                                                                                                </strong>(10 points)

Write a script HarryPotterize.m that

<ol>

 <li>Reads cv cover.jpg, cv png, and hp cover.jpg.</li>

 <li>Computes a homography automatically using MatchPics and computeH ransac.</li>

 <li>Warps hp cover.jpg to the dimensions of the cv png image using the provided warpH function.</li>

 <li>At this point you should notice that although the image is being warped to the correct location, it is not filling up the same space as the book. Why do you think this is happening? How would you modify hp cover.jpg to fix this issue?</li>

 <li>Implement the function:</li>

</ol>

Figure 3: Text book

Figure 4: HarryPotterized Text book

function [ composite img ] = compositeH( H2to1, template, img ) to now compose this warped image with the desk image as in Figure 4.

<ol start="6">

 <li>Include your result in your write-up.</li>

</ol>

<h1>3           Creating your Augmented Reality application</h1>

<strong>Q3.1 Incorporating video                                                                                                                                 </strong>(20 points)

Now with the code you have, you’re able to create you own Augmented Reality application. What you’re going to do is HarryPoterize the video ar source.mov onto the video book.mov. More specifically, you’re going to track the computer vision text book in each frame of book.mov, and overlay each frame of ar source.mov onto the book in book.mov. Please write a script ar.m to implement this AR application and save your result video as ar.avi in the result/ directory. You may use the function loadVid.m that we provide to load the videos. Your result should be similar to the LifePrint project. You’ll be given full credits if you can put the video together correctly. See Figure 5 for an example frame of what the final video should look like.

Note that the book and the videos we have provided have very different aspect ratios (the ratio of the image width to the image height). You must either use imresize or crop each frame to fit onto the book cover.

Cropping an image in Matlab is easy. You just need to extract the rows and columns you are interested in. For example, if you want to extract the subimage from point (40<em>,</em>50) to point

(100<em>,</em>200), your code would look like img cropped = img(50:200, 40:100). In this project, you must crop that image such that only the central region of the image is used in the final output. See Figure 6 for an example.

Also, the video book.mov only has translation of objects. If you want to account for rotation of objects, scaling, etc, you would have to pick a better feature point representation (like ORB).

Figure 5: Rendering video on a moving target

Figure 6: Crop out the yellow regions of each frame to match the aspect ratio of the book

<h1>4           Extra Credit</h1>

<strong>Q4.1x: Make Your AR Real Time                                                                                                                  </strong>(15 points)

Write a script ar ec.m that implements the AR program described in Q3.1 in real time. As an output of the script, you should process the videos frame by frame and have the combined frames played in real time. You don’t need to save the result video for this question. The extra credits will be given to fast programs measured by FPS (frames per second). More specifically, we give 5 points to programs that run faster than 3 FPS, 10 points to programs running faster than 5 FPS and 15 points to programs running faster than 7 FPS.

Include the FPS and describe your techniques used for speeding up in your write-up. You should record the FPS <strong>in the virtual barn environment</strong>, computed as the total processing time divided by the total number of frames. You may exclude the data loading and displaying time.

<strong>Q4.2x: Create a Simple Panorama                                                                                                               </strong>(10 points)

Take two pictures with your own camera, separated by a pure rotation as best as possible, and then construct a panorama with panorama.m. Be sure that objects in the images are far enough away so that there are no parallax effects. You can use Matlab’s cpselect to select matching points on each image or some automatic method. Submit the original images, the final panorama, and the script panorama.m that loads the images and assembles a panorama. We have provided two images for you to get started (data/pano left.png and data/pano right.png). Please use your own images when submitting this project. In your submission, include your original images and the panorama result image in your write-up. See the figures below for example.

Figure 7: Original Image 1 (left)

Figure 8: Original Image 2 (right)

Figure 9: Panorama