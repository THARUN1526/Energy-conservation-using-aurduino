#include <SPI.h>
#include <MFRC522.h>
#include <LiquidCrystal_I2C.h>
#include <Wire.h>
#include <LiquidCrystal_PCF8574.h>
LiquidCrystal_PCF8574 lcd(0x3F);  // set the LCD address to 0x27 for a 16 chars and 2 line display

#define RST_PIN         5          // Configurable, see typical pin layout above
#define SS_PIN          53         // Configurable, see typical pin layout above

#include "RTClib.h"
String otpstring = "";

RTC_DS1307 rtc;
int hr=0;
int mi=0;
int cnt1=0;
int p1=0;
int a=0;
int b=0;
MFRC522 mfrc522(SS_PIN, RST_PIN);  // Create MFRC522 instance
String uid1 = "8A 8F 05 09";
String uid2 = "FA CA 7A 08";
String uid3 = "E7 F6 F4 3D";
String uid4 = "29 42 95 83";

#include <Keypad.h>
const byte ROWS = 4;
const byte COLS = 4;
char hexaKeys[ROWS][COLS] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'}
};
byte rowPins[ROWS] = {22,24,26,28};
byte colPins[COLS] = {30,32,34,36};
Keypad customKeypad = Keypad( makeKeymap(hexaKeys), rowPins, colPins, ROWS, COLS);

void getotp() 
{
  String y = "";
  int a = y.length();
  while (a < 4)
  {
    char customKey = customKeypad.getKey();
    if (customKey) {
      lcd.setCursor(0, 1);
      y = y + customKey;
      lcd.print(y);
      a = y.length();
    }
  }
  Serial.print("Entered OTP is ");
  Serial.println(y);
  if (y == "1234")
  {
  digitalWrite(2, LOW);   // turn the LED on (HIGH is the voltage level)
  lcd.setCursor(13,0);
  lcd.print("ON ");
  }
  
}


