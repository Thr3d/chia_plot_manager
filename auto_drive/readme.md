<h2 align="center">
  <a name="chia_drive_logo" href="https://github.com/rjsears/chia_plot_manager"><img src="https://github.com/rjsears/chia_plot_manager/blob/main/images/chia_plot_manager_v2.png" alt="Chia Plot Manager"></a><br>
Auto Drive - Automatic Drive Formatting, Mounting & Chia Config Updating!
  <p align="center">

  </h2>
  </p>
  <p align="left"><font size="3">
  
As the title implies, I designed this script to help with the adding of new hard drives to your Chia Harvester/NAS. This is a command line drive interactive python script. Please see the top fo the script for more information. Also remember that running `auto_drive.py` invokes `fdisk` and `mkfs`. Make sure the user that you are running the
script as somone who has the ability to execute those commands. `sudo` should work in this situation.
  
  Hopefully, this will be useful for someone!
  
  
<h2 align="center">
  <a name="chia_auto_drive_screen" href="https://github.com/rjsears/chia_plot_manager/tree/main/auto_drive"><img src="https://github.com/rjsears/chia_plot_manager/blob/main/images/chia_auto_drive_output.png" alt="Chia Auto Drive"></a><br>
  <br><br>
</h2>  
  <h3>Installation and usage Instructions:</h3>
  
  1) Download a copy of this script and the `get_drive_uuid.sh` shell script and place it in your working directory.
  
  2) Edit the auto_drive.py script and alter the line pointing to your chia configuration file:<br>
     `chia_config_file = '/root/.chia/mainnet/config/config.yaml'`
  
  3) Alter the path_glob line to match your mount point directory structure. This script is specifically set up and tested
     with the directory structure that I have laid out in the main readme file for chia_plot_manager. It is possible that 
     it will work with other structures but it is way beyond my capability to test for every possible directory structure
     combination. I would recommend reading and understanding what the `get_next_mountpoint()` function does and then 
     see if it will work with your directory structure.<br>
     `path_glob = '/mnt/enclosure[0-9]/*/column[0-9]/*`

  4) Alter the following line to point to where you have installed the `get_drive_uuid.sh` shell script. Leave script name as well.<br>
     `get_drive_uuid = '/root/plot_manager/get_drive_uuid.sh'` 
  
  5) `chmod +x` both the auto_drive.py script and the `get_drive_uuid.sh` script.
  
  6) Change to the directory where you have installed this script and `./auto_drive.py` If the script finds any new drives it will
     then prompt you to accept the drive and mountpoint it plans to use. Once the drive has been readied for the system and mounted
     it will ask you if you want it added to your chia configuration file.
  
  <br><hr>
  <h3>Requirements........</h3>
  
  The only requirements beyond standard linux tools are the following:
  
  <ul>
  <li><a href="https://pypi.org/project/natsort/">Natsort (7.1.1)</a> - Used for natural sorting of the mount points</li>
  <li><a href="https://pypi.org/project/PyYAML/">PyYAMLK</a> - (Optional, you could use any YAML Python Library)</li>
 </ul>
  
  <hr>
  
  <h3>Notes......</h3><br>
  
  There are a couple of things to remember with `auto_drive.py`. First, in order to protect any possible data on drives, we will only 
  look for drives that <em>do not</em> include a partition already on the drive. If you are reusing drives, and they have a partition
  already on the drive, `auto_drive.py` <em>will not</em> select any of those drive to work on. The best way to `reuse` a drive is to
  manually `fdisk` the drive in question, delete any existing partitions, and then `auto_drive.py` will then be able to use those
  drives. <br>
  <br>
  Second, I have determined that drives that have previously had VMFS partitions on them require special handling. `SGDisk` does not
  appear to have the ability to overwrite that partition information. In fact, `fdisk` seems to have an issue as well. After multiple
  attempts to figure this litte problem out, I found that this is the best solution:<br><br>
  
  1) Using `fdisk`, access the device in question.
  2) Delete any existing partition and issue the `w` command.
  3) Rerun the `fdisk` command on the disk, utilize the `g` command to create a new GPT signature.
  4) Create a new partition by using the `n` command.
  5) When you go to save the partition with the `w` command, you should get a notice that there is an existing VMFS partition.
  6) Answer `y` to this question and issue the `w` command, exiting `fdisk`.
  7) Once again, rerun `fdisk` and delete this new partition and issue the `w` command. 
  8) You should now be ready to use `auto_drive.py` with that drive. 
  
  Finally, `auto_drive.py` will not <em>automatically</em> add your new mountpoint to your chia config file. It will first ask
  you if you want it to do this. This is just an added step in the process to make sure you are aware of what the script is 
  doing before it does it. If you choose not to have `auto_drive.py` add the mountpoint to your chia config, be sure to do it
  on your own otherwise your harvester will not see any plots on that mountpoint.
  <br><hr>
  
  <h3>Troubleshooting Auto_Drive......</h3><br>
  
  As I noted above, the main issues I have run into running `auto_drive.py` on my and my friend's systems revolve around the
  way a `used` drive may have been partitioned. In the event we cannot correctly manage a drive, `auto_drive.py` may leave 
  your system in a state in which you want to correct to retry the operation. For example, if the drive is not partitioned
  correctly due to a `VMFS`, `ZFS`, or other type of partition, the `UUID` will be incorrect causing `auto_drive.py` to report an error mounting
  the drive due to an incorrect `uuid`. In this case, you would need to:
  
  1) Correct the partitioning problem and verify that the partition has been removed with `lsblk`
  2) Remove the incorrect entry from `/etc/fstab`. Any entries added by `auto_drive.py` will be noted as such.
  3) Run `fdisk /dev/xxx` against the drive in question. Remove `all` partitions and `w`.
  4) Run `fdisk /dev/xxx` (yes again), create a new partition and `w`, answer `yes` when it warns you about existing signature.
  5) Rerun `auto_drive.py` again allowing it to reselect the drive in question.
  
  I am currently working on a solution to this issues, but I am having difficulty finding a suitable tool to wack the drive wholesale.


  Beyond that, other issues could be related to the person running the script. Make sure you have sufficient user privileges
  to run the following commands:

  1) mkfs.xfs or mkfs.ext4
  2) get_drive_uuid.sh
  3) sgdisk
  4) mount (although we do use the `user`option on our mount point so this should <em>never</em> be an issue).
  5) Make sure the user running the script has write access to your chia configuration file.
 
  One last thing, if you get an error adding the mountpoint to your chia configuration file and this is the `first`
  drive you have added, please verify that your plot_directories entry in your chia configuration file looks like
  the following. If it does not have the trailing `[]` it will not work:
  
  `plot_directories: []`
  
  
 
 
  
  <br>
  <h3>Enjoy!!</h3>
