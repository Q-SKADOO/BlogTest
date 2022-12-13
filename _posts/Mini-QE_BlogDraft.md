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
  
## Extract Data File (Files need to be updated for correct device data validation_
* Input Data
`tar -xvf data.tar.gz` //PlanScaleBeforeExecz2z_fd
* Validation Data
`tar -xvf validation.tar.gz` //PlanScaleAfterDo_fd.txt

## Run
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
