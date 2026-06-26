
 - **main branch** --> [pointing-dummies]( https://github.com/MIA-Ground/UGVC-26/tree/high_level/pointing_dummies)

# Yaw/Pitch

## 1. Handle YawState 
 - Code file --> [handle-Yaw/pointing-dummies]( https://github.com/MIA-Ground/UGVC-26/blob/high_level/pointing_dummies/dummy_pointing/dummy_pointing/handle_YAW.py) 

- it's responsible for rotating the camera left & right until the detected object is **horizontally** centered.
- it only handles the **yaw** axis (turning around the **vertical** axis)

### Concluded that
- if the feedback error is **+ve** then --> then the object is on the left.
- if the feedback error is **-ve** then --> the object is on the right.
- if the feedback error is **zero** then --> the object is centered.🥳

- The **PID** automatically determines when the rotate either left, right & how fast the rotation is based on the sign & the size of the error.

#### Conceptually,

this state is doing one simple thing:

--> Keep rotating the camera until the center of the detected object matches the center of the camera's image. Once the difference is less than 20 **pixels** it stops & returns that the **yaw** axis is aligned.

## 2. Handle Yaw with Servo
- Code file --> [handle-Yaw-with-Servo/pointing-dummies](https://github.com/MIA-Ground/UGVC-26/blob/high_level/pointing_dummies/dummy_pointing/dummy_pointing/handle_YAW_servo.py)

**The purpose of this state is:**

- 1.Get the camera frame.
- 2.Get the detected object's bounding box.
- 3.Check whether the object is centered horizontally.
- 4.If not centered, move the servo left or right.
- 5.Return "aligned" when centered.
- 6.Return "lost" if the object disappears.

Servo starts centered, typically:
- 0° = full left
- 90°  = center
- 180° = full right

### Alignment Tolerance :
 
`self.tolerance = 20.0`  
meaning:
- If object center is within ±20 pixels of image center, it's considered aligned.

### Step size function : 
`self.step_size = 1.01`   
meaning: every correction by servo moves by **1 degree**.

### Calculating the object's center :

      
    (x1,y1) ------- (x2,y1)

       |               |
       |      Face     |
       |               |

    (x1,y2) ------- (x2,y2)

- Horizontal center = Center{x} = (x1 + x2 / 2)

for example; x1 = 100 & x2 = 300    
Its result :  Center{x} = 200

- Calculating the horizontal error:  
- The image's center, for a 640 pixel wide image, therefore Xcenter = 320  
- we get the error by : **center X - cx**  

Example :   
cx = 250  
center = 320  
Error:  
320 - 250 = +70   **---> +ve error**  
Therefore,  
- If error{x} > 0   
→ object is left

- If error{x} < 0  
→ object is right

### Debug information by drawing: 
- **Blue line** shows image's center 
- **Yellow line** shows distance from image's center to target 
- **Green rectangle** shows detected face/object 
- **Red dot** shows object's center

### Alignment check :
 Since en el `tolerance = 20`, Therefore, if errorX is less than **20px** it returns **true**. Other, if errorX is more than **20px** it returns **false**  
 - True --> means the object is centered enough

## 3. Handle Pitch
- Code file --> [handle-pitch/pointing-dummies](https://github.com/MIA-Ground/UGVC-26/blob/high_level/pointing_dummies/dummy_pointing/dummy_pointing/handle_bitch.py)

instead of controlling the yaw axis (left-right movement), it controls the pitch axis (up-down movement)  
- The goal is to keep the detected face/object **vertically centered**
by moving the pitch servo **up or down**

#### What the camera sees: 

               +----------------------+
               |                      |
               |       FACE           |
               |                      |
               |----------------------| ← Image center
               |                      |
               |                      |
               +----------------------+

If the face is:

- Above the center → tilt camera up/down until centered
- Below the center → tilt camera up/down until centered
- the state finishes when the face reaches the center area

### Computing the vertical error :

- if the image's height is **480px**, then centerY = **240**  
--> this tells us how far the face is from the **vertical** center.

Example :   
if the face is above the center
- Center = 240
- Face   = 150  
Then:  
**240 - 150 = +90**  
--> **Positive error**  
meaning;   
- errorY > 0 → **face is above center**
- errorY < 0 **→ face is below center**

### Debug graphics by drawing:
- **Blue line** draws the horizontal center line 
- **Yellow line** shows the distance between the face center and the image center 
- **Green rectange** shows the binding box around the face 
- **Red dot** shows the exact center of the detected face.

### Alignment check: 
supposing that errorY = 15 & `tolerance = 20`  
Since, 15<20, therefore it returns true  
**--> meaning, the face is close enough to the vertical center.**

### Increasing the pitch angle: 

`self.current_pitch_angle -= self.step_size`   
--> This command moves the servo by **+1°**

`self.current_pitch_angle -= self.step_size`   
--> This command moves the servo by **-1°**

# Difference between Yaw & Pitch states 
### Movements 
- Yaw state controls left-right (horizontal) movements 
- Pitch state controls up-down (vertical) movements
 -----
### Error calculations
- Yaw --> error X
- Pitch --> error Y
------
### image dimensions 
- Yaw uses image's width
- Pitch uses image's height 
------
- Yaw publishes to `/servo_yaw `
- Pitch publishes to `/servo_pitch`
------
- Yaw centers the object horizontally 
- Pitch centers the object vertically
-----
So, both states perform 2-axis visual tracking.
