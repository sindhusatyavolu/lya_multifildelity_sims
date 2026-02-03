# lya_multifildelity_sims
Repository of all codes/helpers to run low resolution Nyx simulations

## Instructions to run a test 64-10Mpc/h sim from scratch to P1D

### 2LPT ICs: 

#### Create config:

Use the conifg file under ICs/params/ or copy one of the ic files from lyssa (they are at /global/cfs/cdirs/desi/users/sindhu_s/codes/ini_conditions_Nyx/lyssa_grid/2lpt_ini/cosmo_grid<grid_number>/ and .ini file), edit the file to change:

- Nmesh and Nsamples to 64
- glasstilefrac to 1
- Boxsize to 10000
- GlassFile location to /global/cfs/cdirs/desi/users/sindhu_s/codes/ini_conditions_Nyx/camb_2lpt_files/dummy_glass_dm_only_from_2comp_64.dat
- File with Transfer to /global/cfs/cdirs/desicollab/users/sindhu_s/codes/ini_conditions_Nyx/lyssa_grid/cosmo_grid<grid_number>/class_cosmo_grid<grid_number>transfer_function_z99.0
- File with Input Spectrum to 
/global/cfs/cdirs/desicollab/users/sindhu_s/codes/ini_conditions_Nyx/lyssa_grid/cosmo_grid<grid_number>/class_cosmo_grid<grid_number>power_spectrum_z99.0
- Outputdir to your scratch folder

#### Run 2lptic :  (recommended to run on interactive node with module load python – I’ve checked that both these run succesfully on nersc-python, which has pynbody v2.0.0)

Usage: srun /global/cfs/cdirs/desi/users/sindhu_s/codes/ini_conditions_Nyx/2LPT/src/2lpt <path_to_your_config_file> 

#### Convert ICs to Nyx format:

Rename xxxx.dat Gadget IC file to xxxx.dat.0  

Usage: srun /global/cfs/cdirs/desi/users/sindhu_s/codes/ini_conditions_Nyx/IC_conversion/convert_ic_gadget_to_nyx.py --from-2lpt <path_to_your_IC_file>

#### Verify ICs

To check the contents of the xxx.nyx file, rename it to xxx.nyx.0 (that’s just how convert_ic_gadget_to_nyx.py is written to work) and do   

/global/cfs/cdirs/desi/users/sindhu_s/codes/ini_conditions_Nyx/IC_conversion/convert_ic_gadget_to_nyx.py --read-only-nyx <path_to_your_Nyx_IC_file>  

I would suggest keeping the .nyx file separate and keeping a copy of it renamed as .nyx.0 for verifying the contents.

### Nyx:

#### Create config:
Use the default config at Nyx/params/ or /global/cfs/cdirs/desi/users/sindhu_s/codes/amrex/Nyx/Exec/LyA/ini_configs/lyssa_low_fid/template_nyx_config.ini  

Edit the following:  

- geometry.prob_hi (boxsize in Mpc) =  10/h  10/h 10/h (h value from IC config)

- amr.n_cell= 64 64 64 

- nyx.particle_init_type = BinaryFile

- nyx.binary_particle_file = .nyx file location 

- nyx.comoving_OmM = value from IC config

- nyx.comoving_h   = value from IC config

- amr.check_file = <outputdir>/chk

- amr.plot_file  = <outputdir>/plt

- nyx.plot_z_values = 6.0 5.0 4.0 3.0 2.0 (for example)

- nyx.analysis_z_values 

- nyx.uvb_rates_file : path to TREECOOL_middle file

- nyx.initial_z = 99.0

- nyx.final_z = 2.0

- amr.checkpoint_files_output = 1 # if you want checkpoint files which can be used to restart the simulation from a given checkpoint redshift

- amr.check_int         = 100 # saves checkpoint for every 100 timesteps

- amr.checkpoint_nfiles = 64 # number of checkpoint files written in parallel

- amr.plot_files_output = 1 # should be 1 to save outputs i.e;  plt snapshots

- nyx.h_species = 0.76

- nyx.he_species = 0.24

- amr.derive_plot_vars=particle_mass_density particle_count particle_x_velocity particle_y_velocity particle_z_velocity

- amr.checkpoint_files_output = 1


Make sure the cosmological parameters are same in the IC config and Nyx config

initial z should be the same as the z at which ICs were generated (z=99 if you are using camb files from lyssa). You can choose the number of redshifts to be saved, and the location of outputs.

#### Run Nyx:
Usage: 
This is a GPU executable, so run on an interactive GPU node.

Request an interactive node 
salloc -N 1 -n 4 -c 32 --account desi --qos interactive -C gpu -t 01:00:00 --gpus-per-task=1
(1 GPU node has 4 gpus, 4 tasks per node and 32 cpus per task which means we are maximising 128 cpus)

On the interactive node at NERSC:
- Export sundials:
export LD_LIBRARY_PATH=/global/cfs/cdirs/nyx/sundials_shared/sundials/instdir/lib:$LD_LIBRARY_PATH
- Run nyx using my executable
srun /global/cfs/cdirs/desi/users/sindhu_s/codes/amrex/Nyx/Exec/LyA/Nyx3d.gnu.TPROF.MTMPI.OMP.CUDA.ex <path_to_config>

Hopefully your Nyx run was successful!
You can look at the plt snapshots and plot them using yt. For details, [check here](https://warpx.readthedocs.io/en/latest/dataanalysis/yt.html)

#### Convert Nyx output to HDF5 (for gimlet):

Under Utils, you will find the code to convert plt folders to HDF5 files. 

Usage:

On an interactive CPU node: salloc -N 1 -n 4 -c 32 --account desi --qos interactive -C cpu -t 01:00:00 

srun /global/cfs/cdirs/desi/users/sindhu_s/codes/amrex/Nyx/Util/Converters/Plotfile2HDF5_grids/convert3d.gnu.x86-milan.PROF.MPI.ex input_path=<path_to_plt_folder> output_path=<path_to_output+name_of_file(e.g., /pscratch/pltxxxx.hdf5)>

### Gimlet 

Gimlet (CPU version only):

No config, give location of HDF5 file as input through command line. Optional arguments to be used if you want mean flux rescaling. 

On an interactive CPU node: salloc -N 1 -n 4 -c 32 --account desi --qos interactive -C cpu -t 01:00:00 

module load cray-fftw

module load cray-hdf5-parallel

srun /global/cfs/cdirs/desi/users/sindhu_s/codes/gimlet2_MW/apps/lya_all_axes_rhoT/lya_all_axes.ex -f <mean_flux_rescale_value> -d xyz <path_to_hdf5_file> <output_directory>





