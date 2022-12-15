### This is V3 of draft (2-in-1: ReadMe and BlogPost)... Missing format and removal of suggested material

# mini-QE-fft
Pulling out the 3D FFT call from QE and sticking it in its own reproducer to facilitate the debugging of the large performance gap with cufft

## Build 
* HIP  
`make hip`
* OpenMP  
`make omp`
* OpenMP with hipFFT callback for scaling loop on device with hipFFT execution  
`make omp_callback`
* Separate OMP/HIP Compilation
`make separate`

----
## hipfftExtPlanScaleFactor Brief
hipfftExtPlanScaleFactor API to efficiently multiply each output element of a FFT by a given scaling factor.

   /*
   Set scaling factor.
 
   hipFFT multiplies each element of the result by the given factor at the end of the transform.
 
   The supplied factor must be a finite number.  That is, it must neither be infinity nor NaN.
 
   This function must be called after the plan is allocated using
   ::hipfftCreate, but before the plan is initialized MakePlanMany. (Currently the only Make Plan function that supports the scaling API)
   */
   
## Order of the Make Plan Directive w/ ExtPlanScaleFactor
```c++
  int FFT_DIM[3] = {200, 128, 128};
  int DATA_DIM[3]  = {200, 128, 128};
  int *ptr = NULL;
  ptr = FFT_DIM;
  int *d_dim = NULL;
  d_dim = DATA_DIM;

  //Create Plan Handle
  hipfftHandle hipfft_plan_3d = nullptr;

  //Create Plan
  hipfftCreate(&hipfft_plan_3d);

  //Implementing EXTPlanScaleFactor
  hipfftExtPlanScaleFactor(hipfft_plan_3d, tscale);

  //Make Plan directive
  hipfftMakePlanMany( hipfft_plan_3d, 3, ptr, d_dim, 1, nPs, d_dim, 1, nPs, HIPFFT_Z2Z, 1, nullptr );

  hipEventSynchronize(stop);

  // Call hipFFT function of interest
  printf("Executing hipFFT\n");
  hipfftExecZ2Z( hipfft_plan_3d, deviceData, deviceData, HIPFFT_FORWARD );

 hipEventSynchronize(stop);
  ```
  
## Extract Data File (Files need to be updated for correct device data validation and given suitable names)
* Input Data
`tar -xvf data.tar.gz` //PlanScaleBeforeExecz2z_fd.txt
* Validation Data
`tar -xvf validation.tar.gz` //PlanScaleAfterDo_fd.txt
* Unscaled Forward Transform Data
`tar -xvf ??.tar.gz` //PlanScaleAfterExecz2z_device.txt

## Run
* HIP_Planscale  
`./exe_hipscale`
* HIP  
`OMP_NUM_THREADS=<numHostThreads> ./exe_hip`
  * Where `<numHostThreads>` is the number of threads to run on the host for the scaling loop
* OMP  
`./exe_omp`
* OpenMP with hipFFT callback for scaling loop on device with hipFFT execution  
`./exe_omp_callback`
* Separate OMP/HIP Compilation
`./exe_exe_linked`

## Output
ElapsedTime measuring the time it took to perform the 3D FFT

## Adding an Inverse Transform
Note that there is not a built-in trigger to prevent scaling of inverse transform when using a plan that utilizes the ExtPlanScaleFactor API.

To Implement an Inverse Transform, the following should be done:
* Allocate another "Plan Handle" and "Plan"
* Initialize this new plan using a make plan function, such as PlanMany or MakePlanMany
* Set Exec function for inverse transform, example: hipfftExecZ2Z( "Plain Plan", deviceData, deviceData, HIPFFT_BACKWARD )

Without utilizing these notes, the resulting data will be scaled by twice the size of the scaling factor or length of signal

