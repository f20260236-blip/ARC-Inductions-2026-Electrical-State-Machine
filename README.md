** Master Code **

      #include <IRremote.hpp>
      #include <LiquidCrystal.h>

   // Pins for the LiquidCrystal display
      LiquidCrystal lcd(12, 11, 5, 4, 3, 2); 

     const int ir = A0; 

     String lastDisplayMsg = "";

     void setup() {
     Serial.begin(9600); 
  
    lcd.begin(16, 2);
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("AWAITING RITUAL");

     IrReceiver.begin(ir, ENABLE_LED_FEEDBACK); 
                }

  void loop() {

        if (IrReceiver.decode()) {
          unsigned long hexVal = IrReceiver.decodedIRData.decodedRawData;

        if (hexVal == 0xFF00BF00) {
            Serial.write(0); 
            }
        else if (hexVal == 0xEF10BF00) {
        Serial.write(1); 
            } 
        else if (hexVal == 0xEE11BF00) {
        Serial.write(2); 
            }
    
    
        IrReceiver.resume();
            }
        if (Serial.available() > 0) {
        String incomingMsg = Serial.readStringUntil('\n');
        incomingMsg.trim(); 

        if (incomingMsg.length() > 0 && incomingMsg != lastDisplayMsg) {
        lastDisplayMsg = incomingMsg;
        lcd.clear();
        lcd.setCursor(0, 0);
        lcd.print(incomingMsg);
    }
  }
}

** Slave Code ** 


   #include <Servo.h>

    Servo myservo;
    const int ldr = A0;
          int gas = A2;
    const int temp = A1;
    const int buzzer = 2;
    const int servo = 3;
          int currentState = 0;  
          int displayToggle = 0; // 0 = Light level, 1 = Gas level
    const int DARK_THRESHOLD = 100; 
    
    void setup() 
    {
      Serial.begin(9600);
      myservo.attach(servo);
      myservo.write(0);
      
      pinMode(ldr, INPUT);
      pinMode(gas, INPUT);
      pinMode(temp, INPUT);
      pinMode(buzzer, OUTPUT);
      digitalWrite(buzzer, LOW);
    }
    
    void loop()
    {
      int r_gas = analogRead(gas);
      int r_ldr = analogRead(ldr);
      int rawTemp = analogRead(temp);
      float voltage = rawTemp * (5.0 / 1024.0); 
      float r_temp = (voltage - 0.5) * 100.0; 
      
    
      if (Serial.available() > 0) {
        int cmd = Serial.read();
    
        if (cmd == 0) { 
          currentState = 0;   
        } 
        else if (cmd == 1) { 
          currentState = 1;   
          displayToggle = 0; 
        }
        else if (cmd == 2) {
          currentState = 1;  
          displayToggle = 1;  
        }
      }
    
     
      if (r_temp > 45.0) {
        myservo.write(180); 
        digitalWrite(buzzer, HIGH);
        Serial.println("COOKED");
      } 
      else if (r_gas > 180) {
        myservo.write(0);
        Serial.println("TOXIC PURGE");
         digitalWrite(buzzer, LOW);
      } 
      else if (r_ldr < DARK_THRESHOLD) {
        myservo.write(0);
        Serial.println("NOCTIS PROTOCOL");
         digitalWrite(buzzer, LOW);
      } 
      else if (currentState == 1) {
        myservo.write(0);
        if (displayToggle == 0) {
          Serial.print("LIGHT: ");
          Serial.println(r_ldr);
           digitalWrite(buzzer, LOW);
        } else {
          Serial.print("GAS: ");
          Serial.println(r_gas);
           digitalWrite(buzzer, LOW);
        }
      } 
      else {
        myservo.write(0);
        Serial.println("AWAITING RITUAL");
         digitalWrite(buzzer, LOW);
      }

        delay(100); 
      }

// https://www.tinkercad.com/things/fj6TCtQAYUZ-frantic-jarv/editel?returnTo=https%3A%2F%2Fwww.tinkercad.com%2Fdashboard&sharecode=B02Bv2LmOSFr0_K8V5iJcNLzZphthaHF60YT1Z0RxjU 
(Tinkercad link)
