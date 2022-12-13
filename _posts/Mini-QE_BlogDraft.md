### This is V1 of draft

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

/*! @brief Set scaling factor.
 *
 *  @details hipFFT multiplies each element of the result by the given factor at the end of the transform.
 *
 *  The supplied factor must be a finite number.  That is, it must neither be infinity nor NaN.
 *
 *  This function must be called after the plan is allocated using
 *  ::hipfftCreate, but before the plan is initialized MakePlanMany. (Currently the only Make Plan function that supports the scaling API)
 *
 *  */
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

![image](https://user-images.githubusercontent.com/112571800/207438905-27e9776b-5498-4277-a6b4-99fda1d121b6.png)