![image](https://user-images.githubusercontent.com/112571800/207444091-bc97c238-ac30-453e-923b-62a641710565.png)

![image](https://user-images.githubusercontent.com/112571800/207438905-27e9776b-5498-4277-a6b4-99fda1d121b6.png)

# Fortran, OpenMP, Quantum Espresso… Oh My (Quentarius Moore - Coop Presentation Fall 2022)

## Overview
* Acknowledgements
* Introduction + About Me
* Real World Science Background
* Computational Framework
   * Quantum Espresso
   * PWscf
* Porting Quantum Espresso
   * CPU Baseline
   * Nvidia Baseline
   * AMD Baseline
   * AMD Optimization
* Lessons Learned
* Future Work

----

## Acknowledgents

### AMD Team

<p float="left" align="center">
  <img src="https://user-images.githubusercontent.com/112571800/207923399-164a2522-4a43-438e-94dd-8835c73fb6b7.png" width="100" />
  <img src="https://user-images.githubusercontent.com/112571800/207923435-3a2c6398-5c60-455b-a8cd-06314d4d0e5b.png" width="100" /> 
  <img src="https://user-images.githubusercontent.com/112571800/207923475-cd30932a-93d0-4a21-925c-492d8e3f943b.png" width="100" />
  <img src="https://user-images.githubusercontent.com/112571800/207923574-70b1de03-2fd4-4a8e-b196-5bea5f7dc2d4.png" width="100" />
</p><br>
<p align="center">
Jose Noudohouenou<sub>OpenMP expert</sub> | Ossian O’Reilly<sub>Quantum Espresso expert</sub> | Steve Leung<sub>roc/hipFFT expert</sub> | Justin Chang<sub>OpenMP Offload + Fortran expert</sub>
</p><br><br>


<p float="left" align="center">
  <img src="https://user-images.githubusercontent.com/112571800/207923328-82827849-8170-4bee-b0eb-350cab89a0f1.png" width="100" />
  <img src="https://user-images.githubusercontent.com/112571800/207923261-5549ae21-e8c6-4b47-969f-3cb13b491f0f.png" width="100" /> 
  <img src="https://user-images.githubusercontent.com/112571800/207923183-f1452237-a577-4621-8539-a5d8570d5127.png" width="100" />
</p> <br>

<p align="center">
Gina Sitaraman<sub>Profiler expert</sub> | Stephanie Craft<sub>Moral expert</sub> <br> Nicholas Malaya<sub>Twitter expert</sub>   [Reached out to me on Twitter about the Co-Op: So grateful for him!]
</p><br><br>

<p float="left" align="center">
  <img src="https://user-images.githubusercontent.com/112571800/207922695-a2471497-59f9-478f-a7c2-123e0662efb0.png" width="100" />
  <img src="https://user-images.githubusercontent.com/112571800/207922838-88795532-fd70-4ec9-bdb3-0b17430a286b.png" width="100" /> 
  <img src="https://user-images.githubusercontent.com/112571800/207922904-a41c5dc6-1e27-45d7-9306-b6350784bdd0.png" width="100" />
</p> <br>
<p align="center">
Noah Wolfe<sub>Micro-Managing expert</sub> | Chris Kime<sub>Macro-Managing expert</sub> | Sophie Roth<sub>HR expert</sub>  
</p><br><br>


----
<br><br>
#### Home Team
Dr. James D. Batteas

The Batteas Research Group

NSF Center for the Mechanical Control of Chemistry

Texas A&M University

Collaborators at Sandia National Laboratories (Contract:DE-NA0003525)



![image](https://user-images.githubusercontent.com/112571800/207920241-a4fa043d-18bd-482b-acc2-77a9fc970baf.png)

![image](https://user-images.githubusercontent.com/112571800/207920448-55c06ae7-4e50-448a-b151-7f4bf64f2432.png)


![image](https://user-images.githubusercontent.com/112571800/207920075-84385af0-8e70-4195-8b88-83597ebea9fb.png)


![image](https://user-images.githubusercontent.com/112571800/207919921-d3198ea5-d286-44ba-bb91-b66e977e664e.png)

![image](https://user-images.githubusercontent.com/112571800/207919937-2760f78b-dbd1-455b-87f0-cde50e4fafdc.png)




<br><br><br>

----
<br><br><br>
## Introduction + About Me <br>
![image](https://user-images.githubusercontent.com/112571800/207917727-d8ab2831-18bc-4b82-8144-cb6ddc0f4895.png)

<br>
<img align="center" width="32%" height="32%" src="https://user-images.githubusercontent.com/112571800/207913106-3f8fe0e2-f63d-49dc-bd97-f171cdd6f79f.png"> <br><br><br>
   

![image](https://user-images.githubusercontent.com/112571800/207913177-051a0ab8-e3d1-4b86-9da3-36191ef8f3cc.png)
*Service to Others...How to make this a caption? Figuring it out*

![image](https://user-images.githubusercontent.com/112571800/207913218-402d9954-5251-4ed2-a1d5-01c4a3284a74.png)
*Service to Others...May need to remove for copyright? Same as the gif at end of blog. not sure*

![image](https://user-images.githubusercontent.com/112571800/207917227-f6ab15cf-a053-4e1d-ba96-b8164ea5d2d4.png)
*AMD Photo Forum*

![image](https://user-images.githubusercontent.com/112571800/207917345-7b96516e-962d-4f08-881f-4fd4de3b5066.png)
*AMD 2022 Fall Chess Tournament*

<p align="center">
<b>“If I have seen further, it is by standing on the shoulders of giants”</b>
</p>



----

## Real World Science Background
![image](https://user-images.githubusercontent.com/112571800/207719397-badaf872-1a7b-4f66-bfbe-285ab52e79cd.png)
<sub>Overview of structural, mechanical, electrical, thermal, and chemical properties of 2D materials that are relevant to tribological performance</sub><br>


Two-Dimensional (2D) Nanomaterials
   * Unique properties and applications
   * Flexible Electronics
   * Energy Storage
   * Medicine
   * Sensors


<img align="center" src="https://user-images.githubusercontent.com/112571800/207719114-174713e9-bb7e-4d63-ae62-00b02b4a957e.png">
 <p align="center"> 
   Want to center this image
<sub><b>The Mariner 9 planetary space probe</b></sub>
   </p>


<p align="center"> 
   <b>2D materials possess exceptional frictional properties for use as solid dry lubricants </b>
   </p>

* Zhang, S; et al. Tribology of two-dimensional materials: From mechanisms to modulating strategies. Mater. Today. 2019, 26, 67-86 <br>
* NASA Dry Lubricant Smooths the Way for Space Travel, Industry https://spinoff.nasa.gov/Spinoff2015/ip_7.html 2015

----

### Real World Science Background - Example
<b>Molybdenum disulfide (MoS<sub>2</sub>)</b> <br>
Center this image
![image](https://user-images.githubusercontent.com/112571800/207717655-6d7b02b7-1800-4403-bf9f-f213b911171b.png)
![image](https://user-images.githubusercontent.com/112571800/207717689-8140dbd5-9a31-436d-bee2-9b4f12a67453.png)
![image](https://user-images.githubusercontent.com/112571800/207717703-1f71246b-a31b-4ae1-91d3-2a0c8850ffb8.png)

* Exciting mechanical, electronic and frictional properties.

* Promising potentials for solid lubricants, ion batteries, low power transistors, optoelectronic 
* Stable against reactions with environmental species.
* Three atomic layers hexagonal

Challenges

* Strained regions can lead to mechano-chemically accelerated chemical breakdown (e.g. oxidation) of 2D materials.

* Our lab’s work has shown that exposure to environmental species, e.g. oxygen and water, has been shown to degrade MoS2 .

* Substrates have been shown to couple with the electronic structure of 2D nanomaterials, affecting reactivity.

<sub>
* Toh, R.; Pumera, M., et al. 3R phase of MoS2 and WS2 outperforms the corresponding 2H phase for hydrogen evolution. Chem. Commun. 2017, 53, 3054-3057 <br>
* Radisavljevic,B., et al. Single-layer MoS2 Transistors. Nature Nanotechnology 2011, 6, pages147–150
</sub>

----

## Computational Framework
### Quantum Espresso & PWscf
* Quantum Espresso
   * What it’s used for
      * Open-Source density functional theory (DFT) code, using a Plane-Wave basis set and pseudopotentials, and widely used in Materials Science and Quantum Chemistry to compute states of complex systems.
   * Language
      * Fortran90
   * What is supported
      * CPU (MPI/openMP)
      * Nvidia GPU (CUDA Fortran)
      * AMD GPU (OpenMP Offload)
         * AMD Internal only (Thanks Ossian)
* PWscf
   * What it’s used for
      * Plane-Wave self consistent field is a set of programs for electronic structure calculations within density functional theory, i.e. Ground state calculations and structural optimization.



----

## Porting Quantum Espresso
### CPU Baseline
* System Details
   * EPYC 7742
* QE Configuration
   * MPI Ranks: 1
   * Threads: 256
   * Atoms: 1 
   * Cutoff: 50 
   * Charge Density: 425 
   * Iterations: 21 (Full run)
* Performance
   * Runtime: 33m 41s wall-time
   * FOM: 15004 (V_SMOOTH * # of iterations / wall-time) <br><br>
<sub><b>FOM = Figure of Merit</b></sub> <br><br>
<sub><b>V_SMOOTH = # of G-vectors (used in charge density expansion) that make up the smooth grid</b></sub>

----

#### CPU Baseline – Runtime Analysis
* Tool
   * Oracle Developer Studio
* Hotspot Kernels
   * qvan2
   * dgemms
   * zgemms
   * lots of ffts <br><br>
![image](https://user-images.githubusercontent.com/112571800/207713134-980bf700-fe31-4abc-aead-a3bd8f84e807.png)


#### CPU Baseline – Call Graph
* Tool
   * Oracle Developer Studio
* Quick Sanity Check
   * Again, lots of FFTs on critical path (hint hint).
* Potentially helpful in future debug tasks.
![image](https://user-images.githubusercontent.com/112571800/207712907-0cc517ea-7bfb-4f22-9f02-e58528b0080c.png)


#### CPU Baseline – Timeline Trace
* Sanity checking load on CPU
   * No gaps
   * No “red flags”
   * CPU is saturated
![image](https://user-images.githubusercontent.com/112571800/207712700-957f065a-0ccb-4a79-ac44-fd59dab9df25.png)

----

### AMD Baseline
* System Details
   * Lockhart node
      * CPU: EPYC 7a53s (Trento)
      * GPU: MI250x (A0)
      * ROCm: 5.3.0-63
* QE Configuration
   * MPI Ranks: 1
   * Num GPUs: 1
   * Host Threads: 1
   * Atoms: 1 
   * Cutoff: 50 
   * Charge Density: 425 
   * Iterations: 21
* Performance
   * Runtime: 64m 0s wall-time
   * FOM: 7901

----

#### AMD Baseline -- Gotchas

* Traps/Tricks
   * Fortran offload requires Cray compiler
      * Forced us to use Lockhart nodes
   * Dependencies/Libraries
      * Thanks to Ossian, most of the issues were patched ahead of time

----


#### AMD Baseline – Timeline Trace
* Sanity checking computational load on AMD MI250x GPU
   * Hmm… very few gaps on GPU? Good thing, right?
      * No code changes yet, so we expect to see work on the host in the critical path (similar to A100).
      * Tells us something is taking too long on device (in comparison to A100).
![image](https://user-images.githubusercontent.com/112571800/207710327-91d0261d-30d0-466e-bbb4-a27f2a44e4af.png)
<p align="center"> 
   <strong>↑ Rocprof + Perfetto</strong>
   </p>

----

#### AMD Baseline – Timeline Trace cont.
* Observation
   * fft_scalar_hipfft is taking 849ms on MI250x!!! 679 occurrences 
* Next Step
   * What do we see on A100? (Need to tackle how to we create continuity and structure in the story without A100 data/comparison)
![image](https://user-images.githubusercontent.com/112571800/207709392-33538a02-b75e-4108-b6fa-0340cf8ebee9.png)

![image](https://user-images.githubusercontent.com/112571800/207709345-a175d3bb-ca4f-4921-ac76-63d5d727abf6.png)

----
  
#### QE FFT Scaling Factor – Loop (Be nice to format similar to the presentation
* fft_scalar_cufft/hipfft.f90
   * Performs 3D FFT on device.
   * Applies scaling factor to the data post fft transform.
      * This can be done naively using a loop.
<p align="center"> 
   Nvidia (CUDA-Fortran)
   </p>
* Scaling loop runs natively on the device and does so quickly.

![image](https://user-images.githubusercontent.com/112571800/207708990-73d4d458-42d4-4298-9a24-5e759cdb32d7.png)

<p align="center">
AMD (Fortran/OpenMP offload)
   </p>
* Scaling loop runs on device using OpenMP offload
   * Timeline trace confirms scaling loop is running on device.
   * Perhaps it’s operating on host data f_d, hence the poor perf?
   * Lots of questions and possible fixes.
      * Time for miniApp!!!
![image](https://user-images.githubusercontent.com/112571800/207709224-713ff16f-6c9c-464d-8744-5940dcc70e3e.png)
![image](https://user-images.githubusercontent.com/112571800/207709242-694aa087-c3a8-4489-aebd-9f6963ccf27f.png)
![image](https://user-images.githubusercontent.com/112571800/207709248-60707dd3-99fd-44ce-b373-73f30b3983e4.png)



----


#### QE FFT Scaling Factor – mini-QE (Be nice to split screen with text and image. Figure out how to if possible)
* mini-QE-fft 
   * Repo: https://github.com/AMD-HPC/mini-QE-fft
   * Removes compiler, library and other dependencies of full QE app.
   * Removes complicated and slow build times and runtimes.
   * Encapsulates single iteration of forward 3D FFT from QE.
   * Runs with real QE FFT data.
   * Validates with real QE FFT scaled output.
   * Performance Optimizations Tested
      * OpenMP offload across the board.
         * Slow
      * HIP only
         * Fast but doesn’t play well with Fortran
      * HIP & OpenMP (Separate Compilation Units)
         * Fast but ugly
      * hipfftExtPlanScaleFactor()
<br><br>

![image](https://user-images.githubusercontent.com/112571800/207706817-e2e57b6f-65bb-40eb-9e80-791f49454052.png)

----

#### hipFFTExtPlanScaleFactor
* Describing the hoops to get the scaling factor first functional (no runtime errors) and then validating.
   * Set $LD_LIBRARY_PATH to rocm-5.4.0 libs.
   * PlanMany overriding pre-scaled entries.
   * Ordering of directives! (Create – PlanScaleFactor – !MakePlanMany! - ExecZ2Z)
      * Bug found where API only scales with MakePlanMany.
   * In QE, accessing the correct data from device for validation.

----

#### QE FFT Scaling Factor – miniApp port back to QE
* MiniApp works, so all done, right? Wrong
* Final debugging session:
   * Issues with multiple iterations of fft calls and using an array of plans instead of a singular plan.
   * Issues with inverse FFT also applying the scaling factor (had to zero out the scaling factor).
   * New declarations for hipfftscaling call for fortran to understand.
   * Makefile needed rocfft-device libraries.
      * Couldn’t install ROCm 5.4 on COS nodes of Lockhart.
      * Had to build rocfft/hipfft from source to build/link the fft compilation units. <br/><br/>


![image](https://user-images.githubusercontent.com/112571800/207702008-acb09e30-002b-4802-9904-d119d55b1d86.png)
                                 <p align="center"> 
   Here’s Your Problem!
   <br/><br/><br/><br/>
   </p>
                                 
                                 
![image](https://user-images.githubusercontent.com/112571800/207704458-5e568663-634e-44c2-b1a5-94eac71a440f.png)
 <p align="center"> 
   Separate plans for forward and inverse fft 
   <br/><br/><br/><br/>
    </p>


![image](https://user-images.githubusercontent.com/112571800/207704506-2bb3eed1-cbd0-4dc1-a59f-590d0c151e7c.png)
 <p align="center"> 
   Inverse fft uses original PlanMany
   <br/><br/><br/><br/>
    </p>

----

### AMD Optimization
* System Details
   * Lockhart node
      * CPU: EPYC 7a53s (Trento)
      * GPU: MI250x (A0)
      * ROCm: 5.4.0
         * Need 5.4.0 for ExtPlanScaleFactor() support
* QE Configuration
   * MPI Ranks: 1
   * Num GPUs: 1
   * Host Threads: 1
   * Atoms: 1 
   * Cutoff: 50 
   * Charge Density: 425 
   * Iterations: 8
* Performance
   * Runtime: 28m 4s wall-time (Figure out how to make text red or resort to bold)
   * FOM: 18012 (Figure out how to make text red or resort to bold)

----

#### AMD Optimization - Timeline Trace
![image](https://user-images.githubusercontent.com/112571800/207700888-7de627d3-9014-4746-8092-b56e966794c0.png)

![image](https://user-images.githubusercontent.com/112571800/207700798-445268ff-abbe-49f7-87d7-3175f1bc036b.png)

----
   
## Lessons Learned
* OpenMP Offload is… fun
* Fortran is… more fun
* Documentation!
* Processes for effective collaborative work
* Compilers are….primadonnas

| | CPU Baseline | AMD Baseline | AMD Optimization |
| --- | --- | --- | --- |
| Wall-Time | 33m 41s | 64m 0s | 28m 4s |
| FOM | 15004 | 7901 | 18012 |

----

## I Learned More Than Expected... Which Is Invaluable To Me
> We went deeper into the weeds than planned… I but loved the adventure
<img align="center" src="https://user-images.githubusercontent.com/112571800/207699432-8e63141e-e86d-4291-b05b-43de17fee640.gif">

----

## Future Work
* Omnitrace to capture work done on Host.
* Saturate work done on device.
* Apply our optimization to other scaling loops in source code. 
* Identify code that’s been cuda-fied in QE vs our development version.
* Continue tracking future FOM values in Grafana.
* Blog post in AMD Lab Notes.
* 15-minute canned tutorial/demo presentation highlighting the optimization work.
* Paper submission in computation chemistry workshop/conference.
* Submission of current/future efforts to scientific journal.
* Partnership between Batteas Group/Center for the Mechanical Control of Chemistry and AMD.






