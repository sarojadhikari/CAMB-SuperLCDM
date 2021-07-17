Assuming that `camb` and `cobaya` are setup with [necessary modifications](README-SuperLCDM.md),
the following steps can be followed to run MCMC chains using `cobaya`.

  1. Generate `yaml` file using `cobaya-cosmo-generator` as the starting point (or start with `slcdm0.yaml` file) and modify accordingly. The `slcdm0.yaml` file includes both `A0` and `eps` parameters as free parameters to be sampled.
  2. Make sure to put the correct path to the modified `camb` in your `yaml` file.
  3. Run chains using `cobaya-run`:
 ```bash
      nohup mpirun -n N cobaya-run slcdm0.yaml -p ../packages -o chains/slcdm* > out_slcdm.log &
 ```
 where, 
   * `N` is the number of chains and MPI threads (make sure that you set `OMP_NUM_THREADS` appropriately) 
   * `-p ../packages` specifies where the cosmology (planck etc) codes are installed automatically using `cobaya`
         (see cobaya documentation) 
   * `-o chains/slcdm*` means that the chains and other output from `cobaya` is put in the chains subfolder in the
         current folder with the base name `slcdm*` (`*` should replace with what you desire the base name of the chains to be)
       
    
