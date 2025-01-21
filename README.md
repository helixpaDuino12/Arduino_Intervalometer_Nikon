// Arduino_Intervalometer_Nikon
// This code is for DIY budget friendly intervalometer

#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int pb[4] = {2, 3, 4, 5};
const int ir = 6; //Nikon IR Remote
int lcdXval1[3] = {0, 3, 6};
int lcdXval2[2] = {0, 1};
int lcdYval[2] = {0, 1};
int pbPin;
int okVal = 0;
int nSelect = 0;
int mSelect = 0;
int menu = 0;
int secNum = 0;
int minNum = 0;
int hourNum = 0;
int Round = 0;
String lcdTimeName[3] = {"H:", "M:", "S:"};
String lcdModeName[2] = {"Norm-L", "Expo-L"};
bool selectVal, valueUpVal, valueDownVal, startstopVal;
bool state = LOW;
unsigned long second = 0;
unsigned long minute = 0;
unsigned long hour = 0;
unsigned long pbDel = 250;
unsigned long delTime;
unsigned long time, interval;
unsigned long timeNow = 0;
unsigned long delOn = 300;
unsigned long delOff = 1000;

void pushButtonStartStop(){
  startstopVal = digitalRead(pb[3]);
  switch(startstopVal){
    case HIGH:
      if(menu == 0){
        menu = 1;
      }
      else{
        okVal++;
      }
      delay(pbDel);
      break;
    default:
      break;
  }
}

void pushButtonSelect(){
  selectVal = digitalRead(pb[0]);
  switch(selectVal){
    case HIGH:
      if(menu == 0){
        mSelect++;
      }
      else{
        nSelect++;
      }
      delay(pbDel);
      break;
    default:
      break;
  }
}

void pushButtonValue(){
  valueUpVal = digitalRead(pb[1]);
  valueDownVal = digitalRead(pb[2]);
  switch(valueUpVal){
    case HIGH:
      if(nSelect == 1){
        secNum++;
      }
      else if(nSelect == 2){
        minNum++;
      }
      else if(nSelect == 3){
        hourNum++;
      }
      delay(pbDel);
      break;
    default:
      break;
  }
  switch(valueDownVal){
    case HIGH:
      if(nSelect == 1){
        secNum--;
      }
      else if(nSelect == 2){
        minNum--;
      }
      else if(nSelect == 3){
        hourNum--;
      }
      delay(pbDel);
      break;
    default:
      break;
  }
}

void timeValue(){
  lcd.setCursor(5, lcdYval[1]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[2], lcdYval[1]);
  if(0 <= second < 10){
    lcd.print("0"+String(second));
  }
  else if(10 <= second <= 59){
    lcd.print(String(second));
  }
  lcd.setCursor(2, lcdYval[1]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[1], lcdYval[1]);
  if(0 <= minute < 10){
    lcd.print("0"+String(minute));
  }
  else if(10 <= minute <= 59){
    lcd.print(String(minute));
  }
  lcd.setCursor(lcdXval1[0], lcdYval[1]);
  if(0 <= hour < 10){
    lcd.print("0"+String(hour));
  }
  else if(10 <= hour <= 24){
    lcd.print(String(hour));
  }
}

void lcdModeViewFull(){
  lcd.setCursor(lcdXval2[1], lcdYval[0]);
  lcd.print(lcdModeName[0]);
  lcd.setCursor(lcdXval2[1], lcdYval[1]);
  lcd.print(lcdModeName[1]);
  lcd.setCursor(lcdXval2[0], lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval2[0], lcdYval[1]);
  lcd.print(" ");
}

void lcdModeViewNormL(){
  lcd.setCursor(lcdXval2[1], lcdYval[0]);
  lcd.print(lcdModeName[0]);
  lcd.setCursor(lcdXval2[1], lcdYval[1]);
  lcd.print(lcdModeName[1]);
  lcd.setCursor(lcdXval2[0], lcdYval[0]);
  lcd.print(">");
  lcd.setCursor(lcdXval2[0], lcdYval[1]);
  lcd.print(" ");
}

void lcdModeViewExpoL(){
  lcd.setCursor(lcdXval2[1], lcdYval[0]);
  lcd.print(lcdModeName[0]);
  lcd.setCursor(lcdXval2[1], lcdYval[1]);
  lcd.print(lcdModeName[1]);
  lcd.setCursor(lcdXval2[0], lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval2[0], lcdYval[1]);
  lcd.print(">");
}

