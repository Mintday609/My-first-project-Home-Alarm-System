# My-first-project-Home-Alarm-System
i was looking through projecthub.arduino but i found smt https://projecthub.arduino.cc/1higgin/simple-alarm-system-e5958f this looked so cool so i tryed to make it and it work (btw i used chatgpt for the codeing cuz i am not that good at it i made the rest and i will show you how to make it its rlly simple btw this is for a arduino uno or nano and use arduino IDE
this is the first code i do not recommend using it but the second code use it
#include <LiquidCrystal.h>

// LCD pin mapping: RS, E, D4, D5, D6, D7
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// Pin definitions
const int trigPin = 9;
const int echoPin = 10;
const int buzzerPin = 8;
const int armBtn = 6;
const int disarmBtn = 7;

// Variables
bool armed = false;
bool triggered = false;
long duration;
int distance;
int threshold = 20; // Distance in cm before alarm triggers

// Delays (in seconds)
int exitDelay = 5;   // Time to leave after arming
int entryDelay = 5;  // Time to disarm after detection

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(buzzerPin, OUTPUT);
  
  pinMode(armBtn, INPUT_PULLUP);
  pinMode(disarmBtn, INPUT_PULLUP);

  lcd.begin(16, 2);
  lcd.print("Alarm System");
  delay(2000);
  lcd.clear();
}

void loop() {
  // Check buttons
  if (digitalRead(armBtn) == LOW && !armed) {
    lcd.clear();
    lcd.print("Arming in...");
    for (int i = exitDelay; i > 0; i--) {
      lcd.setCursor(0, 1);
      lcd.print(i);
      lcd.print(" sec       ");
      delay(1000);
    }
    armed = true;
    triggered = false;
    lcd.clear();
    lcd.print("System ARMED");
    delay(500);
  }

  if (digitalRead(disarmBtn) == LOW) {
    armed = false;
    triggered = false;
    noTone(buzzerPin);
    lcd.clear();
    lcd.print("System DISARMED");
    delay(500);
  }

  // If armed, check sensor
  if (armed && !triggered) {
    distance = getDistance();
    lcd.setCursor(0, 1);
    lcd.print("Dist: ");
    lcd.print(distance);
    lcd.print("cm   ");
    
    if (distance > 0 && distance < threshold) {
      // Entry delay before alarm
      lcd.setCursor(0, 0);
      lcd.print("Entry delay...");
      for (int i = entryDelay; i > 0; i--) {
        lcd.setCursor(12, 0);
        lcd.print(i);
        lcd.print("s ");
        delay(1000);
        if (digitalRead(disarmBtn) == LOW) { // Disarm in time
          armed = false;
          lcd.clear();
          lcd.print("Disarmed in time");
          delay(1000);
          return;
        }
      }
      // Alarm triggered
      triggered = true;
    }
  }

  // Alarm state
  if (armed && triggered) {
    lcd.setCursor(0, 0);
    lcd.print("!!! INTRUDER !!!");
    tone(buzzerPin, 1000);
  }

  // Idle display
  if (!armed) {
    lcd.setCursor(0, 0);
    lcd.print("Idle Mode       ");
    lcd.setCursor(0, 1);
    lcd.print("Press ARM Btn   ");
  }
}

// Function to measure distance
int getDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH, 20000); // Timeout 20ms
  if (duration == 0) return -1; // No reading
  return duration * 0.034 / 2;
}
 second code its beter to use this one

 #include <LiquidCrystal.h>

// LCD pin mapping: RS, E, D4, D5, D6, D7
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);

// Pin definitions
const int trigPin = 9;
const int echoPin = 10;
const int buzzerPin = 8;
const int armBtn = 6;
const int unarmBtn = 7;

// Variables
bool armed = false;
bool triggered = false;
long duration;
int distance;

// Thresholds
int warningThreshold = 30;
int intruderThreshold = 20;

// Delays
int exitDelay = 5; // seconds countdown for arming

// Timing
unsigned long previousBlink = 0;
bool blinkState = false;

// Startup
unsigned long startupStart = 0;
bool startupDone = false;

// Siren sweep
int sirenFreq = 500;
int sirenDir = 20;
unsigned long previousSiren = 0;

void setup() {
  pinMode(trigPin, OUTPUT);
  pinMode(echoPin, INPUT);
  pinMode(buzzerPin, OUTPUT);

  pinMode(armBtn, INPUT_PULLUP);
  pinMode(unarmBtn, INPUT_PULLUP);

  lcd.begin(16, 2);

  startupStart = millis(); // mark startup time
}

