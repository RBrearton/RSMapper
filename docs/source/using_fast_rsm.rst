How to use fast_rsm
===================

If you on are site at Diamond




Skip to sidebar
Skip to main content
Skip to breadcrumbs
Skip to search
Diamond Confluence

    Spaces
    People
    Calendars
    Analytics
    Create
    Create

    Help
    0
    Profile picture for rpy65944


I07 Surface and Interface Diffraction

Reciprocal space mapping with fast_rsm

    Edit
    View inline comments

    Save for later
    Watching
    Share

    Pages

    Data reduction and analysis 

    Analytics

    Created by Richard Brearton [X], last modified by Philip Mousley on Jan 15, 2025

Current issues with code   - table to keep track of bugs with the fast_rsm code

Example datasets for testing and benchmarking fast_rsm mapper

version history
Steps to use fast_rsm 

step-by-step instructions on how to use fast_rsm at i07 are detailed below. If you are entirely unfamiliar with linux/computers and you can't follow along, ask a beamline member of staff or email philip.mousley@diamond.ac.uk



Prior to loading and using the fast_rsm package, you will need to setup your SSH connection to the wilson server which is used to submit jobs to the computer cluster.

This is done with the following steps, note that commands to enter are the text after the $ symbol

    log into a workstation with your fedID
    Create a fresh ssh key with the commands:
        use command                 $   ssh-keygen
        will ask to provide a name for the key pair,  go with default option id_rsa             $   /home/<fed12345>/.ssh/id_rsa
        enter a new passphrase twice, which can be something really simple, you will need to remember it to use it in a few steps  time. Note the command line remains empty as you type the password. 
    authorise the use of the key by adding it to their  trusted keys         $       cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
    alter SSH settings to read-only with the following settings 
        $  chmod 700 ~/.ssh
        $  chmod 600 ~/.ssh/*
    Connect to wilson  $   ssh wilson 
    enter  ssh passphrase created in earlier step, note this is NOT your fedID password
    if all has gone well then you should see this screen : 
     
    Check you can connect without needing password -  type   $ exit
    Type  $   ssh wilson 
    This should go straight to the welcome text above, without the need to input a passphrase
    Once this is successfully connecting to the wilson using SSH you are ready to follow the steps

    Login to a diamond linux machine. If you're on the beamline, the beamline scientists will show you which machine uses linux. If you're at home, use nomachine [https://www.diamond.ac.uk/Users/Experiment-at-Diamond/IT-User-Guide/Not-at-DLS/Nomachine.html] to access a linux workstation with access to the diamond servers.
    type "module load fast_rsm" into the terminal 
     If this is your first time using the fast_rsm mapper, type "firstrun" to setup a folder in your home directory
    type "activate" into the terminal to activate the fast_rsm conda environment


    if you need a mask file for your data, make a mask by typing "makemask -dir  path/to/experiment/directory   -s   ##scan##number#" to open up the mask GUI. Save the created mask and note down the file path   ###/###/#####.edf
    if you do not have an experiment file, make an experiment setup file by typing "makesetup" to open up a template experimental file, edit with your experimental information. this will include
        edfmaskfile   = path to the mask you just created.
        local_output_path
        local_data_path
        beam_centre
        detector_distance
        ensure your process_outputs are correct - explained lower down 

There is a very important switch that changes the operating mode of fast_rsm: map_per_image. map_per_image defaults to False, in which case all of the scans will be combined into a single volume in reciprocal space. If map_per_image is set to true, then a very large amount of information will be saved per image. Generally speaking, if map_per_image is set to true and you are mapping 1GB of images.

If map_per_image = True, then, for each image in the scans selected, the following will be saved:

    A binned reciprocal space map, of exactly the same form as those generated when map_per_image is false. Each binned reciprocal space map will have the size set by 
    Every q-vector for every pixel in the image.
    Every intensity obtained for every pixel in the image, without polarisation correction being applied.
    Every intensity obtained for every pixel in the image, with  polarisation correction applied.

Because map_per_image stores every q-vector for every pixel in the image, the complete scattering information is saved. From this, exact maps to e.g. I(Q) and Qxy-Qz can be computed. 

    Process_outputs explained

     'full_reciprocal_map'

    calculates a full reciprocal space map combining all scans listed into a single volume. Use this option for scan such as crystal truncation rod scans, fractional order rod scans, or  in-plane HK scans, 


     'pyfai_qmap'  

    calculates 2d q_parallel Vs q_perpendicular plots using pyFAI. Use this options for GIWAXS measurements either with a static detector or a moving detector.


    'pyfai_ivsq'

      calculates 1d Intensity Vs Q using pyFAI. Use this options for GIWAXS measurements either with a static detector or a moving detector.


    ***DEPRECATED OPTIONS***

     'pyfai_2dqmap_IvsQ' - use parallel multiprocessing to calculate both 2d Qpara Vs Qperp map, as well as 1d  Intensity Vs Q integration - both using pyFAI package


    'large_moving_det' 

    utilise MultiGeometry option in pyFAI for scan with a moving detector and a large number of images (~1000s), outputs: I, Q, two theta, caked image,Q_para Vs Q_perp

    'curved_projection_2D'   (use 'large_moving_det' option instead)

    this projects a series of detector images into a single 2D image, treating the images as if there were all from a curved detector. This will give a projected 2d image. This should only be used when a detector has been moved on the diffractometer arm during a scan, and the images need to be combined together. NOTE: this will not work on a continuous scans with ~1000s of images - for these scan types use the 'large_moving_det' option. 


     'pyfai_1D' - ( use 'pyfai_2dqmap_IvsQ' option instead)

    Does an azimuthal integration on an image using PONI and MASK settings described in corresponding files. This can handle a short series of images (~50) for individual integrations. If used in combination with 'curved_projection_2D', this will simply integrate the large projected image, and not the series of small images. 


    'qperp_qpara_map' - ( use 'pyfai_2dqmap_IvsQ' option instead)

    projects GIWAXS image into q_para,q_perp plot.  NOTE: Similar to 'curved_projection_2D', this will not work with 1000's of images - for these scans use the 'large_moving_det' option.

    Then save this exp_setup.py file  noting the path 
    Usually the default calc_setup.py file that contains the calculation settings will be suitable, however if a bespoke calc_setup.py is needed, copy over the calc_setup.py file in the fast_rsm/CLI/i07  folder and edit accordingly. contact beamline staff or i07 data analysis scientist for guidance on this.

    run the processing by typing  "process_scans -exp  'path/to/exp_setup.py' -s scan-numbers-to-be-mapped
        e.g.   process_scans  -exp   /home/rpy65944/fast_rsm/example_exp_setup.py -s 441187 441188
    alternatively use the -sr option to define an evenly spaced range of scans using the format [[start,stop,stepsize]]  note  double brackets are needed even when specifying only one range
         -sr [[41187, 441189,1]]
    use a list of lists to define several sets of evenly spaced scans using the format [[start1,stop1,stepsize1],[start2,stop2,stepsize2]],  where the ranges are inclusive i.e. the stop value is the final scan in the range which you want analysed
         -sr [[41187, 441189,1] ,[41192, 441195,1]]


Submitted batch job ######   --> this line is printed if job successfully submitted to cluster

Job submitted, waiting for SLURM output.  Timer=## --> this line checks for a new SLURM output file every 5 seconds, current time limit of 250 seconds


Slurm output file: /path/to/home/fast_rsm//slurm-#####.out  --> if new SLURM output file is found within timer limit, output file path


***********************************
 ***STARTING TO MONITOR TAIL END OF FILE, TO EXIT THIS VIEW PRESS ANY LETTER FOLLOWED BY ENTER****   --> monitoring of tail end of slurm out file to monitor progress of calculation

*********************************** 


 










