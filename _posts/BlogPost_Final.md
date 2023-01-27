### This is V4 of draft (BlogPost)...Polished Version... Questions: Any copyright issues with the images? Ok to mention CUDA in comp framework?



# Fortran, OpenMP, Quantum Espresso… Oh My <br> <sub> (Quentarius Moore - Coop Presentation Fall 2022)</sub>

## Overview
* Acknowledgements
* Introduction + About Me
* Real World Science Background
* Computational Framework
   * Quantum Espresso
   * PWscf
* Porting Quantum Espresso
   * CPU Baseline
   * AMD Baseline
   * AMD Optimization
* Lessons Learned
* Future Work

----

## Acknowledgents

### AMD Team


   
<p float="left" align="center">
  <img src="https://user-images.githubusercontent.com/112571800/207923399-164a2522-4a43-438e-94dd-8835c73fb6b7.png" width="15%" />
  <img src="https://user-images.githubusercontent.com/112571800/207923435-3a2c6398-5c60-455b-a8cd-06314d4d0e5b.png" width="100" /> 
  <img src="https://user-images.githubusercontent.com/112571800/207923475-cd30932a-93d0-4a21-925c-492d8e3f943b.png" width="100" />
  <img src="https://user-images.githubusercontent.com/112571800/207923574-70b1de03-2fd4-4a8e-b196-5bea5f7dc2d4.png" width="100" />
</p><br>
<p align="center">
Jose Noudohouenou<font size ="5">OpenMP expert</font> | Ossian O’Reilly<sub>Quantum Espresso expert</sub> | Steve Leung<sub>roc/hipFFT expert</sub> | Justin Chang<sub>OpenMP Offload + Fortran expert</sub>
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
### Home Team
* Dr. James D. Batteas
* The Batteas Research Group
* NSF Center for the Mechanical Control of Chemistry
* Texas A&M University
* Collaborators at Sandia National Laboratories (Contract:DE-NA0003525)

