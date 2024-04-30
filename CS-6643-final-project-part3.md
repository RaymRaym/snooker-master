# Snooker Master

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





