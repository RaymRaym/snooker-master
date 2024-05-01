# Snooker Master

## Identify the balls

As a popular ball game, snooker puts forward certain requirements for ball color recognition in competition and training scenarios. Correctly identifying the color of billiard balls is not only crucial to the fairness of game referees, but also enhances viewing pleasure and improves training efficiency.



Traditionally, this work has relied on manual execution, but this approach is time-consuming and can be affected by human error. Therefore, it is particularly important to develop an automated ball color recognition system that can accurately identify the ball color on the table in real time, support game referees and training feedback, and improve efficiency and accuracy.



### Define the color range of various balls

Identifying the color of balls can help determine the type of ball and its corresponding color in an automated system. We used the HSV color space to define the color range of various billiard balls. The HSV color model is more intuitive and effective than the RGB model in processing real-world colors, especially in color segmentation and object tracking. In the code, we define a dictionary named `color_ranges`, which contains the HSV ranges of balls of different colors. Each color label corresponds to a pair of HSV values that define the minimum and maximum range of the color. The specific colors and their corresponding HSV ranges are as follows:

```python
color_ranges = {
    'blue': ((25, 70, 50), (35, 255, 255)),
    'yellow': ((85, 210, 240), (95, 220, 250)),
    'green': ((50, 210, 150), (60, 220, 160)),
    'brown': ((80, 190, 100), (90, 200, 110)),
    'pink': ((110, 130, 220), (120, 140, 230)),
    'black': ((50, 200, 30), (60, 210, 40)),
    'white': ((80, 50, 250), (100, 70, 255))
}
```

`color_ranges` can be used in image processing to identify and classify billiard balls of different colors by comparing whether the HSV value of each pixel in the image falls within the range defined above. By precisely defining the color range of billiard balls, we can effectively automatically identify the color of billiard balls in digital images, thereby improving game interactivity and viewing.



### Identify the color of balls on a 2D table

Previously we defined the HSV range for various billiard ball colors, next we need to preprocess the images in order to identify balls of specific colors in these images.

```python
hsv_image = cv2.cvtColor(image_2d, cv2.COLOR_BGR2HSV)
```

We use the `cv2.cvtColor` function of the OpenCV library to convert the image from BGR color space to HSV color space. In computer vision, BGR is the default color space format and the HSV color space is better suited for color recognition:

- Hue: the type of color
- Saturation: the intensity or purity of a color
- Value: the brightness of the color

By converting to the HSV color space, we can more easily identify colors with simple range checks.



### Contour detection and calculation of the center of the ball

For each defined color range, we need to filter the image using a color mask, and then use contour detection to find balls of the corresponding color in the image. Only when the detected contour area is large enough, it is considered to be a valid sphere contour. We calculate the position of its center of mass from the effective contour, and this information is used to determine the center of the ball for subsequent marking.

```python
# Store recognition results
balls_detected = []

for color_name, (lower, upper) in color_ranges.items():
    # Create a color mask
    mask = cv2.inRange(hsv_image, np.array(lower), np.array(upper))

    # Find contours in mask
    contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

    # Iterate through all contours and calculate their centroid
    for contour in contours:
        area = cv2.contourArea(contour)
        if area > 50:
            M = cv2.moments(contour)
            if M['m00'] != 0:
                # Calculate center of ball
                cx = int(M['m10'] / M['m00'])
                cy = int(M['m01'] / M['m00'])
                balls_detected.append((cx, cy, color_name))
                
balls_detected
```

After repeating this process across all color ranges, `balls_detected` list will contain the center coordinates and corresponding colors of all detected balls. This information can be used for further image analysis or to provide visual aids in game tracking systems.



Finally, we draw a circle at the location of each detected billiard ball and label the ball's color name.

```python
# Draw a circle to mark the position of the ball and write the color name of the ball
for (x, y, color_name) in balls_detected:
    cv2.circle(marked_image, (x, y), 10, (0, 255, 0), 2)
    cv2.putText(marked_image, color_name, (x, y - 12), cv2.FONT_HERSHEY_SIMPLEX, 0.4, (0, 255, 0), 1)
```