void loop() {
  unsigned long now = millis();

  // -------- STARTUP ANIMATION (8s total, 5 smooth dots) --------
  if (!startupDone) {
    int elapsed = now - startupStart;
    int totalDots = 5;
    int dotStep = elapsed / 160; // ~160ms per dot
    if (dotStep > totalDots) dotStep = totalDots;

    lcd.setCursor(0, 0);
    lcd.print("Starting Up    ");
    lcd.setCursor(0, 1);
    lcd.print("                "); // clear row
    lcd.setCursor(0, 1);
    for (int i = 0; i < dotStep; i++) lcd.print("."); // smooth dots

    if (elapsed >= 8000) { // stop after 8s
      startupDone = true;
      armed = false;
      triggered = false;
      lcd.clear();
    }
    return; // skip rest until startup finishes
  }

  // -------- CHECK UNARM BUTTON (only works if armed or triggered) --------
  if ((armed || triggered) && digitalRead(unarmBtn) == LOW) {
    armed = false;
    triggered = false;
    noTone(buzzerPin);

    // UNARM sequence loading bar
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("UNARMING...");
    lcd.setCursor(0,1);
    lcd.print("[         ]");
    for(int i=1; i<=9; i++){
      lcd.setCursor(i,1);
      lcd.print("=");
      tone(buzzerPin, 700, 80);
      delay(100);
    }
    noTone(buzzerPin);
    lcd.clear();
    lcd.setCursor(0,0);
    lcd.print("SYSTEM UNARMED");
    // blinking [ARM] on bottom
    previousBlink = now; 
    blinkState = true;
    return;
  }

  // -------- IDLE / UNARMED --------
  if (!armed && !triggered) {
    lcd.setCursor(0,0);
    lcd.print("SYSTEM UNARMED ");

    // Continuous blinking [ARM]
    if (now - previousBlink >= 500) {
      previousBlink = now;
      blinkState = !blinkState;
    }
    lcd.setCursor(0,1);
    if (blinkState) lcd.print("[ARM]         ");
    else lcd.print("               ");

    // Check arm button
    if (digitalRead(armBtn) == LOW) {
      // ARMED loading bar sequence
      lcd.clear();
      lcd.setCursor(0,0);
      lcd.print("ARMING...");
      lcd.setCursor(0,1);
      lcd.print("[         ]");
      for(int i=1;i<=9;i++){
        lcd.setCursor(i,1);
        lcd.print("=");
        tone(buzzerPin, 1000, 100);
        delay(100);
      }
      noTone(buzzerPin);
      armed = true;
      triggered = false;
      lcd.clear();
      return;
    }
    return; // keep looping idle
  }

  // -------- ARMED STATE --------
  if (armed && !triggered) {
    distance = getDistance();
    lcd.setCursor(0,0);
    lcd.print("SYSTEM ARMED   ");

    lcd.setCursor(0,1);
    lcd.print("Dist:");
    lcd.print(distance);
    lcd.print("cm ");

    // Blink UNARM at end of bottom row
    if (now - previousBlink >= 500) {
      previousBlink = now;
      blinkState = !blinkState;
    }
    lcd.setCursor(11,1);
    if (blinkState) lcd.print("UNARM");
    else lcd.print("     ");

    // Distance thresholds
    if (distance > 0 && distance < intruderThreshold) {
      triggered = true;
    } else if (distance > 0 && distance < warningThreshold) {
      tone(buzzerPin, 800, 100); // soft warning beep
    }
  }

  // -------- INTRUDER ALARM --------
  if (armed && triggered) {
    // Blink !!! INTRUDER !!!
    if (now - previousBlink >= 500) {
      previousBlink = now;
      blinkState = !blinkState;
    }
    lcd.setCursor(0,0);
    if (blinkState) lcd.print("!!! INTRUDER !!!");
    else lcd.print("                 ");
    lcd.setCursor(0,1);
    lcd.print("                ");

    // Gradual siren sweep (non-blocking)
    if (now - previousSiren >= 20) {
      previousSiren = now;
      tone(buzzerPin, sirenFreq);
      sirenFreq += sirenDir;
      if (sirenFreq >= 1200 || sirenFreq <= 500) sirenDir = -sirenDir;
    }
  }
}

// -------- FUNCTION TO MEASURE DISTANCE --------
int getDistance() {
  digitalWrite(trigPin, LOW);
  delayMicroseconds(2);
  digitalWrite(trigPin, HIGH);
  delayMicroseconds(10);
  digitalWrite(trigPin, LOW);

  duration = pulseIn(echoPin, HIGH, 20000);
  if (duration == 0) return -1;
  return duration * 0.034 / 2;
}
