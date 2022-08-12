# 4DOF Smart Robot Mechanical Arm
The 4DOF Mechanical Arm is a controllable arm that consists of 4 servos for movement. The user is able to utilize a wired controller or connect an android phone via bluetooth.

| **Engineer** | **School** | **Area of Interest** | **Grade** |
|:--:|:--:|:--:|:--:|
| Jeffrey Y. | Aliso Niguel High School | Mechanical Engineering | Incoming Senior

![Headstone Image](https://github.com/BlueStampEng/BSE_Template_Portfolio/blob/4655d8c4b2f1d0fa5912511d0b39542520b9f88e/branding/BlueStamp-Engineering-Logo-White.png)
  
# Final Milestone
My final milestone was me implementing motion controls for the mechanical arm. For this, I utilized the accelerometer and tested its values to make sure it was working. After this, I combined and wrote my own code in order to send inputs from the accelerometer to the servo motors. There were no limits for the accelerometer, so I had to create a deadzone to hold the arm still. Originally I wanted to make the accelerometer wireless, but the bluetooth HC-06 module was not cooperating with the HC-05 module due to issues with the sensor board. In the limited time I had, I just made a wired connection, but theoretically, it could still work. For the controls, all directions worked except the yaw axis. In order to fixed this, I made a separate button on a breadboard circuit to control the claw.

<iframe width="620" height="348.75" src="https://www.youtube.com/embed/W1bAl9p6qxw" title="Jeffrey Y Milestone 3" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

# Second Milestone
My second milestone was working on how to control my mechanical arm. First I implemented a controller and used arduino code to take  inputs from the controller and output servo movement based on each joystick direction (8 total). After that, I began working on a wireless controller. I was able to get my hands on an android phone and attempted to use a premade app, but it required an older version of android. This led me into making my own app in the MIT App Inventor, where I designed and coded the android app to connect via bluetooth. The bluetooth connection was made possible by the HC-06 module, which was a little finicky to use. This process took many attempts and tests in order to calibrate the controls correctly.

<iframe width="620" height="348.75" src="https://www.youtube.com/embed/q5HAsUY1p20" title="Jeffrey Y Milestone 2" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
# First Milestone
  

My first milestone was constructing the physical mechanical arm and mounting the arduino with the extension board on the base. I first downloaded arduino and plugged in the individual servos in order to ensure that everything was functional. After that, most of the time was dedicated towards making the physical arm. Halfway through I realized that I was using the wrong screws as the 10mm and 12mm were extremely similar. To avoid any issues, I had to unscrew the wrong screws and replace them with the correct ones. When finishing the arm, I also had to loosen some of the screws in order for the servos to move freely. Finally, I plugged a single servo to the board and tested its functionality.

<iframe width="620" height="348.75" src="https://www.youtube.com/embed/L_sFzxW_xlw" title="Jeffrey Y Milestone 1" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

```Arduino
#include <Servo.h> // add the servo libraries
#include <Wire.h>
#include <MPU6050.h>

Servo myservo1;  // create servo object to control a servo
Servo myservo2;
Servo myservo3;
Servo myservo4;
int pos1=90, pos2=90, pos3=90, pos4=90;  // define the variable of 4 servo angle and assign the initial value( that is the boot posture angle value)

MPU6050 mpu;

void setup()
{
  Serial.begin(9600);
  
   // boot posture
  myservo1.write(pos1);  
  delay(1000);
  myservo2.write(pos2);
  myservo3.write(pos3);
  myservo4.write(pos4);
  delay(1500);
  Serial.begin(9600); //  set the baud rate to 9600

  Serial.println("Initialize MPU6050");

  while(!mpu.begin(MPU6050_SCALE_2000DPS, MPU6050_RANGE_2G))
  {
    Serial.println("Could not find a valid MPU6050 sensor, check wiring!");
    delay(500);
  }
  Serial.begin(9600); //added

  pinMode(1, INPUT_PULLUP); //added

  pinMode(0, OUTPUT); //added
}

void loop()
{
  int sensorVal = digitalRead(2); //added
  Serial.println(sensorVal); //added
  sensorVal = digitalRead(0);
  
  myservo1.attach(3);  // set the control pin of servo 1 to D3
  myservo2.attach(5);  // set the control pin of servo 2 to D5
  myservo3.attach(6);   // set the control pin of servo 3 to D6
  myservo4.attach(9);   // set the control pin of servo 4 to D9

  // Read normalized values 
  Vector normAccel = mpu.readNormalizeAccel();

  // Calculate Pitch & Roll
  int pitch = -(atan2(normAccel.XAxis, sqrt(normAccel.YAxis*normAccel.YAxis + normAccel.ZAxis*normAccel.ZAxis))*180.0)/M_PI;
  int roll = (atan2(normAccel.YAxis, normAccel.ZAxis)*180.0)/M_PI;
  
  if ((pitch * (180/3.14)) <= -400) {
    if(pos2<25) // limit the retracted angle
    {
    pos2=25;
    }
    else
    {
    pos2=pos2-2;
    myservo2.write(pos2); // lower arm will draw back
    delay(5);
    }
    
  } else if ((pitch * (180/3.14)) >= 400) {
    if(pos2>180) // limit the stretched angle
    {
    pos2=180;
    }
    else
    {
    pos2=pos2+2;
    myservo2.write(pos2); // lower arm will stretch out
    delay(5);
    }
    
  } else if ((roll * (180/3.14)) <= -400) {
    if(pos1>180) //limit the angle when turn right 
    {
    pos1=180;
    }
    else
    {
    pos1=pos1+2; 
    myservo1.write(pos1); // arm turns left
    delay(5);
    }
    
    
  } else if ((roll * (180/3.14)) >= 400) {
    if(pos1<1) // limit the angle when turn left
    {
    pos1=1;
    }
    else
    {
    pos1=pos1-2; 
    myservo1.write(pos1); //servo 1 operates the motion, the arm turns right. 
    delay(5);
    }

    
  } else if (sensorVal == HIGH) {
    
    pos4=pos4+3; 
    myservo4.write(pos4); //servo 4 operates the motion, the claw gradually opens. 
    delay(5);
    if(pos4>120)
    {
    pos4=120;
    }
 
  } else if (sensorVal == LOW) {
    
    pos4=pos4-3; 
    myservo4.write(pos4); //servo 4 operates the motion, the claw gradually opens. 
    delay(5);
    if(pos4<45)
    {
    pos4=45;
    } 
  }
  
  delay(10);
}
```