We can get the billiards picture on the two-dimensional table and the corresponding identification content, including location information and color, etc.

![marked image](marked image.png)





## Generate the path

In the final step, we are going to take aim. On the snooker table, to finish a hit, we have three main components. The cue ball, the target ball and the pot. Basically, we need to determine a ball as our target and choose a pot that is suitable for the target to get in. 



First, let's review the basic rules of snooker. The only ball we can hit is the cue ball which is known as the white ball. We hit the cue ball, then it starts to move. During its moving, it hits other balls and our goal is to make them roll into a pot one by one. 



The basic situation is shown in the folloing illustration. 

![](https://my-images-bucket-wrhlh.s3.us-east-2.amazonaws.com/notebook/cv-final-basic-snooker.png)

Three main components are in the same line. Intuitively, we will choose to hit the cue ball in the same direction as the line. 



But, **what if they are randomly located on the table?**  I have to say this is the most common situation on the snooker table.  See below.

![](https://my-images-bucket-wrhlh.s3.us-east-2.amazonaws.com/notebook/cv-final-basic-snooker-ad.png)

What strategy should we take now?



Our solution is to find the **hitting point** first. The so called hitting point is the position where the cur ball should hit our target ball. How to find it? Basically, it should be the **tanget circle** of our taget ball and the center of it should also be on the same line given by the target ball and the pot.  

![](https://my-images-bucket-wrhlh.s3.us-east-2.amazonaws.com/notebook/cv-final-basic-snooker-hit-logic.png)

So how to calculate the center of the tanget circle.

See the illustatraion below.



![](https://my-images-bucket-wrhlh.s3.us-east-2.amazonaws.com/notebook/cv-messy-math2.png)

Given the Target $(x_1, y_1)$, Pot $(x_0, y_0)$, and the line function between them $y = kx + b$ , $k = \frac{x_1-x_0}{y_1-y_0}$

the equation for the Hitting point $(x_2, y_2)$ is:

$x_2 = x_1 + \frac{d}{\sqrt{1+k^2}}$

$y_2 = y_1 + \frac{dk}{\sqrt{1+k^2}}$



Then we connect the hitting point and the cue ball. This direction will be the one that we will take.



**Wait! What if there is another ball blocking the way?**

![](https://my-images-bucket-wrhlh.s3.us-east-2.amazonaws.com/notebook/cv-final-basic-snooker-hit-logic-blocked.png)

### Show time! Play a bank shot!



But, how to find the bank point ?

![](https://my-images-bucket-wrhlh.s3.us-east-2.amazonaws.com/notebook/cv-messy-math2-3.png)The principle behind this: **Incidence angle equals reflection angle**.

Thanks to the fundamental law of reflection, we have two similar tiangles here. Given this fact, it is not difficult to find the bank point. Hitting Point $(x_1, y_1)$, Cue Ball$(x_2, y_2)$

$\frac{x_1}{x_2} = \frac{a}{b}$

$a+b = y_2-y_1$

so $a = \frac{x_1(y_2-y_1)}{x_1+x_2} $

Bank Point's position (in this situation)will be $(0, y_1 + \frac{x_1(y_2-y_1)}{x_1+x_2})$

The equation will be slightly different when the border is located at the right, up or botton.



### Play it !

We designed an interactive user interface in our jyupter notebook.  Choose the target ball and the pot. We will generate the path for you. 

![](https://my-images-bucket-wrhlh.s3.us-east-2.amazonaws.com/notebook/cv-final-basic-snooker-ui-1.png)

Show your "brilliant skills" by using our Snooker Master !!

![](https://my-images-bucket-wrhlh.s3.us-east-2.amazonaws.com/notebook/cv-final-basic-snooker-ui-2.png)

![](https://my-images-bucket-wrhlh.s3.us-east-2.amazonaws.com/notebook/cv-final-basic-snooker-ui-3.png)





