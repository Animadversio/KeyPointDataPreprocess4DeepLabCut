# ImageJ Key Point Data Preprocess for DeepLabCut Training
A short script to re-format the key point coordinate data got from ImageJ (txt) into format (hdf5) required by DeepLabCut to make training dataset. 

DeepLabCut is a powerful tool for automatic key point detection in video, thus a game changing tool for behavior quantification. But currently the GUI in DeepLabCut is not very stable, this script enable a workaround of frame extraction and frame annotating procedure. We could do frame extraction and annotation in `ImageJ`, using a plugin called `Point Picker`. And using this script to reformating the data into `hdf5` required by deeplabcut for further training set construction. 

**Tools**

* [`DeepLabCut`](https://github.com/AlexEMG/DeepLabCut)
* [`ImageJ`](https://imagej.nih.gov/ij/download.html): traditional image analysis software for scientific image analysis. It has pretty robust and easy to use GUI and lots of plugins. It can transform the `.avi` movie into frame sequences in many formats, and can also take screenshots and save as image files. Thus, it's suitable for **Frame Extraction**
    * [`Point Picker`](http://bigwww.epfl.ch/thevenaz/pointpicker/):  An ImageJ plugin developped by EPFL scientists which can pick up to 1024 points in an image, and can deal with image sequence well. Thus it's suitable to **Label Frames** 
* Interface (data format transforming) script: (The only part I write) Transform the output file of `Point Picker` to the required formats of `deeplabcut.create_training_dataset(.)`. 
    * [`Pandas`](https://pandas.pydata.org/pandas-docs/stable/): `DataFrame` in `Pandas` is the intermediate data structure I used to do the transformation. 

## Workflow combining `ImageJ` and `DeepLabCut`
Here is the complete workflow / protocol we can follow when starting a behavioral quantification project. 

* **Video recording**
* Create a project in `DeepLabCut`: 

    - `config_path =deeplabcut.create_new_project('Name of the project','Name of the experimenter', ['Full path of video 1','Full path of video2'], working_directory='Full path of the working directory',copy_videos=True/False)`
* **Frame Extraction**: 
    - We can use the automatic extraction tool in `deeplabcut` as baseline: `deeplabcut.extract_frames(config_path,'automatic','kmeans', crop=True, checkcropping=True)` (which is relatively robust)
    * And then, supplement it with manual selection from `ImageJ` 
    * Open the video with `ImageJ`. Slide and select the key frames and saveas `png` files in `ImageJ`
    * Note the training set can consist of frames of different size, cropped and uncropped mixed together. 
* **Label the Frames**:
    - We can use the GUI tool in `deeplabcut.label_frames(config_path)`
    - Or
    - Use the `ImageJ`, `Point Picker` plugin
        + `Import>Image Sequence` select the folder of the extracted frames
        + Open `Point Picker` and mark the key points in sequence from the first image. 
        + Export the coordinates data with `Import/Export` tool. save the `data.txt` file
    - Convert the `txt` date into `csv` and `hdf5`
* **Check the annotation**: 

    - `deeplabcut.check_labels(config_path)`
* **Create Dataset**: 

    - `deeplabcut.create_training_dataset(config_path,num_shuffles=1)`
* **Train Network**: 

    - `deeplabcut.train_network(config_path,shuffle=1)`
* **Evaluate Network**: See the training loss and test loss

    - `deeplabcut.evaluate_network(config_path,shuffle=[1], plotting=True)`
* **Video Analysis**: 
``` python
deeplabcut.analyze_videos(config_path,['/analysis/project/videos/reachingvideo1.avi'], shuffle=1, save_as_csv=True)
deeplabcut.create_labeled_video(config_path ['/analysis/project/videos/reachingvideo1.avi','/analysis/project/videos/reachingvideo2.avi'])
deeplabcut.plot_trajectories(config_path,['/analysis/project/videos/reachingvideo1.avi'])
```

