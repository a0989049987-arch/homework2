const int pushButton = 2; // 按鈕接在 D2 腳位
const int redPin = 3;     // RGB LED 紅色 (R) 接在 D3 (PWM)
const int greenPin = 5;   // RGB LED 綠色 (G) 接在 D5 (PWM)
const int bluePin = 6;    // RGB LED 藍色 (B) 接在 D6 (PWM)

const bool commonAnode = true;
const int pressedLevel = HIGH;

int mood = 10;

int lastReading = LOW;
int stableButton = LOW;
unsigned long lastChangeTime = 0;
const unsigned long debounceTime = 30;

unsigned long lastPetTime = 0;
unsigned long lastDropTime = 0;

const unsigned long lonelyTime = 5000;
const unsigned long dropInterval = 1000;

void setup() {
  pinMode(redPin, OUTPUT);
  pinMode(greenPin, OUTPUT);
  pinMode(bluePin, OUTPUT);
  pinMode(pushButton, INPUT);

  Serial.begin(9600);

  lastReading = digitalRead(pushButton);
  stableButton = lastReading;

  lastPetTime = millis();
  lastDropTime = lastPetTime;

  showMood();
  Serial.println(mood);
}

void loop() {
  unsigned long now = millis();

  int reading = digitalRead(pushButton);

  if (reading != lastReading) {
    lastChangeTime = now;
    lastReading = reading;
  }

  if (now - lastChangeTime >= debounceTime) {
    if (reading != stableButton) {
      stableButton = reading;

      if (stableButton == pressedLevel) {
        mood = min(mood + 1, 20);

        lastPetTime = now;
        lastDropTime = now;

        Serial.println(mood);
      }
    }
  }
  if (now - lastPetTime >= lonelyTime &&
      now - lastDropTime >= dropInterval) {

    mood = max(mood - 1, 0);
    lastDropTime = now;

    Serial.println(mood);
  }

  showMood();
}

void showMood() {
  int red;
  int green;
  int blue;

  if (mood <= 10) {
    red = map(mood, 0, 10, 255, 0);
    green = map(mood, 0, 10, 0, 255);
    blue = 0;
  } else {
    red = 0;
    green = map(mood, 10, 20, 255, 0);
    blue = map(mood, 10, 20, 0, 255);
  }

  if (commonAnode) {
    red = 255 - red;
    green = 255 - green;
    blue = 255 - blue;
  }

  analogWrite(redPin, red);
  analogWrite(greenPin, green);
  analogWrite(bluePin, blue);
}