void lcdTimeNameViewFull(){
  lcd.setCursor(5, lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[2], lcdYval[0]);
  lcd.print(lcdTimeName[2]);
  lcd.setCursor(2, lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[1], lcdYval[0]);
  lcd.print(lcdTimeName[1]);
  lcd.setCursor(lcdXval1[0], lcdYval[0]);
  lcd.print(lcdTimeName[0]);
  timeValue();
}

void lcdTimeNameViewSecond(){
  lcd.setCursor(5, lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[2], lcdYval[0]);
  lcd.print(lcdTimeName[2]);
  lcd.setCursor(2, lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[1], lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[0], lcdYval[0]);
  lcd.print(" ");
  if(secNum > 59){
    secNum = 0;
  }
  else if(secNum < 0){
    secNum = 59;
  }
  second = secNum;
  timeValue();
}

void lcdTimeNameViewMinute(){
  lcd.setCursor(5, lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[2], lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(2, lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[1], lcdYval[0]);
  lcd.print(lcdTimeName[1]);
  lcd.setCursor(lcdXval1[0], lcdYval[0]);
  lcd.print(" ");
  if(minNum > 59){
    minNum = 0;
  }
  else if(minNum < 0){
    minNum = 59;
  }
  minute = minNum;
  timeValue();
}

void lcdTimeNameViewHour(){
  lcd.setCursor(5, lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[2], lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(2, lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[1], lcdYval[0]);
  lcd.print(" ");
  lcd.setCursor(lcdXval1[0], lcdYval[0]);
  lcd.print(lcdTimeName[0]);
  if(hourNum > 24){
    hourNum = 0;
  }
  else if(hourNum < 0){
    hourNum = 24;
  }
  hour = hourNum;
  timeValue();
}

void lcdModeViewOverall(){
  if(mSelect == 0){
    lcdModeViewFull();
  }
  else if(mSelect == 1){
    lcdModeViewNormL();
  }
  else if(mSelect == 2){
    lcdModeViewExpoL();
  }
  else{
    if(mSelect > 2){
      mSelect = 1;
    }
  }
}

void lcdTimeNameViewOverall(){
  lcd.setCursor(10, lcdYval[0]);
  if(mSelect == 1){
    lcd.print(lcdModeName[0]);
  }
  else if(mSelect == 2){
    lcd.print(lcdModeName[1]);
  }
  if(nSelect == 0){
    lcdTimeNameViewFull();
  }
  else if(nSelect == 1){
    lcdTimeNameViewSecond();
  }
  else if(nSelect == 2){
    lcdTimeNameViewMinute();
  }
  else if(nSelect == 3){
    lcdTimeNameViewHour();
  }
  else{
    if(nSelect > 3){
      nSelect = 1;
    }
  }
}

void exposureLapse(){
  lcdTimeNameViewFull();
  if(Round == 0){
    interval = delOff;
  }
  else if(Round == 1){
    interval = delTime;
  }
  if(time - timeNow >= interval){
    Round++;
    if(Round > 1){
      Round = 0;
    }
    timeNow = time;
    if(state == LOW){
      state = HIGH;
    }
    else{
      state = LOW;
    }
    digitalWrite(ir, state);
  }
}

void normalLapse(){
  lcdTimeNameViewFull();
  if(Round == 0){
    interval = delTime;
  }
  else if(Round == 1){
    interval = delOn;
  }
  if(time - timeNow >= interval){
    Round++;
    if(Round > 1){
      Round = 0;
    }
    timeNow = time;
    if(state == LOW){
      state = HIGH;
    }
    else{
      state = LOW;
    }
    digitalWrite(ir, state);
  }
}

void delayProgram(){
  delTime = 1000 * ((hour * 3600) + (minute * 60 ) + second);
}

void modeRun(){
  pushButtonSelect();
  pushButtonValue();
  pushButtonStartStop();
  if(okVal == 0){
    Serial.println("settings");
    if(menu == 0){
      lcdModeViewOverall();
    }
    else{
      lcdTimeNameViewOverall();
    }
    digitalWrite(ir, LOW);
  }
  else if(okVal == 1){
    Serial.println("run");
    time = millis();
    delayProgram();
    nSelect = 0;
    lcdTimeNameViewOverall();
    if(mSelect == 1){
      normalLapse();
    }
    else if(mSelect == 2){
      exposureLapse();
    }
  }
  else if(okVal > 1){
    okVal = 0;
  }
  Serial.println("");
}

void setup(){
  Serial.begin(9600);
  lcd.init();
  lcd.backlight();
  pinMode(ir, OUTPUT);
  for(pbPin = 0; pbPin <= 3; pbPin++){
    pinMode(pb[pbPin], INPUT);
  }
}

void loop(){
  modeRun();
}


