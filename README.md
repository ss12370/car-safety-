#include <Wire.h>
#include <Adafruit_Sensor.h> 
#include <Adafruit_ADXL345_U.h>
Adafruit_ADXL345_Unified accel = Adafruit_ADXL345_Unified();

#include<Wire.h>
#include<LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27,16,2);

int relay=3;


int ldr=17;
int ir=9;

int triger=15;    // front uv sensor
int echo=16;
int buzzer=8;

int gas=A0;


long duration;
int distance;



int m1=6;
int m2=7;
int m3=5;
int m4=4;
char ch;

float latitude=13.2064132;
float longitude=77.3767946;




void setup()
{
  Serial.begin(9600);
//  Serial2.begin(9600);

  pinMode(gas,INPUT);
  
  pinMode(ir,INPUT);
  pinMode(buzzer,OUTPUT);
  lcd.init();
  lcd.backlight();

  lcd.clear();
  pinMode(relay,OUTPUT);
  
  
  pinMode(ldr,INPUT);
  pinMode(triger,OUTPUT);
  pinMode(echo,INPUT);
  
  pinMode(m1,OUTPUT);
  pinMode(m2,OUTPUT);
  pinMode(m3,OUTPUT);
  pinMode(m4,OUTPUT);
  
  Serial.println("CAR SAFETY AND SECURITY FEATURES");
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Car Safety");
  lcd.setCursor(0,1);
  lcd.print("Security System");
//  SEND_SMS("+916363355977","Multifunctional Vehicle Security System");
  delay(2000);
 if(!accel.begin())
{

      Serial.println("No valid sensor found");
      delay(500);
      while(1);

   }  
//  Serial.begin(9600);

 digitalWrite(m1,LOW);
 digitalWrite(m2,LOW);
 digitalWrite(m3,LOW);
 digitalWrite(m4,LOW);
 digitalWrite(relay,LOW);
 
    FORWARD();


   
   
}

void loop()
{
  FRONT_UV_SENSOR();
  LDR_CHECK();
  IR_CHECK();
  ADXL();
  FRONT_UV_SENSOR();
  gas_SENSOR();
  FRONT_UV_SENSOR();
  serialEvent();
  
}

void serialEvent()
{
  if(Serial.available()>0)
  {
    char ch=Serial.read();
    Serial.println(ch);
    delay(500);
    if(ch=='A')
    {
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("Drowziness ");
      lcd.setCursor(0,1);
      lcd.print("Detected");
      digitalWrite(buzzer,HIGH);
      delay(1000);
      digitalWrite(buzzer,LOW);
      delay(1000);
    }
  }
}

void ADXL()
{
  sensors_event_t event; 

   accel.getEvent(&event);

  float X_val=event.acceleration.x;

  float Y_val=event.acceleration.y;

  float Z_val=event.acceleration.z;
   

   Serial.print("X: "); Serial.print(event.acceleration.x); Serial.print("  ");
//
   Serial.print("Y: "); Serial.print(event.acceleration.y); Serial.print("  ");
//
   Serial.print("Z: "); Serial.print(event.acceleration.z); Serial.print("  ");
//
//   Serial.println("m/s^2 ");
    delay(500);

   
   
   
   
   
   
   if((X_val<-7.50)||(X_val>7.5)||(Y_val<-7.50)||(Y_val>7.5))
   {
    
   Serial.println("$Accident Occured#");
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Accident");
    lcd.setCursor(0,1);
    lcd.print(" Occured ");
    digitalWrite(buzzer,HIGH);
    delay(1000);
    digitalWrite(buzzer,LOW);
    delay(1000);
    while(1)
    {
      STOP();
    }
  }
  serialEvent();

   }
   

void IR_CHECK()
{
  if(digitalRead(ir)==LOW)
  {
    Serial.println("Blind Spot Detected");
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Blind Spot");
    lcd.setCursor(0,1);
    lcd.print(" Detected  ");
    digitalWrite(buzzer,HIGH);
    delay(1200);
    digitalWrite(buzzer,LOW);
    delay(1000);
  }
  serialEvent();
}






void LDR_CHECK()
{
  int ldrval=digitalRead(ldr);
  Serial.println("LDR:"+String(ldrval));
  delay(500);
if(ldrval==HIGH)
{
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Low Light");
  lcd.setCursor(0,1);
  lcd.print("Detected..");
  digitalWrite(relay,HIGH);
  delay(1000);
  
}
else
{
  digitalWrite(relay,LOW);
  delay(500);
}




}



void FRONT_UV_SENSOR()
{
  digitalWrite(triger,LOW);  
  delayMicroseconds(2);
  digitalWrite(triger,HIGH);  
  delayMicroseconds(10);
  digitalWrite(triger,LOW);
  duration=pulseIn(echo,HIGH);
  distance=duration*0.034/2;  
  Serial.print("Distance:");
  Serial.println(distance);
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Distance:");
  lcd.setCursor(10,0);
  lcd.print(distance);
  delay(1000);
  if(distance>0 && distance<30)
  {
    STOP();
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("Object Detected");
    lcd.setCursor(0,1);
    lcd.print("Front Side");
//    Serial.println("Object Detected");
    delay(1000);
  }
  else
  {
    FORWARD();
  }

  
  
}




void gas_SENSOR()
{
  int smokeval=analogRead(gas);
//  Serial.println("Smoke:"+String(smokeval));
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Alcohol:"+String(smokeval));
  delay(1000);
  if(smokeval>250)
  {
    
  lcd.clear();
  lcd.setCursor(0,0);
  lcd.print("Alcohol Detected");
  Serial.println("$Alcohol Detected#");
  digitalWrite(buzzer,HIGH);
  delay(1200);
  digitalWrite(buzzer,LOW);
  delay(1000);
  STOP();
  delay(2500);
  
  FORWARD();
  }
 
  
}


void FORWARD()
{
  digitalWrite(m1,HIGH);
  digitalWrite(m2,LOW);
  digitalWrite(m3,HIGH);
  digitalWrite(m4,LOW);
  delay(1000);
}

void STOP()
{
  digitalWrite(m1,LOW);
  digitalWrite(m2,LOW);
  digitalWrite(m3,LOW);
  digitalWrite(m4,LOW);
  delay(1000);
}
