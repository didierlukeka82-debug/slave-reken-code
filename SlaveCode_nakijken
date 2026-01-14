#include <CommunicatieLib.h>
#include <MD_MAX72xx.h>
#include <MFRC522.h>
#include <SPI.h>
#include <TM1637Display.h>

 // TM1637 Display

TM1637Display display(CLK, DIO);
#define CLK 13
#define DIO 12


/* ================== RFID ================== */
#define RST_PIN 22
#define SS_PIN  21


MFRC522 mfrc522(SS_PIN, RST_PIN);
MFRC522::MIFARE_Key key;

/* ================== LED MATRIX ================== */
#define HARDWARE_TYPE MD_MAX72XX::FC16_HW
#define DATA_PIN 12
#define CLK_PIN  13
#define CS_PIN   14
#define MAX_DEVICES 1

MD_MAX72XX mx(HARDWARE_TYPE, DATA_PIN, CLK_PIN, CS_PIN, MAX_DEVICES);

/* ================== COMMUNICATIE ================== */
CommunicatieLib communicatie(1, 2);
Message msg;

/* ================== GAME VARIABELEN ================== */
int a = 0;
int b = 0;
int getal = 0;

bool aGelezen = false;
bool bGelezen = false;

bool rfidtagSelected = true;   // ← zet op true als RFID actief is
bool tagSelected = false;

byte uid[10];
byte uidSize;

Operators ontvangenOperators[3];
bool operatorBekend = false;

/* ================== LED PATRONEN ================== */
uint8_t plusP[8] = {
  0b00011000,0b00011000,0b00011000,0b11111111,
  0b11111111,0b00011000,0b00011000,0b00011000
};

uint8_t minP[8] = {
  0b00000000,0b00000000,0b00000000,0b11111111,
  0b11111111,0b00000000,0b00000000,0b00000000
};

uint8_t gedeeldP[8] = {
  0b00000000,0b00011000,0b00011000,0b00000000,
  0b00000000,0b00011000,0b00011000,0b00000000
};

uint8_t *activePattern = nullptr;
unsigned long showUntil = 0;

/* ================== FUNCTIES ================== */
int rfidNaarGetal(byte *uid, byte uidSize) {
  int waarde = 0;
  for (byte i = 0; i < uidSize; i++) {
    waarde += uid[i];
  }
  return waarde % 10;
}

int berekenSom(int x, int y, Operators op) {
  switch (op) {
    case OP_PLUS:    return x + y;
    case OP_MIN:     return x - y;
    case OP_GEDEELD: return (y != 0) ? x / y : 0;
  }
  return 0;
}

void toonOperator(Operators op, unsigned long duurMs) {
  switch (op) {
    case OP_PLUS:    activePattern = plusP; break;
    case OP_MIN:     activePattern = minP; break;
    case OP_GEDEELD: activePattern = gedeeldP; break;
  }
  showUntil = millis() + duurMs;
}

void showPattern(uint8_t pattern[]) {
  for (int i = 0; i < 8; i++) {
    mx.setRow(0, i, pattern[i]);
  }
}

/* ================== SETUP ================== */
void setup() {
  Serial.begin(9600);
  mx.begin();
  mx.clear();
  communicatie.sendPing();
  randomSeed(analogRead(0));
}

/* ================== LOOP ================== */
void loop() {

  /* ---- Communicatie ---- */
  if (communicatie.receive(msg, 2000)) {
    switch (msg.type) {

      case MSG_PING:
        communicatie.sendAck(msg.msgId);
        break;

      case MSG_GAME_DATA: {
        ontvangenOperators[0] = static_cast<Operators>((msg.data[0] << 8) | msg.data[1]);
        ontvangenOperators[1] = static_cast<Operators>((msg.data[2] << 8) | msg.data[3]);
        ontvangenOperators[2] = static_cast<Operators>((msg.data[4] << 8) | msg.data[5]);
        operatorBekend = true;
        break;
      }

      case MSG_GAME_WIN: {
        uint8_t playerNumber = msg.data[0];
        Serial.print("Player wint: ");
        Serial.println(playerNumber);
        break;
      }
    }
  }

  /* ---- RFID ---- */
  if (rfidtagSelected) {
  if (mfrc522.PICC_IsNewCardPresent()) {
    if (mfrc522.PICC_ReadCardSerial()) {
      uidSize = mfrc522.uid.size;
      for (byte i = 0; i < uidSize; i++) {
        uid[i] = mfrc522.uid.uidByte[i];
      }

      tagSelected = true;
      getal = rfidNaarGetal(uid, uidSize);

      if (!aGelezen) {
        a = getal;
        aGelezen = true;
      } else if (!bGelezen) {
        b = getal;
        bGelezen = true;
      }
    }
  }
}

  /* ---- Berekening ---- */
  if (aGelezen && bGelezen && operatorBekend) {

    Operators gekozenOperator = ontvangenOperators[random(0, 3)];

    toonOperator(gekozenOperator, 2000);
    delay(2000);

    int resultaat = berekenSom(a, b, gekozenOperator);
    Serial.print("Resultaat: ");
    Serial.println(resultaat);

    aGelezen = false;
    bGelezen = false;
    operatorBekend = false;
  }

  /* ---- LED refresh ---- */
  if (activePattern) {
    showPattern(activePattern);
    if (millis() > showUntil) {
      mx.clear();
      activePattern = nullptr;
    }
  }
}
