# ABTAS

Aquatic Bio-Telemetry Analysis Software (ABTAS) for use in removing false positive and overlap detections from aquatic radio telemetry projects.

# Introduction
ABTAS, Kleinschmidt’s radio telemetry analysis software is comprised a suite of Python scripts and an importable python module (abtas.py).  Each of the scripts carries out a task one may have when analyzing radio telemetry data, including: creating a standardized project database, importing raw data directly from receiver downloads, identifying and removing false positive detections, cross validating and assessing the quality of training data, producing an array of standard project statistics (no. recaptured at receiver, etc.), removing overlap between neighboring receivers with larger detection zones, and producing data appropriate for analysis with Competing Risks and Mark Recapture (Cormack Jolly Seber and Live Recapture Dead Recover) methods.  In future iterations of the software, we will replace the functional sample scripts with a Jupyter Notebook to guide users through training, classification, and data preparation.
The scripts in their current form take advantage of the multiprocessing module when necessary to make the data filtering process more efficient.  When activated, all system resources will be utilized unless otherwise directed by the end user.  It is not recommended to create more processes than CPUs in your computer, please understand the limitations of your machine before proceeding.  
