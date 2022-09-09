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
conda activate ImageToText
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
pip install requirements.txt
```

---

Algorithm
=============

1.Crop on ROI
------------

Giving your *Path* to image, Crops in the 20% bottom right of the image:
```ruby
import numpy as np
import cv2
import matplotlib.pyplot as plt

image = cv2.imread(Path)
#Location of the bottom right part
Echelle = 0.8
CropEchelle = image[np.round(image.shape[0]*Echelle,0).astype(int):,
              np.round(image.shape[1]*Echelle,0).astype(int):,:]
```

As a result, the following processing is made on image:

<img src="https://github.com/BenjaminMartin86/ImageToText-ScaleDetection/blob/main/Pictures/Step1.png" width="600">

2.Thresolding
------------

Once image cropped, a segmentation is performed between 200 and 255, to get a 2-colors images. All pixels lower than 200 turn to black, others turn to white.
```ruby
CropEchelleGray = cv2.cvtColor(CropEchelle, cv2.COLOR_BGR2GRAY)
ret, thresh = cv2.threshold(CropEchelleGray, 250, 255, cv2.THRESH_BINARY)
```

As a result, the following processing is made on image:

<img src="https://github.com/BenjaminMartin86/ImageToText-ScaleDetection/blob/main/Pictures/Step2.png" width="600">

3.Contour Detection
------------

Using `findContours` method of Opencv, a contour detection of the thresolded image. After that, get only the biggest contour, corresponding to the insert with scale information.
```ruby
# find the largest contour in the threshold image
cnts = cv2.findContours(thresh.copy(), cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_SIMPLE)
cnts = imutils.grab_contours(cnts)
c = max(cnts, key=cv2.contourArea)

# draw the shape of the contour on the output image, compute the
# bounding box, and display the number of points in the contour
output = CropEchelleGray.copy()
cv2.drawContours(output, [c], -1, (0, 255, 0), 5)
```

As a result, the following processing is made on image:

<img src="https://github.com/BenjaminMartin86/ImageToText-ScaleDetection/blob/main/Pictures/Step3.png" width="600">

4.Crop on Scale
------------

Only keep the croped image defined by the biggest contour.
```ruby
#Crop on contour
mask = np.zeros_like(output) # Create mask where white is what we want, black otherwise
cv2.drawContours(mask, [c], -1, 255, -1) # Draw filled contour in mask
out = np.zeros_like(output) # Extract out the object and place into output image
out[mask == 255] = output[mask == 255]

# Now crop
(y, x) = np.where(mask == 255)
(topy, topx) = (np.min(y), np.min(x))
(bottomy, bottomx) = (np.max(y), np.max(x))
out = out[topy:bottomy+1, topx:bottomx+1]
```

As a result, the following processing is made on image:

<img src="https://github.com/BenjaminMartin86/ImageToText-ScaleDetection/blob/main/Pictures/Step4.png" width="600">

4.Detect lengh and value
------------

Detect the (horizontal) length of the scale. Converts the pixels to text, giving information on value (number) and metric (letters) provided by the scale.  
```ruby
import easyocr

#Segment Length
black_pixels = np.array(np.where(out.T ==0))
first_black_pixel = black_pixels[:,0]
last_black_pixel = black_pixels[:,-1]
LengthSegment = last_black_pixel[0]-first_black_pixel[0]
print('Segment Length : ',LengthSegment)

#Scale value detection
reader = easyocr.Reader(['en'])
result = reader.readtext(out)
ScaleNumber = int(re.findall('\d+', result[0][-2])[0])
Metric = result[0][-2].split('{} '.format(ScaleNumber))[-1]
print('Scale : ', Echelle)

TextLargeurSegment = 'Segment Length : {} px '.format(LengthSegment)
TextValeurEchelle = 'Segment Value : {} {}'.format(ScaleNumber,Metric)
cv2.putText(image, TextLargeurSegment, (600, 300), cv2.FONT_HERSHEY_COMPLEX_SMALL,3, (0,255,0), 6)
cv2.putText(image, TextValeurEchelle, (600, 400), cv2.FONT_HERSHEY_COMPLEX_SMALL,3, (0,255,0), 6)
```

As a result, the following processing is made on image:

<img src="https://github.com/BenjaminMartin86/ImageToText-ScaleDetection/blob/main/Pictures/Step5.png" width="600">
