#define PC_BAUDRATE 9600
#define BLUETOOTH_BAUDRATE 9600
#define trigPin 13
#define echoPin 12
#define led 11
#define led2 10

#include <SoftwareSerial.h>

/*WaterLevel / Distance */
char incoming_value;
char outgoing_value;
int prev = -100;
int pHPrev = -1000;
const int buzzer = 9; //buzzer to arduino pin 9

/*PH*/
#define SensorPin A0            //pH meter Analog output to Arduino Analog Input 0
#define Offset 0.00            //deviation compensate
unsigned long int avgValue;     //Store the average value of the sensor feedback

/*Create Variables for output*/
String waterLevelString = String("WaterLevel:");
String outPutString = String();

String pHString = String("PH:");

SoftwareSerial SerialBT(4, 5); // RX, TX

void setup() {
    Serial.begin(PC_BAUDRATE);
    SerialBT.begin(BLUETOOTH_BAUDRATE);

    /* Distance Sensor setup */
    pinMode(trigPin, OUTPUT);
    pinMode(echoPin, INPUT);
    pinMode(led, OUTPUT);
    pinMode(led2, OUTPUT);
    pinMode(buzzer, OUTPUT); // Set buzzer - pin 9 as an output
}

void loop() {
  // waterph
    long duration, distance;
    digitalWrite(trigPin, LOW);  // Added this line
    delayMicroseconds(2); // Added this line
    digitalWrite(trigPin, HIGH);
  //  delayMicroseconds(1000); - Removed this line
    delayMicroseconds(10); // Added this line
    digitalWrite(trigPin, LOW);
    duration = pulseIn(echoPin, HIGH);
    distance = (duration/2) / 29.1;

    if (distance < 4) 
    {  // This is where the LED On/Off happens
        digitalWrite(led,HIGH); // When the Red condition is met, the Green LED should turn off
        digitalWrite(led2,LOW);
        tone(buzzer, 1000); // Send 1KHz sound signal...
        delay(1000);        // ...for 1 sec
        noTone(buzzer);     // Stop sound...
        delay(100);        // ...for 1sec
    }
    else 
    {
        digitalWrite(led,LOW);
        digitalWrite(led2,HIGH);
    }
  
    if (distance >= 200 || distance <= 0)
    {
        Serial.println("Out of range");
    }
    else 
    {
        if (distance != prev || prev == -100)
        {
            outPutString = waterLevelString + distance;
            
            Serial.println(outPutString);
            SerialBT.println(outPutString);
            
            prev = distance;
        }
    }


    // PH
    int buf[10]; //buffer for read analog
    for (int i = 0; i < 10; i++) //Get 10 sample value from the sensor for smooth the value
    {
        buf[i] = analogRead(SensorPin);
        delay(10);
    }
    for (int i = 0; i < 9; i++) //sort the analog from small to large
    {
        for (int j = i + 1; j < 10; j++) {
            if (buf[i] > buf[j]) {
                int temp = buf[i];
                buf[i] = buf[j];
                buf[j] = temp;
            }
        }
    }
    
    avgValue = 0;
    for (int i = 2; i < 8; i++) //take the average value of 6 center sample
    {
      avgValue += buf[i];
    }
    
    float phValue = (float) avgValue * 3.8 / 1030 / 6; //convert the analog into millivolt
    phValue = 3.3 * phValue + Offset; //convert the millivolt into pH value

    if (phValue != pHPrev || pHPrev == -1000)
    {
        outPutString = pHString + phValue;
                
        Serial.println(outPutString);
        SerialBT.println(outPutString);
    }
    delay(1000);
}
