int EnteredPin = 0, pinog = 0, seconds = 0;
int PinSent = 1, DisplayForPin = 0, EnteredKey;
int count = 0;
bool Lock = false;
unsigned long previousMillis = 0;

#include <SoftwareSerial.h> // Bluetooth
SoftwareSerial bluetooth(12, 13); // RX, TX
#include <Servo.h> // Servo
Servo motor;
#include <Wire.h> // LCD
#include <LiquidCrystal_I2C.h>
LiquidCrystal_I2C lcd(0x27, 16, 2);
// Keypad
#include <Keypad.h>
const byte rows = 4;
const byte columns = 4;
char octaKey[rows][columns] = {
  {'1', '2', '3', 'A'},
  {'4', '5', '6', 'B'},
  {'7', '8', '9', 'C'},
  {'*', '0', '#', 'D'},
};
byte RowPins[rows] = {9, 8, 7, 6};
byte ColPins[columns] = {5, 4, 3, 2};
Keypad customKeypad = Keypad(makeKeymap(octaKey), RowPins, ColPins, rows, columns);

void displayArray(char* array, int length) {
  Serial.print("[ ");
  for (int i = 0; i < length; i++) {
    Serial.print(array[i]);
    Serial.print(" ");
  }
  Serial.println("]");
}

void encryptPIN(char* genpin, char* encryptedPIN, char key) {
  // XOR cipher
  for (int i = 0; i < 5; i++) {
    encryptedPIN[i] = genpin[i] ^ key;
  }

  Serial.print("XOR'ed PIN: ");
  displayArray(encryptedPIN, 5);

  // Bit rotate (left rotation by 3 bits)
  for (int i = 0; i < 5; i++) {
    encryptedPIN[i] = ((encryptedPIN[i] << 3) | ((unsigned char)encryptedPIN[i] >> 5)); // Circular left shift by 3 bits
  }

  Serial.print("Bit rotated PIN: ");
  displayArray(encryptedPIN, 5);

  encryptedPIN[5] = '\0'; // Null terminator to end the string
}

void setup() {
  bluetooth.begin(9600);
  lcd.init();
  lcd.backlight();
  Serial.begin(9600);
  randomSeed(analogRead(0));
  motor.attach(10);
  motor.write(90);
  delay(150);
}

void Display() { // LCD display
  lcd.setCursor(0, 0);
  lcd.print("PIN SENT TO APP");
  delay(2000);
  lcd.clear();
  lcd.print("ENTER PIN");
  lcd.setCursor(0, 1);
  PinSent = 0;
}

int arrtoint(char* a) { // Function to convert char array to int
  int sum = 0;
  for (int i = 0; i < 5; i++) {
    sum = sum * 10 + (a[i] - '0');
  }
  return sum;
}

void CheckPin(int a, int b) { // Pin check
  if (a == b && a != 0) {
    lcd.setCursor(0, 0);
    lcd.print("PIN IS CORRECT");
    motor.write(0);
    delay(150);
    lcd.setCursor(0, 1);
    lcd.print("OPENING VAULT");
    delay(2000);
    EnteredPin = 0;
    Lock = true;
    lcd.clear();
    lcd.print("VAULT OPENED");
  }
}

void LockVault() { // Locking vault
  char Key = customKeypad.getKey();
  if (Key) {
    if (Key == '#') {
      lcd.clear();
      delay(1000);
      lcd.print("LOCKING VAULT");
      motor.write(90);
      delay(750);
      delay(150);
      Lock = false;
      lcd.clear();
      lcd.print("VAULT LOCKED");
    }
  }
}

void loop() {
  static unsigned long previousMillis = 0;
  static int seconds = 0;

  unsigned long currentMillis = millis();
  if (currentMillis - previousMillis >= 1000) {
    previousMillis = currentMillis;
    seconds++;
  }

  if (seconds == 6 || seconds == 0) { // Generation of PIN
    char genpin[6]; // 5 characters for PIN + 1 for null terminator
    for (int i = 0; i < 5; i++) {
      genpin[i] = '0' + random(10); // Generate a random digit (0-9)
    }
    genpin[5] = '\0'; // Null terminator to end the string

    pinog = arrtoint(genpin); // Converts array to int for comparison

    // Print the generated PIN
    Serial.print("Random PIN: ");
    Serial.println(genpin);

    // Encrypt the PIN
    char key = 'z';
    char encryptedPIN[6];
    encryptPIN(genpin, encryptedPIN, key);

    // Print the encrypted PIN
    Serial.print("Encrypted PIN: ");
    Serial.println(encryptedPIN);

    // Send encrypted PIN via Bluetooth
    bluetooth.print("PIN is: ");
    bluetooth.println(encryptedPIN);

    seconds = 1;
  }

  if (PinSent) {
    Display(); // Display call
  }

  char Key = customKeypad.getKey(); // Input for pin
  if (Key != NO_KEY) {
    if (Key != '*') {
      lcd.print(Key);
      EnteredKey = Key;
      EnteredPin = EnteredPin * 10 + (EnteredKey - '0');
      Serial.println(EnteredPin);
    } else if (Key == '*') { // Reset pin entry
      lcd.clear();
      lcd.setCursor(0, 0);
      EnteredPin = 0;
      lcd.print("RE Enter pin");
      lcd.setCursor(0, 1);
    }
  }

  CheckPin(EnteredPin, pinog); // Unlocking call
  if (Lock) {
    LockVault();
  }
}
