[![View imtool3D_td on File Exchange](https://www.mathworks.com/matlabcentral/images/matlab-file-exchange.svg)](https://fr.mathworks.com/matlabcentral/fileexchange/74761-imtool3d_td)  
# imtool3D
This is an image viewer designed to view a 3D stack of image slices. For example, if you load into matlab a DICOM series of CT or MRI images, you can visualize the images easily using this tool. It lets you scroll through slices, adjust the window and level, make ROI measurements, and export images into standard image formats (e.g., .png, .jpg, or .tif) or the 3D mask as NIFTI file (.nii). Use imtool3D_nii to load NIFTI files.
This tool is written using the object-oriented features of matlab. This means that you can treat the tool like any graphics object and it can easily be embedded into any figure. So if you're designing a GUI in which you need the user to visualize and scroll through image slices, you don't need to write all the code for that! Its already done in this tool! Just create an imtool3D object and put it in your GUI figure.

<p align="center">
  <img src="Capture.PNG" width="600">
</p>
  
imtool3D is used heavily by several other projects:
* [qMRLab](https://github.com/qMRLab/qMRLab)
* [imquest](https://gitlab.oit.duke.edu/railabs/SameiResearchGroup/imquest)
* [lesionTool](https://gitlab.oit.duke.edu/railabs/SameiResearchGroup/lesionTool)

# Documentation

* [Dependencies](#dependencies)
* [Demo](#demo)
* [Tutorial](#tutorial)
* [Authors](#authors)

# Dependencies
* Matlab's image processing toolbox (ROI tools are disabled otherwise)
* [dicm2nii](https://github.com/xiangruili/dicm2nii) (if NIFTI images are used)

# Demo
[Brain tumor segmentation](https://www.dailymotion.com/embed/video/x7okm8h) using `imtool3D_nii_3planes.m` or `imtool3D_3planes.m`  
[Integration in qMRLab](https://qmrlab.readthedocs.io/en/master/gui_usage.html#data-viewer)
## Mouse control
![](https://github.com/qMRLab/qMRLab/blob/master/docs/source/_static/imtool3D/imtool3D_mouse.gif)  
## Multi-label mask (ROI)
### edit mask
![](https://github.com/qMRLab/qMRLab/blob/master/docs/source/_static/imtool3D/imtool3D_roi.gif)  
### Smart brush
![](https://github.com/qMRLab/qMRLab/blob/master/docs/source/_static/imtool3D/imtool3D_smartbrush.gif)
### Polygon tool
![](https://github.com/qMRLab/qMRLab/blob/master/docs/source/_static/imtool3D/imtool3D_polygon.gif)

# Tutorial
## open a 5D volume
````matlab
A = rand(100,100,30,10,3);
imtool3D(A)
````

## open an MRI volume
````matlab
load mri % example mri image provided by MATLAB
D = squeeze(D);
D = permute(D(end:-1:1,:,:),[2 1 3]); % LPI orientation
tool = imtool3D(D);
tool.setAspectRatio([1 1 2.5]) % set voxel size to 1mm x 1mm x 2.5mm
````

## include in a GUI
````matlab
% Add viewer in a panel in the middle of the GUI
GUI = figure('Name','GUI with imtool3D embedded');
annotation(GUI,'textbox',[0 .5 1 .5],'String','Create your own GUI here',...
               'HorizontalAlignment','center','VerticalAlignment','middle');
Position = [0 0 1 .5]; % Bottom. normalized units
tool = imtool3D([],Position,GUI)

% set MRI image
load mri % example mri image provided by MATLAB
D = squeeze(D);
D = permute(D(end:-1:1,:,:),[2 1 3]); % LPI orientation
tool.setImage(D)
tool.setAspectRatio([1 1 2.5]) % set voxel size to 1mm x 1mm x 2.5mm
````

## show RGB image
#### Display a Grayscale Image
````matlab
corn_gray = imread('corn.tif',3); % load the gray version
imtool3D(corn_gray)
````
#### Display an RGB Image
````matlab
corn_RGB = imread('corn.tif',2); % load RGB version
tool=imtool3D(corn_RGB);
tool.isRGB = 1;
````
#### Display an Indexed Image
````matlab
[corn_indexed,map] = imread('corn.tif',1); % load indexed version
tool = imtool3D(corn_indexed);
h = tool.getHandles;
colormap(h.Axes(tool.getNvol),map)
colormap(h.HistImageAxes,map)
tool.setClimits([0 255])
````

## show montage
````matlab
tool = imtool3D;
tool.montage = 1;
````
<p align="center">
  Brain montage: 
</p>
<p align="center">
  <img src="https://user-images.githubusercontent.com/7785316/93062018-a4381700-f674-11ea-9d7c-684a406fa85a.png" width="400">
</p>

## play a video
````matlab
v = VideoReader('xylophone.mp4');
tool = imtool3D(v.read([1 Inf]));
tool.isRGB = 1;
````
use left/right arrows to move through image frames  
use shift+right for fast forward (10-by-10 frames)  
## show RGB volume
For this example, we will display color-coded nerve direction of the brain.  
Download HCP Diffusion MRI Template http://brain.labsolver.org/diffusion-mri-templates/hcp-842-hcp-1021
````matlab
%% load HCP Diffusion MRI Template
load('HCP842_1mm.fib','-mat')
% reshape FA
fa0 = reshape(fa0,dimension); fa0 = fa0(:,end:-1:1,:);
% Find diffusion peak direction
peak = reshape(odf_vertices(:,index0+1)',[dimension 3]); 
peak = peak(:,end:-1:1,:,:);

%% Open imtool3D and display
tool = imtool3D(repmat(fa0,[1 1 1 3]).*abs(peak));

% use RGB mode
tool.isRGB = 1;
tool.RGBdim = 4;
tool.RGBindex = [1 2 3];
% move to slice 63
tool.setCurrentSlice(63)
````
````matlab
% Optional: add vectors
u = peak(:,:,63,1);
v = peak(:,:,63,2);
fa0slice = fa0(:,:,63);
[X,Y] = meshgrid(1:size(peak,2),1:size(peak,1));
h = quiver(tool.getHandles.Axes(tool.getNvol()), X(:),Y(:),-v(:).*fa0slice(:)*2,u(:).*fa0slice(:)*2);
h.ButtonDownFcn = tool.getHandles.I.ButtonDownFcn;
````

<p align="center">
  Brain nerve bundles color-coded by direction: 
</p>
<p align="center">  
(red: left-right, green: antero-posterior, blue: superior-inferior)
</p>
<p align="center">
  <img src="CaptureRGBmode.PNG" width="400">
</p>
Use button bellow left slider ('R' on the screenshot) to turn between RGB and grayscale and to select active color channel 

## Overlay image
For this example we will display a map of brain activation extracted from an FMRI dataset.  
Download fmri dataset: http://www2.bcs.rochester.edu/sites/raizada/Matlab/fMRI/speech_brain_images.mat
````matlab
load('speech_brain_images.mat'); % load dataset
tool = imtool3D({subj_3danat speech_Tmap});
tool.setNvol(2); % show top image (brain activation for speech task)
tool.setClimits([1 7]); % threshold activation >1
tool.changeColormap('hot')
tool.setOpacity(.3)
````
<p align="center">
  Brain activation during a speech task:
</b>
<p align="center">
  <img src="https://user-images.githubusercontent.com/7785316/93019813-af8a3480-f5d9-11ea-9423-6bc3d7a489df.png" width="400">  
</p>

# what is new in this fork? 
* Support for 5D volumes (scroll through time and volumeS with arrows)
* Keyboard shortcut
* Multi-label mask
* Save mask
* NIFTI files (.nii) support (double click on a. nii file in Matlab filebrowser) 
* New tools for mask (interpolate slices, active contour...)
* Convert Mask2poly and poly2mask
* splines in polygons (double click a circle)
* 3 planes view

# Authors
Justin Solomon (Original release)  
Tanguy Duval (4D (time) and 5D (different contrast); multi-label mask; active_contour, undo button, mask2poly, poly2mask, shortcuts)  

# Original release
https://fr.mathworks.com/matlabcentral/fileexchange/40753-imtool3d
