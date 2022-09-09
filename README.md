# ImageToText-ScaleDetection
<img alt="alt_text" src="https://github.com/BenjaminMartin86/ImageToText-ScaleDetection/blob/main/Pictures/indexTotal.png" />

**ImageToText-ScaleDetection** is an image processing repository that detect scale length and value on industrial images and convert them to text and numbers, that can be easily used for extract features measures on your image. The algorithm identifies ROI scale place in image, and after processing, gives an equivalence on pixels and millimeters (or any other metric). Images come from microscopic industrial weld beams. 

Image processing follows five steps:

1. **Crop on ROI**: Crop in the 20% bottom right of the image, assuming scale is always on the same location of the image
2. **Thresolding**: segmentation of the croped image between 200 and 255, to get a 2-colors images. All pixels lower than 200 turn to black, others turn to white
3. **Contour Detection**: Contour detection of the thresolded image. After that, get only the biggest contour, corresponding to the insert with scale information
4. **Crop on Scale**: Only keep the croped image defined by the biggest contour
5. **Detect lengh and value**: Detect the (horizontal) length of the scale. Converts the pixels to text, giving information on value (number) and metric (letters) provided by the scale.  

---

Installation
=============

Create your own virtual environnement
------------

Provides the conda environnement interface to ensure compatibility of methods and librairies.
Create numerical environnement :
```
conda create --yes --name ImageToText python=3.7
```

Active it : 
```
conda activate StatisticalMethodologies
```

Set new environnement on Jupyter Notebook
------------
Activate new environnement on Jupyter Notebook
```
conda install --yes -c anaconda ipykernel
```

We need to manually add the kernel if we want to have the virtual environment in the Jupyter Notebook
```
python -m ipykernel install --user --name=ImageToText
```

Install required modules
------------

To install compatible versions of modules for the notebook, you can install them from the command :
```
conda install --yes --file requirements.txt
```

---

Algorithm
=============
