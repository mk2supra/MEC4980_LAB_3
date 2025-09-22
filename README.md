//# MEC4980_LAB_3
//LAB 3 Thermostat
// couldn't find out how to upload this correctly so here is this
// Working CODE !!!!!!!!!!!!
#define MICRO
//#define NARROW
//#define TRANSPARENT

//////////////////////////////////////////////////////////////////////////////////////////

#include <stdint.h>
//include BME280
#include <Wire.h>
#include <SPI.h>
#include <Adafruit_Sensor.h>
#include <Adafruit_BME280.h>

// Include the SparkFun qwiic OLED Library
#include <SparkFun_Qwiic_OLED.h>

#define SEALEVELPRESSURE_HPA (1013.25)

Adafruit_BME280 bme;

#if defined(TRANSPARENT)
QwiicTransparentOLED myOLED;
const char * deviceName = "Transparent OLED";

#elif defined(NARROW)
QwiicNarrowOLED myOLED;
const char * deviceName = "Narrow OLED";

#else
QwiicMicroOLED myOLED;
const char * deviceName = "Micro OLED";

#endif

int yoffset;

float targetTemperature=20.0;
char degreeSys[]="C";
int pinButtonGREEN=10;
bool prevPressed=0;
bool prevUp=false;
bool prevDown=false;
bool useFahrenheit=false; //also for unit swap
bool prevUnitChange=false; // for unit swaop

int pinButtonBLUE=11;
int pinButtonRED=12;
int pinREDLED=13;

enum MachineStates{
  DisplayTemps, //actually equal to 0
  SetTemp, // 1
  ChooseSystem //2
};

MachineStates currentState;

////////////////////////////////////////////////////////////////////////////////////////////////
// setup()
// 
// Standard Arduino setup routine

void setup()
{
  currentState = DisplayTemps;
  pinMode(pinButtonGREEN, INPUT_PULLDOWN);
  pinMode(pinButtonBLUE, INPUT_PULLDOWN);
  pinMode(pinButtonRED, INPUT_PULLDOWN);
  pinMode(pinREDLED, OUTPUT);

  Serial.begin(9600);
  delay(3000);
  Serial.println("Testing BME sensor");
  //delay to fix code error hopefully
  //BME280 start up stuf
  if (! bme.begin(0x77, &Wire)) {
        Serial.println("Could not find a valid BME280 sensor, check wiring!");
        while (1);
    }

    delay(500);   //Give display time to power on
    
    Serial.println("\n\r-----------------------------------");

    Serial.print("Running Test #5 on: ");
    Serial.println(String(deviceName));

    if(!myOLED.begin()){

        Serial.println("- Device Begin Failed");
        while(1);
    }

    yoffset = (myOLED.getHeight() - myOLED.getFont()->height)/2;

    delay(1000);
}

float cToF(float degC){
  return (degC*1.8)+32.0;
}
/////////////////////////////////////////////////////////////////////////////////////////////////
/////////////////////////////////////////////////////////////////////

void loop()
{
  float temp=bme.readTemperature(); //move up here so I can use temp later on
  float tempF=cToF(temp); //calcs other unit temp
  float targetTempF=cToF(targetTemperature); //^^^^
  bool bothPressed=digitalRead(pinButtonBLUE) && digitalRead(pinButtonRED);
  
  // if the target temp is greater than current for any state turn on heat LED signal
  if (targetTemperature>temp){
        digitalWrite(pinREDLED, HIGH);
      }else{
        digitalWrite(pinREDLED, LOW);
      }

  if(digitalRead(pinButtonGREEN) && !prevPressed){
    currentState= MachineStates(((int)currentState+1)%3);   //if adds beyond 2 it will crash so add mod function %3 to drop to zero
}
prevPressed=digitalRead(pinButtonGREEN);//this line fixed the super fast button press issue causing the fast cycling through states
char myNewText [50];

if(digitalRead(pinButtonRED) && !prevUp){
}


if(currentState == DisplayTemps){

    //sprintf(myNewText, "TC: %.1f", temp);
    // if useFahrenheit is true, show F, else show C
    sprintf(myNewText, "TC: %.1f", useFahrenheit? tempF: temp);

    myOLED.erase();
    myOLED.text(3, yoffset +1, myNewText); //text height scroll CHANGE THIS LATER
        
    //sprintf(myNewText, "Ttar: %.1f", targetTemperature);
    //// if useFahrenheit is true, show F, else show C
    sprintf(myNewText, "Ttar: %.1f", useFahrenheit?  targetTempF: targetTemperature);
    myOLED.display();
    
  }else if(currentState == SetTemp){
    if(digitalRead(pinButtonRED) && !prevUp){
      targetTemperature++;
    }
    if(digitalRead(pinButtonBLUE) && !prevDown){
      targetTemperature--;
    }
    prevUp=digitalRead(pinButtonRED);
    prevDown=digitalRead(pinButtonBLUE);
    //sprintf(myNewText, "Ttar: %.1f", targetTemperature);
    //// if useFahrenheit is true, show F, else show C
    sprintf(myNewText, "Ttar: %.1f", useFahrenheit?  targetTempF: targetTemperature);
    myOLED.erase();
    myOLED.text(3, yoffset +1, myNewText);
    myOLED.display();
    
  }else if(currentState == ChooseSystem){
    // if in choose system and both red and blue press = unit swap
      if (bothPressed && !prevUnitChange){
        delay(200); //delay to stop flash
        useFahrenheit=!useFahrenheit;
      }
    //sprintf(myNewText, "System: %s", degreeSys);
    // if useFahrenheit is true, show F, else show C
    sprintf(myNewText, "System: %s", useFahrenheit ? "F" : "C");
    myOLED.erase();
    myOLED.text(3, yoffset, myNewText);
    myOLED.display();

  };
  
};
