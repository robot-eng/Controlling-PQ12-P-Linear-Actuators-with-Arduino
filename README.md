# Controlling-PQ12-P-Linear-Actuators-with-Arduino
### Circuit

<p align="center">
  <img src="/image/1.png" />
</p>
<p>M+, M- link L298N like a motor normal one motor / P+ connect +12v, P- connect -12V</p>

### Code Arduino + H-Bridge (L298N) and PQ12-P
```c++
#include <Servo.h>

#define in1 3
#define in2 4
#define enA 2


const int wiper1 = A0; //Position Potentionmeter Feedback
const int sense1 = A1; //Channel A Current Sense

int wip1 = 0;
int sen1 = 0;

float voltage1 = 0;
float current1 = 0;

int acSpeed = 0;

void setup() 
{
 // put your setup code here, to run once:
 pinMode(enA, OUTPUT); //Sets Enable A pin as OUTPUT for controlling Actuator Speed
 pinMode(in1, OUTPUT); //Sets INPUT 1 pin as OUTPUT for Motor Power and Potentiameter Refference
 pinMode(in2, OUTPUT); //Sets INPUT 2 pin as OUTPUT for Motor Power and Potentiameter Refference
 pinMode(wiper1, INPUT); //Sets Analog Pin 0 as an INPUT Pin

 Serial.begin(9600);
}

void gotoPos(uint16_t x)
{
   if (x <30) x=30;
   if (x> 990) x=990;
 
   uint8_t poof=0;
   if (analogRead(wiper1)<x)
   {
     digitalWrite(in1,HIGH);
     digitalWrite(in2,LOW);
     digitalWrite(enA,HIGH);
     do{ Serial.print("<");Serial.println(analogRead(wiper1)); delay(100);poof++; if (poof>=40)goto fakeout1;}while (analogRead(wiper1)<x);
     digitalWrite(enA,LOW);
   }
   else
   {
     digitalWrite(in2,HIGH);
     digitalWrite(in1,LOW);
     digitalWrite(enA,HIGH);
     do { Serial.print(">");Serial.println(analogRead(wiper1)); delay(100);poof++;if (poof>=40)goto fakeout2;}while (analogRead(wiper1)>x);
     digitalWrite(enA,LOW);
   }
   return;

   fakeout1:
   digitalWrite(in2,HIGH);
     digitalWrite(in1,LOW);
     delay(250);
     digitalWrite(enA,LOW);
     return;
   fakeout2:
   digitalWrite(in1,HIGH);
     digitalWrite(in2,LOW);
     delay(250);
     digitalWrite(enA,LOW);
     return;
}

#define MAXSPEED 255
#define MINSPEED 254

void gotoPos_P(uint16_t x)
{
   if (x <30) x=30;
   if (x> 990) x=990;
 
   uint8_t poof=0;
   uint16_t currPos = analogRead(wiper1);
   if (currPos<x)
   {
     digitalWrite(in1,HIGH);
     digitalWrite(in2,LOW);
     //digitalWrite(enA,HIGH);
     do{ 
       analogWrite(enA,map(abs(currPos-x),0,1023,MINSPEED,MAXSPEED));
       currPos= analogRead(wiper1);
       
       //Serial.print("<");Serial.println(currPos); delay(100);
       }      while (currPos<x);
     digitalWrite(enA,LOW);
   }
   else
   {
     digitalWrite(in2,HIGH);
     digitalWrite(in1,LOW);
     //digitalWrite(enA,HIGH);

      //digitalWrite(enA,HIGH);
     do{ 
       analogWrite(enA,map(abs(currPos-x),0,1023,MINSPEED,MAXSPEED));
       
       currPos= analogRead(wiper1);
       //Serial.print(">");Serial.println(currPos); delay(100);
     }      while (currPos>x);
     
     
     digitalWrite(enA,LOW);
   }

   Serial.print(">");Serial.println(currPos); delay(100);
   return;
   
   fakeout1:
   digitalWrite(in2,HIGH);
     digitalWrite(in1,LOW);
     delay(250);
     digitalWrite(enA,LOW);
     return;
   fakeout2:
   digitalWrite(in1,HIGH);
     digitalWrite(in2,LOW);
     delay(250);
     digitalWrite(enA,LOW);
     return;
   
}

void loop() 
{
 uint16_t val=512;
   
   while(1)
   {
     if (val!=0){
       gotoPos(val);
       Serial.print("sending:");
       Serial.println(val);
     } 
     
     while(Serial.available()<2);
     val=Serial.parseInt();
     
   }
}
```