![image](https://user-images.githubusercontent.com/112571800/207920241-a4fa043d-18bd-482b-acc2-77a9fc970baf.png)

<p float="left" align="left">
<img src="https://user-images.githubusercontent.com/112571800/207920448-55c06ae7-4e50-448a-b151-7f4bf64f2432.png" width="32%">    <img src="https://user-images.githubusercontent.com/112571800/207920075-84385af0-8e70-4195-8b88-83597ebea9fb.png" width="32%"><img src="https://user-images.githubusercontent.com/112571800/207919937-2760f78b-dbd1-455b-87f0-cde50e4fafdc.png" width="32%">
</p>



<br><br><br>

----
<br><br><br>
## Introduction + About Me <br>
![image](https://user-images.githubusercontent.com/112571800/207917727-d8ab2831-18bc-4b82-8144-cb6ddc0f4895.png)

<br>
<p float="left" align="center">
<img align="center" width="32%" height="32%" src="https://user-images.githubusercontent.com/112571800/207913106-3f8fe0e2-f63d-49dc-bd97-f171cdd6f79f.png"> <br><br><br>
</p>

<br>
<p float="center" align="center">
<img align="center" width="32%" height="32%" src="https://user-images.githubusercontent.com/112571800/207913177-051a0ab8-e3d1-4b86-9da3-36191ef8f3cc.png"> <br><br><br>
</p>
<p align="center">
Service to Others
</p>
<br>
<p float="left" align="center">
<img align="center" width="32%" height="32%" src="https://user-images.githubusercontent.com/112571800/207913218-402d9954-5251-4ed2-a1d5-01c4a3284a74.png"> <br><br><br>
</p>
<p align="center">
Rocket League with Mentor...Outside of Work Hours of course :)
</p>
<br>
<p float="left" align="center">
<img align="center" width="32%" height="32%" src="https://user-images.githubusercontent.com/112571800/207917227-f6ab15cf-a053-4e1d-ba96-b8164ea5d2d4.png"> <br><br><br>
</p>
<p align="center">
AMD Photo Forum
</p>
<br>
<p float="left" align="center">
<img align="center" width="32%" height="32%" src="https://user-images.githubusercontent.com/112571800/207917345-7b96516e-962d-4f08-881f-4fd4de3b5066.png"> <br><br><br>
</p>
<p align="center">
AMD 2022 Fall Chess Tournament
</p>
<p align="center">
<b>“If I have seen further, it is by standing on the shoulders of giants”</b>
</p>



----

## Real World Science Background
![image](https://user-images.githubusercontent.com/112571800/207719397-badaf872-1a7b-4f66-bfbe-285ab52e79cd.png)
<p float="left" align="center">
<sub>Overview of structural, mechanical, electrical, thermal, and chemical properties of 2D materials that are relevant to tribological performance</sub><br>
</p>

Two-Dimensional (2D) Nanomaterials
   * Unique properties and applications
   * Flexible Electronics
   * Energy Storage
   * Medicine
   * Sensors

<p float="left" align="center">
<img align="center" src="https://user-images.githubusercontent.com/112571800/207719114-174713e9-bb7e-4d63-ae62-00b02b4a957e.png">
   </p>
 <p align="center"> 
<sub><b>The Mariner 9 planetary space probe</b></sub>
   </p>


<p align="center"> 
   <b>2D materials possess exceptional frictional properties for use as solid dry lubricants </b>
   </p>

* Zhang, S; et al. Tribology of two-dimensional materials: From mechanisms to modulating strategies. Mater. Today. 2019, 26, 67-86 <br>
* NASA Dry Lubricant Smooths the Way for Space Travel, Industry https://spinoff.nasa.gov/Spinoff2015/ip_7.html 2015

----

### Real World Science Background - Example
<p float="left" align="center">
<b>Molybdenum disulfide (MoS<sub>2</sub>)</b> <br>
</p>

<p float="left" align="center">
<img src="https://user-images.githubusercontent.com/112571800/207717655-6d7b02b7-1800-4403-bf9f-f213b911171b.png">
<img src="https://user-images.githubusercontent.com/112571800/207717689-8140dbd5-9a31-436d-bee2-9b4f12a67453.png">
<img src="https://user-images.githubusercontent.com/112571800/207717703-1f71246b-a31b-4ae1-91d3-2a0c8850ffb8.png">
</p> <br>

* Exciting mechanical, electronic and frictional properties.

* Promising potentials for solid lubricants, ion batteries, low power transistors, optoelectronic 
* Stable against reactions with environmental species.
* Three atomic layers hexagonal
<p float="left" align="center">
Challenges
</p>
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
      * CUDA
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
* Potentially helpful in future debug tasks. <br>
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
      * This can be done natively using a loop.
* Scaling loop runs natively on the device and does so quickly.

```fortran
       tscale = 1.0_DP / DBLE( nx * ny * nz )
!$cuf kernel do(1) <<<*,(16,16,1),0,stream>>>
        DO i=1, ldx*ldy*ldz*howmany
           f_d( i ) = f_d( i ) * tscale
        END DO
```



<p align="center">
AMD (Fortran/OpenMP offload)
   </p>
   
* Scaling loop runs on device using OpenMP offload
   * Timeline trace confirms scaling loop is running on device.
   * Perhaps it’s operating on host data f_d, hence the poor perf?
   * Lots of questions and possible fixes.
      * Time for miniApp!!!

```fortran
istat = hipfftPlanMany( hipfft_plan_3ds( icurrent), RANK, c_loc(FFT_DIM), &
                            c_loc(DATA_DIM), STRIDE, DIST, &
                            c_loc(DATA_DIM), STRIDE, DIST, &
                           HIPFFT_Z2Z, BATCH )
```

```fortran
!$omp target data use_device_ptr(f_d)
istat = hipfftExecZ2Z( hipfft_plan_3d(ip), c_loc(f_d), c_loc(f_d), HIPFFT_FORWARD )
!$omp end target data
```

```fortran
tscale = 1.0_DP / DBLE( nx * ny * nz )
!$omp target parallel do
   DO i=1, ldx*ldy*ldz*howmany
      f_d( i ) = f_d( i ) * tscale
   END DO
```






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

```c++
//Start Scaling Timer
hipEvent_t start_scaling, stop_scaling;
hipEventCreate(&start_scaling);
hipEventCreate(&stop_scaling);

// Create plan
printf("Creating HIPfft plan...\n");
hipfftHandle hipfft_plan_3d;      //= hipfft_params::INVALID_PLAN_HANDLE;
hipfftResult hipfft_rt = hipfftCreate(&hipfft_plan_3d);
hipfftPlan3d(&hipfft_plan_3d, 128, 128, 200, HIPFFT_Z2Z);

roctxRangePush("omp_target_region");

printf("Beginning omp targets...\n");
#pragma omp target enter data map(to:deviceData)
#pragma omp target data use_device_ptr(deviceData)
hipEventRecord(start_fft, 0);

// Call hipFFT function of interest
hipfftExecZ2Z( hipfft_plan_3d, deviceData, deviceData, HIPFFT_FORWARD );
hipEventRecord(stop_fft, 0);

hipDeviceSynchronize();

double tscale = 1.0f/(128*128*200);
std::cout << tscale << std::endl;

hipEventRecord(start_scaling, 0);
roctxRangePush("Scaling_region");

#pragma omp teams distribute parallel for num_threads(1024) num_teams(60)
for (int i = 0;i < (128*128*200); i++){
   deviceData[i].x = deviceData[i].x * tscale;
}

roctxRangePop();
hipEventRecord(stop_scaling, 0);

#pragma omp target update from(deviceData)
#pragma omp target exit data map(delete:deviceData)
roctxRangePop();
```


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

```fortran
ELSE IF( isign > 0 ) THEN
print *, 'Here is your problem.'

!$omp target data use_device_ptr(f_d)
istat = hipfftExecZ2Z( hipfft_plan_3d(ip), c_loc(f_d), c_loc(f_d), HIPFFT_BACKWARD )
!$omp end target data

call fftx_error__(" fft_scalar_hipFFT: cfft3d_omp ", " hipfftExecZ2Z failed ", istat)

CALL hipCheck(hipDeviceSynchronize())
```


   

```f90
! Plan used by scaling API
IF( hipfft_plan_3d( icurrent) /= c_null_ptr ) THEN
   istat = hipfftDestroy( hipfft_plan_3d(icurrent) )
   call fftx_error__(" fft_scalar_hipFFT: cfft3d_omp ", " hipfftDestroy failed ", istat)
ENDIF

! Plain plan used by inverse transform
IF( hipfft_plan_3ds( icurrent) /= c_null_ptr ) THEN
   istat = hipfftDestroy( hipfft_plan_3ds(icurrent) )
   call fftx_error__(" fft_scalar_hipFFT: cfft3d_omp ", " hipfftDestroy failed ", istat)
ENDIF
```
                                 


```fortran
! MakPlanMany initializes the scaled plan
istat = hipfftMakePlanMany( hipfft_plan_3d(icurrent), RANK, c_loc(FFT_DIM), &
                             c_loc(DATA_DIM), STRIDE, DIST, &
                             c_loc(DATA_DIM), STRIDE, DIST, &
                              HIPFFT_Z2Z, BATCH, c_null_ptr )
call fftx_error__(" fft_scalar_hipFFT: cfft3d_omp ", " hipfftMakePlanMany failed ", istat)

! PlanMany initializes plain plan       
istat = hipfftPlanMany( hipfft_plan_3ds( icurrent), RANK, c_loc(FFT_DIM), &
                            c_loc(DATA_DIM), STRIDE, DIST, &
                            c_loc(DATA_DIM), STRIDE, DIST, &
                           HIPFFT_Z2Z, BATCH )
```


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
> We went deeper into the weeds than planned… but I loved the adventure
<img align="center" src="https://user-images.githubusercontent.com/112571800/207699432-8e63141e-e86d-4291-b05b-43de17fee640.gif">

----

## Future Work
* Omnitrace to capture work done on Host.
* Saturate work done on device.
* Apply our optimization to other scaling loops in source code. 
* Continue tracking future FOM values in Grafana.
* 15-minute tutorial/demo presentation highlighting the optimization work.
* Paper submission in computation chemistry workshop/conference.
* Submission of current/future efforts to scientific journal.
* Partnership between Batteas Group/NSF Center for the Mechanical Control of Chemistry and AMD.





