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

To check the contents of the xxx.nyx file, rename it to xxx.nyx.0 (that’s just how convert_ic_gadget_to_nyx.py is written to work) and /global/cfs/cdirs/desi/users/sindhu_s/codes/ini_conditions_Nyx/IC_conversion/convert_ic_gadget_to_nyx.py --read-only-nyx <path_to_your_Nyx_IC_file>
I would suggest keeping the .nyx file separate and keeping a copy of it renamed as .nyx.0 for verifying the contents.

