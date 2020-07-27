[![KIEE](https://user-images.githubusercontent.com/57785792/88495612-69f7b500-cff5-11ea-8d13-9d8f5b3f7260.PNG)](http://www.kiee.or.kr/)

# 2020 Mini drone autonomous flight competition
### <img src="https://user-images.githubusercontent.com/57785792/87592435-d08ef000-c724-11ea-84ad-f33e3684752b.jpg" width="40" height="30">Kyunghee University  

***
![License](https://img.shields.io/badge/license-MIT-blue.svg) ![License](https://img.shields.io/badge/Raspberry%20pi%20zero-pass-blue) ![License](https://img.shields.io/badge/python-v3.7.3-brightgreen) ![License](https://img.shields.io/badge/open.cv-v3.2.0-brightgreen) 

## Context 
**1. Progress strategy**   
**2. Algorithm**   
**3. Source code**   
**4. Data sheet of Humming bird**   
**5. Author**   
**6. License**   

***
## 1. Progress strategy
### 1-a. Understanding of python
### 1-b. Understanding of Raspberry pi communicate 
   * Control a drone by pc through Raspberry pi
   ![communication](https://user-images.githubusercontent.com/57785792/87245242-c77cf500-c47e-11ea-8509-cd09d20e656e.PNG)

### 1-c. Practicing of using programs for contest
  
   ### Using program

   #### <img src="https://user-images.githubusercontent.com/57785792/87244515-e5475b80-c478-11ea-9177-7aef730dd40a.jpg" width="20" height="20"> PyCharm Community Edition 2020
   * Using for editting python 
   * [Guide](https://dora-guide.com/pycharm-install/)

   #### <img src="https://user-images.githubusercontent.com/57785792/87244655-9221d880-c479-11ea-9f17-bd71250f5528.jpg" width="20" height="20"> WinSCP
   * Using for data file trasfer
   * [Guide](http://blog.naver.com/PostView.nhn?blogId=websarang_&logNo=100052630947&viewDate=&currentPage=1&listtype=0)


   ####  <img src="https://user-images.githubusercontent.com/57785792/87244362-a95fc680-c477-11ea-9a8d-75ccf17f3cb1.png" width="20" height="20"> VNC viewer 
   * Using for connect PC and Raspberry pi
   * [Guide](https://itgroovy.tistory.com/549)

   #### <img src="https://user-images.githubusercontent.com/57785792/87244698-052b4f00-c47a-11ea-9a52-2520feb5dfed.png" width="20" height="20"> Sourcetree
   * Using for github update
   * [Guide](https://ux.stories.pe.kr/181)

### 1-d. Theorical understanding for flight of drone 
   * Changing of Throttle, Roll, Pitch and Yaw for position control
<img src="https://user-images.githubusercontent.com/57785792/87271991-e7102e00-c50f-11ea-8bd5-b28e52916819.png" width="443" height="333">

### 1-e. Position codes of drone and application codes
   ```python
   def sendControlPosition(self, positionX, positionY, positionZ, velocity, heading, rotationalVelocity):
   ```
   | Variable name | form | range | unit | explain |
   |:---------:|:----:|:----:|:---:|:---:|
   |position X|float|-10.0 ~ 10.0|meter|forward(+), behind(-)|
   |position Y|float|-10.0 ~ 10.0|meter|left(+), right(-)|
   |position Z|float|-10.0 ~ 10.0|meter|up(+), down(-)|
   |velocity|float|0.5 ~ 2.0|meter|moving velocity|
   |heading|Int16|-360 ~ 360|degree|left turn(+), right turn(-)|
   |rotationalVelocity|Int16|10 ~ 360|degree/s|rotational velocity|
   * Reference code : http://dev.byrobot.co.kr/documents/kr/products/e_drone/library/python/e_drone/

### 1-f. Map 
 <img src="https://user-images.githubusercontent.com/57785792/87243596-343dc280-c472-11ea-9ef1-3f49f39e3d71.jpg" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87506097-7569e880-c6a5-11ea-9207-a845a97a5c6d.jpg" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87506091-7438bb80-c6a5-11ea-91df-7ae24ff78986.jpg" width="320" height="240"> 

   * Fix below the ceiling but we can change height
   * About the third ring, it can change right and left
   * Have some patterns on the floor for stable hovering 
 
### 1-g. Error handling
#### Hardware problem
    Use sensor reset in controller for H.W problem
    Setting trim for stable hovering
    Check the whole manual 
  Go to "[manual](http://www.roboworks.co.kr/web/home.php?mid=10&go=pds.list&pds_type=1&start=0&num=23&s_key1=&s_que=) "
   
#### Software problem
    Used the try catch grammer for landing that you have any errors in flight

## 2. Algorithm
    In mode3, if the circle color is red go back to mode 1
    After in mode3, if the circle color is blue just Landing
   ![real_last_algorithm](https://user-images.githubusercontent.com/57785792/87593697-d259b300-c726-11ea-9e77-56ba574bf0dd.PNG)


## 3. Source code
  ### Setting the mask of ring
  * Pictures from contest map
  ```python
lower_green = (60, 150, 55)
upper_green = (80, 255, 255)
lower_red1 = (150, 30, 30)  
upper_red1 = (190, 255, 255)  
lower_red2 = (-10, 100, 55)  
upper_red2 = (10, 255, 255)  
lower_blue = (90, 190, 75)  
upper_blue = (110, 255, 255) 

drone = Drone()
drone.open()
  ```
   ### Function 
   * The function of find the ring
   * Return value is center of the ring
```python 
def detect_rect(img_hsv):
    try:
        img_mask_green = cv2.inRange(img_hsv, lower_green, upper_green)
        dst3 = cv2.medianBlur(img_mask_green, 15)

        _, contours, hierarchy = cv2.findContours(dst3, cv2.RETR_LIST, cv2.CHAIN_APPROX_SIMPLE)

        area_temp = 640 * 480
        area_temp2 = 0

        for cnt in contours:
            epsilon = 0.01 * cv2.arcLength(cnt, True)  
            approx = cv2.approxPolyDP(cnt, epsilon, True)
            area = cv2.contourArea(approx)
            mmt = cv2.moments(approx)
            cx = int(mmt['m10'] / mmt['m00'])
            cy = int(mmt['m01'] / mmt['m00'])

            if (8000 < area) and (area < area_temp):
                area_temp = area
                pt_temp = (cx, cy)

            if (8000 < area) and (area > area_temp2):
                area_temp2 = area
                approx_temp = approx

        if len(approx_temp) == 4:
            approx_list = [tuple(approx_temp[0, 0]), tuple(approx_temp[1, 0]), tuple(approx_temp[2, 0]),
                           tuple(approx_temp[3, 0])]
            approx_list.sort()
            left_length = abs(approx_list[0][1] - approx_list[1][1])
            right_length = abs(approx_list[2][1] - approx_list[3][1])
        else:
            left_length = 0
            right_length = 0

        return pt_temp, left_length, right_length

    except Exception as e:
        return 'fail'
```

 <img src="https://user-images.githubusercontent.com/57785792/87419185-05f4ea00-c60e-11ea-8a12-afb6386b9094.jpg" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87418485-b8c44880-c60c-11ea-9531-df2a25db001e.jpg" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87418488-b9f57580-c60c-11ea-8c97-bb9fb3afe7ec.jpg" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87418488-b9f57580-c60c-11ea-8c97-bb9fb3afe7ec.jpg" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87418494-ba8e0c00-c60c-11ea-9446-05676e2bf838.jpg" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87418496-bbbf3900-c60c-11ea-9cc4-01175ab64b2e.jpg" width="320" height="240"> 
   | **original** | **HSV** |
   |:--------:|:--------:|
   |**mask_green**|**morphology**|
   |**median**|**result**|
***
   * The function of find the color of circle
   * Return value is color of circle
   ```python
   def detect_color(img_hsv):
    img_mask_red1 = cv2.inRange(img_hsv, lower_red1, upper_red1)
    img_mask_red2 = cv2.inRange(img_hsv, lower_red2, upper_red2)
    img_mask_red = cv2.add(img_mask_red1, img_mask_red2)
    img_mask_blue = cv2.inRange(img_hsv, lower_blue, upper_blue)

    R = np.sum(img_mask_red == 255, axis=None)
    B = np.sum(img_mask_blue == 255, axis=None)

    if R > B:  # red
        circle_color = 'red'
    else:  # blue
        circle_color = 'blue'
    return circle_color
   ```
   ***
   * The function of find the circle
   * Return value is center of circle
   ```python
   def detect_circle(img_hsv, circle_color):
    if circle_color == 'red':
        img_mask_red1 = cv2.inRange(img_hsv, lower_red1, upper_red1)
        img_mask_red2 = cv2.inRange(img_hsv, lower_red2, upper_red2)
        img_mask_red = cv2.add(img_mask_red1, img_mask_red2)
        dilation_red = cv2.morphologyEx(img_mask_red, cv2.MORPH_CLOSE, np.ones((15, 15), np.uint8))
        dilation_red = cv2.dilate(dilation_red, np.ones((7, 7), np.uint8), iterations=1)
        dst_red = cv2.medianBlur(dilation_red, 7)
        circles = cv2.HoughCircles(dst_red, cv2.HOUGH_GRADIENT, 1, 50, param1=50, param2=10, minRadius=3, maxRadius=70)
    elif circle_color == 'blue':
        img_mask_blue = cv2.inRange(img_hsv, lower_blue, upper_blue)
        dilation_blue = cv2.morphologyEx(img_mask_blue, cv2.MORPH_CLOSE, np.ones((15, 15), np.uint8))
        dilation_blue = cv2.dilate(dilation_blue, np.ones((7, 7), np.uint8), iterations=1)
        dst_blue = cv2.medianBlur(dilation_blue, 7)
        circles = cv2.HoughCircles(dst_blue, cv2.HOUGH_GRADIENT, 1, 50, param1=50, param2=10, minRadius=3, maxRadius=70)

    if circles is not None:
        max_circle = max(circles[0, :, 2])
        for j in circles[0]:
            if int(max_circle) == int(j[2]):
                return j[0], j[1]
    else:
        return 0, 0
   ```
  
<img src="https://user-images.githubusercontent.com/57785792/87421845-90d7e380-c612-11ea-82bb-f9ac96bb3489.png" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87421847-92091080-c612-11ea-83de-7477b3c74693.png" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87421849-92091080-c612-11ea-9955-6d8fe7c361ef.png" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87421851-92a1a700-c612-11ea-973b-3cd6551b83aa.png" width="320" height="240"> <img src="https://user-images.githubusercontent.com/57785792/87421852-92a1a700-c612-11ea-88c5-3e11347e3141.png" width="320" height="240"> 
  | **img_mask_blue** | **morphology** |
  |:--------:|:-------:|
  | **dilate** | **medianBlur** |
  |  **result** |  |
  ***

  ### Inside of try
  * After take off, go forward for 0.7m & begin mode 2
  ```python 
try:
    drone.sendTakeOff()
    sleep(5)
    camera = PiCamera()
    camera.resolution = (640, 480)  # (2592,1944)
    camera.framerate = 32
    rawCapture = PiRGBArray(camera, size=(640, 480))

    circle_color = 'red'
    mode1 = False
    mode2 = True
    mode3 = False

    ring_range = 40
    
    i = 0
    no_ring_cnt = 0
    drone.sendControlPosition16(7, 0, 0, 5, 0, 0)
    sleep(3)
    
    for frame in camera.capture_continuous(rawCapture, format="bgr", use_video_port=True):
        img = frame.array
        img = cv2.flip(img, 0)  
        img = cv2.flip(img, 1)  
        img_hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
        cv2.imwrite("total_capture/{}.jpg".format(i), img)
        i = i + 1

        rawCapture.truncate(0)  
  ```
  * mode 1 : go forward for 1.3m & change to mode 2
   ```python 
       if mode1:  # 1.3m
            drone.sendControlPosition16(13, 0, 0, 5, 0, 0)
            sleep(5)
            mode1 = False
            mode2 = True
            
  ```
  
   * mode 2 : Control left right and degree adjustment (watch the camera straight forward) & change to mode 3 
   ```python 
        
        elif mode2:
            if 'fail' == detect_rect(img_hsv):
                continue
            else:
                pt_temp, left_length, right_length = detect_rect(img_hsv)
                if pt_temp[1] > 480 - 90+20:
                    drone.sendControlPosition16(0, 0, -7, 5, 0, 0)
                    sleep(3)
                elif pt_temp[1] < 90+20:
                    drone.sendControlPosition16(0, 0, 7, 5, 0, 0)
                    sleep(3)
                else:
                    if pt_temp[1] > 480 - 130+20:
                        drone.sendControlPosition16(0, 0, -3, 5, 0, 0)
                        sleep(1)
                    elif pt_temp[1] < 130+20:
                        drone.sendControlPosition16(0, 0, 3, 5, 0, 0)
                        sleep(1)
                    elif pt_temp[0] < 100:
                        drone.sendControlPosition16(0, 11, 0, 5, 0, 0)
                        sleep(5)
                    elif pt_temp[0] > 640 - 100:
                        drone.sendControlPosition16(0, -11, 0, 5, 0, 0)
                        sleep(5)
                    else:
                        if pt_temp[1] > 240 + ring_range + 30:
                            drone.sendControlPosition16(0, 0, -1, 5, 0, 0)
                            sleep(0.5)
                        elif pt_temp[1] < 240 - ring_range + 30:
                            drone.sendControlPosition16(0, 0, 1, 5, 0, 0)
                            sleep(0.5)
                        elif pt_temp[0] > 320 + ring_range:
                            drone.sendControlPosition16(0, -1, 0, 5, 0, 0)
                            sleep(0.5)
                        elif pt_temp[0] < 320 - ring_range:
                            drone.sendControlPosition16(0, 1, 0, 5, 0, 0)
                            sleep(0.5)
                        else: #각도 미세조정
                            if left_length - right_length > 25:
                                drone.sendControlPosition16(0, 0, 0, 0, 10, 20)
                                sleep(3)
                            elif right_length - left_length > 25:
                                drone.sendControlPosition16(0, 0, 0, 0, -10, 20)
                                sleep(3)
                            else:
                                drone.sendControlPosition16(8, 0, 0, 5, 0, 0)
                                sleep(3)
                                mode2 = False
                                mode3 = True
  ``` 
  * mode 3 : go forward after find a color of the circle and find cirle   (Blue color is Landing, Red color is rotate for 90 degrees to counter clockwise) 
   ```python 
        elif mode3:
            circle_color = detect_color(img_hsv)
            circle_x, circle_y = detect_circle(img_hsv, circle_color)
            if circle_x == 0:
                continue
            else:   
                    drone.sendControlPosition16(0, -1, 0, 5, 0, 0)
                    sleep(0.5)
                elif circle_x < 320 - 40:
                    drone.sendControlPosition16(0, 1, 0, 5, 0, 0)
                    sleep(0.5)
                else:
                    sleep(1)
                    drone.sendControlPosition16(10, 0, 0, 5, 0, 0)
                    sleep(6)
                    if circle_color == 'blue':
                        sleep(1)
                        drone.sendLanding()
                        drone.close()
                        break
                    elif circle_color == 'red':
                        drone.sendControlPosition16(0, 0, 0, 0, 90, 20)
                        sleep(6)
                        mode3 = False
                        mode1 = True
```
   
   * If there is errors just landing (prevention of collision)
   ```python 
except Exception as e:
    drone.sendLanding()
    drone.close()
   ```
   
  
  
## 4. Data sheet of humming bird 
![Hummingbird](https://user-images.githubusercontent.com/57785792/87243462-20459100-c471-11ea-912a-3ae214db21a1.PNG)
* Go to [Details]( http://www.roboworks.co.kr/web/home.php?go=page_view&gubun=1&mid=10)
## 5. Author
### Team leader: [Seungjik Kim](https://github.com/ksj961323)    
### Team members: [Yeongin Kim](https://github.com/yeongin1230), [Soyoung Jung](https://github.com/jsy5236)
## 6. License

This project is licensed under the MIT License - see the [LICENSE](https://github.com/yeongin1230/Bleague_drronedrrone/blob/master/LICENSE) file for details