void setup() 
{
char daysOfTheWeek[7][12] = {"Sunday", "Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday"};
  // initialize digital pin LED_BUILTIN as an output.
  pinMode(2, OUTPUT);
  digitalWrite(2, HIGH);   // turn the LED on (HIGH is the voltage level)
  pinMode(3, OUTPUT);
  digitalWrite(3, LOW);   // turn the LED on (HIGH is the voltage level)
  
 Wire.begin();
 Wire.beginTransmission(0x3F);
 lcd.begin(16, 2);  // initialize the lcd
 lcd.setBacklight(255);
 Serial.begin(9600);   // Initiate a serial communication
 SPI.begin();      // Initiate  SPI bus
 mfrc522.PCD_Init();   // Initiate MFRC522
 Serial.println("Approximate your card to the reader...");
 Serial.println();
  if (! rtc.begin()) {
    Serial.println("Couldn't find RTC");
    Serial.flush();
    abort();
  lcd.clear();
}
 if (! rtc.isrunning()) {
    Serial.println("RTC is NOT running, let's set the time!");
    // When time needs to be set on a new device, or after a power loss, the
    // following line sets the RTC to the date & time this sketch was compiled
   // rtc.adjust(DateTime(F(__DATE__), F(__TIME__)));
    // This line sets the RTC with an explicit date & time, for example to set
    // January 21, 2014 at 3am you would call:
    // rtc.adjust(DateTime(2014, 1, 21, 3, 0, 0));
  }

}
void loop() 
{

  if(cnt1<=0)
  {
  digitalWrite(2, HIGH);   // turn the LED on (HIGH is the voltage level)
  lcd.setCursor(13,0);
  lcd.print("OFF");
  }
  if(cnt1>0)
  {
  digitalWrite(2, LOW);   // turn the LED on (HIGH is the voltage level)
  lcd.setCursor(13,0);
  lcd.print("ON ");
  }
  
   DateTime now = rtc.now();
   hr=now.hour();
   mi=now.minute();   
  lcd.setCursor(0,1);
  lcd.print(hr);
  lcd.setCursor(2,1);
  lcd.print(":");
  lcd.setCursor(3,1);
  lcd.print(mi);
 
  lcd.setCursor(0,0);
  lcd.print("Scan Tag  ");
 if((hr>9 && hr<=12) || (hr>13  && hr<16) )
 {
  lcd.setCursor(8,1); 
  lcd.print("CL");
 }
 if((hr==13)  || (hr>16) )
 {
  lcd.setCursor(8,1);
  lcd.print("BR");
 }
 
 // Reset the loop if no new card present on the sensor/reader. This saves the entire process when idle.
  if ( ! mfrc522.PICC_IsNewCardPresent()) 
  {
    return;
  }
  // Select one of the cards
//  if((hr>9 && hr<13) || (hr>14 && hr<16))
  if(hr>16)

  {
  if ( ! mfrc522.PICC_ReadCardSerial()) 
  {
    return;
  }
  //Show UID on serial monitor
  Serial.print("UID tag :");
  String content= "";
  byte letter;
  for (byte i = 0; i < mfrc522.uid.size; i++) 
  {
     Serial.print(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " ");
     Serial.print(mfrc522.uid.uidByte[i], HEX);
     content.concat(String(mfrc522.uid.uidByte[i] < 0x10 ? " 0" : " "));
     content.concat(String(mfrc522.uid.uidByte[i], HEX));
  }
  lcd.clear();
  Serial.println();
  Serial.print("Message : ");
  content.toUpperCase();
  if (content.substring(1) == uid1) //change here the UID of the card/cards that you want to give access
  {
 digitalWrite(3, HIGH);   // turn the LED on (HIGH is the voltage level)
 delay(500);
 digitalWrite(3, LOW);   // turn the LED on (HIGH is the voltage level)
 
    Serial.println("Authorized access");
    Serial.println();
    lcd.setCursor(0,0);
    lcd.print("STUDENT 1");
    a=!a;
   // lcd.setCursor(12,0);
  ///  lcd.print(a);
  if(a==1)
  {
   cnt1=cnt1+1; 
  }
  if(a==0)
  {
   cnt1=cnt1-1; 
  }
    lcd.setCursor(14,1);
    lcd.print(cnt1);
    delay(2000);
 
  }

 else if (content.substring(1) == uid2) //change here the UID of the card/cards that you want to give access
  {
 digitalWrite(3, HIGH);   // turn the LED on (HIGH is the voltage level)
 delay(500);
 digitalWrite(3, LOW);   // turn the LED on (HIGH is the voltage level)
    Serial.println("Authorized access");
    Serial.println();
    lcd.setCursor(0,0);
    lcd.print("STUDENT 2");
    b=!b;
  //  lcd.setCursor(12,0);
  //  lcd.print(b);
  if(b==1)
  {
   cnt1=cnt1+1; 
  }
  if(b==0)
  {
   cnt1=cnt1-1; 
  }
  lcd.setCursor(14,1);
  lcd.print(cnt1);
  delay(2000);
  }
  else if (content.substring(1) == uid3) //change here the UID of the card/cards that you want to give access
  {
    Serial.println("Authorized access");
    Serial.println();
    lcd.setCursor(0,0);
    lcd.print("TEACHER");
      getotp();  
delay(2000);
  }

 /*else if (content.substring(1) == uid4) //change here the UID of the card/cards that you want to give access
  {
    Serial.println("Authorized access");
    Serial.println();
    lcd.setCursor(0,0);
    lcd.print("Tag 04");
    lcd.setCursor(0,1);
    lcd.print("Authorized access");
    delay(3000);
    lcd.clear();
  }*/
 else   {
    Serial.println(" Access Denied");
    lcd.setCursor(3,0);
    lcd.clear();
    lcd.print("Unknown");
    lcd.setCursor(0,1);
    lcd.print("Access Denied");
    delay(3000);
    lcd.clear();
  }
}
} 