# BasketballArduinoGame
This code is basketball arduino game which was our project when I was in my second year. I used r3 microcontroller, LCD with I2C, 3 red and green LEDs, breadboard, IR proximity sensor, jumper wires, and push button.


#include <Wire.h>
#include <LiquidCrystal_I2C.h>

LiquidCrystal_I2C lcd(0x27, 16, 2);

const int irSensorPin = 9;
const int buzzerPin = 8;
const int redLedPin = 7;
const int greenLedPin = 6;
const int startButtonPin = 5;

int score = 0;
int gameRound = 1;
bool gameOver = false;

const int ROUND1_SCORE = 25;
const int ROUND2_SCORE = 50;
const int ROUND3_SCORE = 75;

const unsigned long ROUND1_TIME = 40000;
const unsigned long ROUND2_TIME = 40000;
const unsigned long ROUND3_TIME = 40000;

unsigned long roundStartTime;
unsigned long currentTime;

int prevIrState = LOW;

void setup() {
  lcd.init();
  lcd.backlight();

  pinMode(irSensorPin, INPUT);
  pinMode(buzzerPin, OUTPUT);
  pinMode(redLedPin, OUTPUT);
  pinMode(greenLedPin, OUTPUT);
  pinMode(startButtonPin, INPUT_PULLUP);

  Serial.begin(115200);
  randomSeed(analogRead(0));

  lcd.setCursor(0, 0);
  lcd.print("Press button to");
  lcd.setCursor(0, 1);
  lcd.print("start game");

  while (digitalRead(startButtonPin) == HIGH);
  delay(500);
  lcd.clear();

  lcdScreen();
  Countdown();
  roundStartTime = millis();
}

void loop() {
  if (gameOver) return;

  currentTime = millis();
  unsigned long elapsedTime = currentTime - roundStartTime;
  unsigned long timeLeft = 0;

  if (gameRound == 1) {
    timeLeft = (ROUND1_TIME - elapsedTime) / 1000;
    if (elapsedTime >= ROUND1_TIME) {
      gameOverScreen("Time's Up!", "You Lose!");
      return;
    }
  }
  if (gameRound == 2) {
    timeLeft = (ROUND2_TIME - elapsedTime) / 1000;
    if (elapsedTime >= ROUND2_TIME) {
      gameOverScreen("Time's Up!", "You Lose!");
      return;
    }
  }
  else if (gameRound == 3) {
    timeLeft = (ROUND3_TIME - elapsedTime) / 1000;
    if (elapsedTime >= ROUND3_TIME) {
      gameOverScreen("Time's Up!", "You Lose!");
      return;
    }
  }

  int currentIrState = digitalRead(irSensorPin);

  if (prevIrState == LOW && currentIrState == HIGH) {
    int points = random(2, 4);
    score += points;

    digitalWrite(buzzerPin, HIGH);
    delay(100);
    digitalWrite(buzzerPin, LOW);

    delay(500);
  }

  prevIrState = currentIrState;

  lcd.setCursor(0, 0);
  lcd.print("Score: ");
  lcd.print(score);
  lcd.print("   ");

  lcd.setCursor(0, 1);
  lcd.print("Rnd ");
  lcd.print(gameRound);
  lcd.print(" Time:");
  if (timeLeft < 10) lcd.print(" ");
  lcd.print(timeLeft);

  if (gameRound == 1 && score >= ROUND1_SCORE) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("You surpass");
    lcd.setCursor(0, 1);
    lcd.print("Round 1!");
    digitalWrite(greenLedPin, HIGH);
    delay(3000);
    digitalWrite(greenLedPin, LOW);
    gameRound = 2;
    Countdown();
    roundStartTime = millis();
    lcd.clear();
  }

  if (gameRound == 2 && score >= ROUND2_SCORE) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("You surpass");
    lcd.setCursor(0, 1);
    lcd.print("Round 2!");
    digitalWrite(greenLedPin, HIGH);
    delay(3000);
    digitalWrite(greenLedPin, LOW);
    gameRound = 3;
    Countdown();
    roundStartTime = millis();
    lcd.clear();
  }

  if (gameRound == 3 && score >= ROUND3_SCORE) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("You Win!");
    lcd.setCursor(0, 1);
    lcd.print("Congrats!");

    for (int i = 0; i < 5; i++) {
      digitalWrite(buzzerPin, HIGH);
      digitalWrite(greenLedPin, HIGH);
      delay(300);
      digitalWrite(buzzerPin, LOW);
      digitalWrite(greenLedPin, LOW);
      delay(300);
    }
    delay(4000);
    gameOver = true;

    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Press button");
    lcd.setCursor(0, 1);
    lcd.print("to restart.");

    while (digitalRead(startButtonPin) == HIGH);
    delay(500);

    score = 0;
    gameRound = 1;
    gameOver = false;
    lcd.clear();
    lcdScreen();
    Countdown();
    roundStartTime = millis();
    return;
  }

  delay(100);
}

void lcdScreen() {
  lcd.setCursor(3, 0);
  lcd.print("WELCOME TO");
  lcd.setCursor(1, 1);
  lcd.print("SHOOTING GAME!");
  delay(3000);
  lcd.clear();
}

void Countdown() {
  for (int i = 3; i > 0; i--) {
    lcd.clear();
    lcd.setCursor(0, 0);
    lcd.print("Game starts in:");
    lcd.setCursor(0, 1);
    lcd.print(i);

    digitalWrite(buzzerPin, HIGH);
    delay(200);
    digitalWrite(buzzerPin, LOW);
    delay(800);
  }

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("GO!");

  digitalWrite(buzzerPin, HIGH);
  delay(500);
  digitalWrite(buzzerPin, LOW);
  delay(1000);
  lcd.clear();
}

void gameOverScreen(const char* line1, const char* line2) {
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print(line1);
  lcd.setCursor(0, 1);
  lcd.print(line2);

  if (strcmp(line1, "Time's Up!") == 0) {
    for (int i = 0; i < 5; i++) {
      digitalWrite(buzzerPin, HIGH);
      digitalWrite(redLedPin, HIGH);
      delay(500);
      digitalWrite(buzzerPin, LOW);
      digitalWrite(redLedPin, LOW);
      delay(300);
    }
  }

  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Game Over!");
  lcd.setCursor(0, 1);
  lcd.print("Try Again!");
  delay(5000);
  lcd.clear();
  lcd.setCursor(0, 0);
  lcd.print("Press button");
  lcd.setCursor(0, 1);
  lcd.print("to restart.");

  while (digitalRead(startButtonPin) == HIGH);
  delay(500);

  score = 0;
  gameRound = 1;
  gameOver = false;
  lcd.clear();
  lcdScreen();
  Countdown();
  roundStartTime = millis();
}
