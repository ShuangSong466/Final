
https://user-images.githubusercontent.com/73319539/116725925-84bee600-a9da-11eb-9db6-c858afe7d1f8.mp4

# Final
GitHub ：https://github.com/ShuangSong466/Final/blob/main/test.py

I used open-cv to realize simple gesture recognition. The knowledge I mainly use is opencv, basic python syntax, and basic knowledge of image processing.

First get the video (camera)
cap = cv2.VideoCapture("C:/Users/lenovo/Videos/1.mp4")#Read file
#cap = cv2.VideoCapture(0)#Read the camera
while(True):
    ret, frame = cap.read() key = cv2.waitKey(50) & 0xFF
    if key == ord('q'):
    break
cap.release()
cv2.destroyAllWindows()


Then: skin tone detection
The ellipse skin color detection model is used here
In the RGB space, the skin color of a human face is greatly affected by the brightness, so it is difficult to separate the skin color points from the non-skin color points. That is to say, after processing in this space, the skin color points are discrete points with many non-skin colors embedded in the middle. , Which brings difficulties to the calibration of skin color areas (face calibration, eyes, etc.). If you convert RGB to YCrCb space, you can ignore the influence of Y (brightness), because the space is affected little by brightness, and skin tones will produce good convergence. In this way, the three-dimensional space will be two-dimensional CrCb, and the skin color points will form a certain shape. For example, if you have a face, you will see an area of a face, and if you have an arm, you will see the shape of an arm. def A(img):

    YCrCb = cv2.cvtColor(img, cv2.COLOR_BGR2YCR_CB) #convert to YCrCb space
    (y,cr,cb) = cv2.split(YCrCb) #Split out Y, Cr, Cb values
    cr1 = cv2.GaussianBlur(cr, (5,5), 0)
    _, skin = cv2.threshold(cr1, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU) #Ostu processing
    res = cv2.bitwise_and(img,img, mask = skin)
    return res



Contour processing
Two functions are mainly used for contour processing, cv2.findContours and cv2.drawContours. These two functions are easy to use. The main problem in this part is that there are many contours extracted, but I only need the contour of the hand. , So I have to use the sorted function to find the largest contour.

def B(img):

    #binaryimg = cv2.Canny(Laplacian, 50, 200) #Binaryization, canny detection
    h = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE) #Find contours
    contour = h[0]
    contour = sorted(contour, key = cv2.contourArea, reverse=True)#The area of the contoured area is sorted
    #contourmax = contour[0][:, 0, :]#The coordinates of the contour point with the largest reserved area
    bg = np.ones(dst.shape, np.uint8) *255#Create a white curtain
    ret = cv2.drawContours(bg,contour[0],-1,(0,0,0),3) #Draw a black contour
    return ret


So the entire code is:
""" Read frame from video and save as picture"""
import cv2
import numpy as np
cap = cv2.VideoCapture("C:/Users/lenovo/Videos/1.mp4")#Read file
#cap = cv2.VideoCapture(0)#Read the camera

#Skin detection
def A(img):

    YCrCb = cv2.cvtColor(img, cv2.COLOR_BGR2YCR_CB) #convert to YCrCb space
    (y,cr,cb) = cv2.split(YCrCb) #Split out Y, Cr, Cb values
    cr1 = cv2.GaussianBlur(cr, (5,5), 0)
    _, skin = cv2.threshold(cr1, 0, 255, cv2.THRESH_BINARY + cv2.THRESH_OTSU) #Ostu processing
    res = cv2.bitwise_and(img,img, mask = skin)
    return res

def B(img):

    #binaryimg = cv2.Canny(Laplacian, 50, 200) #Binaryization, canny detection
    h = cv2.findContours(img,cv2.RETR_EXTERNAL,cv2.CHAIN_APPROX_NONE) #Find contours
    contour = h[0]
    contour = sorted(contour, key = cv2.contourArea, reverse=True)#The area of the contoured area is sorted
    #contourmax = contour[0][:, 0, :]#The coordinates of the contour point with the largest reserved area
    bg = np.ones(dst.shape, np.uint8) *255#Create a white curtain
    ret = cv2.drawContours(bg,contour[0],-1,(0,0,0),3) #Draw a black contour
    return ret


while(True):

    ret, frame = cap.read()
    #The following three lines can be adjusted according to your own computer
    src = cv2.resize(frame,(400,350), interpolation=cv2.INTER_CUBIC)#Window size
    cv2.rectangle(src, (90, 60), (300, 300 ), (0, 255, 0))#Frame the interception position
    roi = src[60:300, 90:300] # Get gesture block diagram

    res = A(roi) # Perform skin tone detection
    cv2.imshow("0",roi)

    gray = cv2.cvtColor(res, cv2.COLOR_BGR2GRAY)
    dst = cv2.Laplacian(gray, cv2.CV_16S, ksize = 3)
    Laplacian = cv2.convertScaleAbs(dst)

    contour = B(Laplacian)#Contour processing
    cv2.imshow("2",contour)

    key = cv2.waitKey(50) & 0xFF
    if key == ord('q'):
            break
cap.release()
cv2.destroyAllWindows()
![image](https://user-images.githubusercontent.com/73319539/116725971-90121180-a9da-11eb-9451-0ffee914ab56.png)
