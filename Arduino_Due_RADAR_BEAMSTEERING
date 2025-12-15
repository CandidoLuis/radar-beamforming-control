//---------- INCLUDES E OBJETOS
#include <LiquidCrystal.h>
#include <Adafruit_NeoPixel.h>
#include <DueFlashStorage.h>
#include <math.h>
#include <string.h>

LiquidCrystal lcd(13, 12, 11, 10, 9, 8);
Adafruit_NeoPixel strip(60, 2, NEO_GRB + NEO_KHZ800);
DueFlashStorage dueFlashStorage;

String degree = String((char)223);

const int buttonPin = A4;    // Botão de menu

//---------- MENU / DEBOUNCE
volatile unsigned long lastDebounceTime = 0;
unsigned long debounceDelay = 200;
volatile int menu = 0;       // 0..5

//---------- VARIÁVEIS GERAIS
int switchState = 0;
String str;
float phi = 0, pot = 0, minpot = 0, maxpot = -100;
int dif = 0, val_1 = 0, val_2 = 0, val_3 = 0, val_1m = 0, val_2m = 0, val_3m = 0, teta = 0, f1 = 0, f2 = 0, f3 = 0, i = 0;
int bin[6] = {0, 0, 0, 0, 0, 0};
int pesos[6] = {5, 11, 22, 45, 90, 180};

int pot1 = 0, pot2 = 0, pot3 = 0;

int pos;
int green;
int red;
const float arr[] = {1.0, 0.7, 0.2};

// limiar ≈ 3 V (3/3.3 * 1023 ≈ 930)
const int POT_HIGH_THR = 930;

//---------- TX
int offset1_Tx = 170, offset2_Tx = 197, offset3_Tx = 225;
int val_1_TxPhase = 0;
int val_2_TxPhase = 0;
int val_3_TxPhase = 0;

struct TxOffsets {
  int o1;
  int o2;
  int o3;
};

TxOffsets txOffsets = {170, 197, 225};
const int FLASH_ADDR_TX_OFFSETS = 0;

int ps3_Tx[6] = {22, 24, 26, 27, 25, 23};
int ps2_Tx[6] = {28, 30, 32, 33, 31, 29};
int ps1_Tx[6] = {34, 36, 40, 41, 39, 35};

//---------- RX
int offset1_Rx = 191, offset2_Rx = 230, offset3_Rx = 230;

int ps3_Rx[6] = {42, 44, 46, 47, 45, 43};
int ps2_Rx[6] = {48, 50, 52, 53, 51, 49};
int ps1_Rx[6] = {16, 17, 18, 21, 20, 19};

//---------- LONG PRESS (para gravar offset)
const unsigned long LONG_PRESS_TIME = 5000;  // 5 s
bool buttonPrev = false;
unsigned long buttonPressTime = 0;
bool longPressHandled = false;

//---------- MODO AUTÓNOMO (varrimento automático)
bool autoScanActive = false;
float tetaAuto = 90.0;
int dirScan = 1;              // +1 / -1

//---------- MENSAGEM OFFSET SALVO NO LCD
bool showOffsetMsg = false;
unsigned long offsetMsgTime = 0;
int lastOffsetTx = 0;  // 1, 2 ou 3

//---------------------- FLASH TX OFFSETS ---------------------------//
void saveTxOffsetsToFlash() {
  txOffsets.o1 = offset1_Tx;
  txOffsets.o2 = offset2_Tx;
  txOffsets.o3 = offset3_Tx;

  Serial.print("[FLASH] Gravando offsets TX: ");
  Serial.print(txOffsets.o1); Serial.print(", ");
  Serial.print(txOffsets.o2); Serial.print(", ");
  Serial.println(txOffsets.o3);

  dueFlashStorage.write(
    FLASH_ADDR_TX_OFFSETS,
    (byte*)&txOffsets,
    sizeof(TxOffsets)
  );

  // leitura imediata para confirmar escrita
  byte* flashPtrCheck = dueFlashStorage.readAddress(FLASH_ADDR_TX_OFFSETS);
  TxOffsets check;
  memcpy(&check, flashPtrCheck, sizeof(TxOffsets));

  Serial.print("[FLASH] Após escrita, lido da flash: ");
  Serial.print(check.o1); Serial.print(", ");
  Serial.print(check.o2); Serial.print(", ");
  Serial.println(check.o3);
}

void loadTxOffsetsFromFlash() {
  // lê ponteiro para a flash
  byte* flashPtr = dueFlashStorage.readAddress(FLASH_ADDR_TX_OFFSETS);

  TxOffsets tmp;
  // copia os bytes da flash para RAM
  memcpy(&tmp, flashPtr, sizeof(TxOffsets));

  Serial.print("[FLASH] Conteúdo bruto na leitura: ");
  Serial.print(tmp.o1); Serial.print(", ");
  Serial.print(tmp.o2); Serial.print(", ");
  Serial.println(tmp.o3);

  // valida offsets (primeira vez a flash pode ter lixo)
  if (tmp.o1 < 0 || tmp.o1 >= 360 ||
      tmp.o2 < 0 || tmp.o2 >= 360 ||
      tmp.o3 < 0 || tmp.o3 >= 360) {
    Serial.println("[FLASH] Valores inválidos na flash. A aplicar defaults e gravar...");
    offset1_Tx = 170;
    offset2_Tx = 197;
    offset3_Tx = 225;
    saveTxOffsetsToFlash();
  } else {
    offset1_Tx = tmp.o1;
    offset2_Tx = tmp.o2;
    offset3_Tx = tmp.o3;
  }

  Serial.print("[FLASH] Offsets TX em uso: ");
  Serial.print(offset1_Tx); Serial.print(", ");
  Serial.print(offset2_Tx); Serial.print(", ");
  Serial.println(offset3_Tx);
}

//--------------------- LONG PRESS / ATUALIZAR OFFSETS --------------//
void handleLongPressOffsets() {
  if (switchState != 0) return;   // só em modo manual

  const int ZERO_THR = 10;   // limiar ~0V

  bool p1_zero = (pot1 < ZERO_THR);
  bool p2_zero = (pot2 < ZERO_THR);
  bool p3_zero = (pot3 < ZERO_THR);

  int numNonZero = (!p1_zero) + (!p2_zero) + (!p3_zero);
  if (numNonZero != 1) {
    Serial.println("[OFFSET] Long press ignorado: deve haver exatamente um pot ativo.");
    return;
  }

  if (!p1_zero) {
    offset1_Tx = val_1_TxPhase;
    lastOffsetTx = 1;
  } else if (!p2_zero) {
    offset2_Tx = val_2_TxPhase;
    lastOffsetTx = 2;
  } else if (!p3_zero) {
    offset3_Tx = val_3_TxPhase;
    lastOffsetTx = 3;
  }

  saveTxOffsetsToFlash();

  showOffsetMsg = true;
  offsetMsgTime = millis();

  Serial.print("[OFFSET] Offsets Tx actualizados: ");
  Serial.print(offset1_Tx); Serial.print(" ");
  Serial.print(offset2_Tx); Serial.print(" ");
  Serial.println(offset3_Tx);
}

//----------------------------- SETUP ------------------------------//
void setup() {

  Serial.begin(9600);
  while (!Serial) {
    ; // espera porta série (útil no Due)
  }

  Serial.println("===== ARRANQUE DO SISTEMA =====");

  // Opcional mas recomendado no Due, se queres 0–1023:
  // analogReadResolution(10);

  pinMode(A0, INPUT);
  pinMode(A1, INPUT);
  pinMode(A2, INPUT);
  pinMode(A5, INPUT);
  pinMode(A3, INPUT);

  for (i = 0; i < 6; i++) {
    pinMode(ps1_Tx[i], OUTPUT);
    pinMode(ps2_Tx[i], OUTPUT);
    pinMode(ps3_Tx[i], OUTPUT);
    pinMode(ps1_Rx[i], OUTPUT);
    pinMode(ps2_Rx[i], OUTPUT);
    pinMode(ps3_Rx[i], OUTPUT);
  }

  strip.begin();
  strip.show();
  strip.setBrightness(0);

  pinMode(buttonPin, INPUT);  // se tiveres pull-up no botão, pode ser INPUT_PULLUP
  attachInterrupt(digitalPinToInterrupt(buttonPin), ISR_button, RISING);

  lcd.begin(16, 2);

  // lê offsets da flash no arranque
  loadTxOffsetsFromFlash();
}

//------------------------------ LOOP ------------------------------//
void loop() {

  pot1 = analogRead(A0);
  pot2 = analogRead(A1);
  pot3 = analogRead(A2);

  int det = analogRead(A5);

  switchState = digitalRead(A3);

  // Long press para offsets
  int buttonState = digitalRead(buttonPin);

  if (buttonState == HIGH && !buttonPrev) {
    buttonPressTime = millis();
    longPressHandled = false;
  }

  if (buttonState == HIGH && !longPressHandled &&
      (millis() - buttonPressTime >= LONG_PRESS_TIME)) {
    handleLongPressOffsets();
    longPressHandled = true;
  }

  if (buttonState == LOW && buttonPrev) {
    longPressHandled = false;
  }

  buttonPrev = (buttonState == HIGH);

  // Gestor do modo autónomo
  if (switchState == 1 && menu == 5) {
    if (!autoScanActive &&
        pot1 >= POT_HIGH_THR &&
        pot2 >= POT_HIGH_THR &&
        pot3 >= POT_HIGH_THR) {
      autoScanActive = true;
      tetaAuto = 90.0;
      dirScan = 1;
      Serial.println("[AUTOSCAN] AutoScan ativado.");
    }
  } else {
    if (autoScanActive) {
      Serial.println("[AUTOSCAN] AutoScan desativado.");
    }
    autoScanActive = false;
  }

  // TX: cálculo val_1..3
  if (switchState == 0) { // manual
    val_1 = min(63, (int)(63 * pot1 / 1004.0));
    val_2 = min(63, (int)(63 * pot2 / 1004.0));
    val_3 = min(63, (int)(63 * pot3 / 1004.0));
  } 
  else if (autoScanActive) { // autoScan
    val_1 = (int)tetaAuto;
    phi = 3.1415 * (val_1 - 90.0) / 180.0;
    dif =  360 * 2.5 * sin(phi) / 5.1;
    val_1 = 63 * ((360 + dif) % 360) / 360;
    val_2 = 63 * ((360 + dif * 2) % 360) / 360;
    val_3 = 63 * ((360 + dif * 3) % 360) / 360;
  }
  else { // Beamforming (controle por pot1)
    val_1 = 180 * pot1 / 1023;
    phi = 3.1415 * (val_1 - 90.0) / 180.0;
    dif =  360 * 2.5 * sin(phi) / 5.1;
    val_1 = 63 * ((360 + dif) % 360) / 360;
    val_2 = 63 * ((360 + dif * 2) % 360) / 360;
    val_3 = 63 * ((360 + dif * 3) % 360) / 360;
  }

  // PS1
  int2bin(val_1, bin);
  val_1m = convbin2deg(bin);

  val_1 = (360 + val_1m + offset1_Tx) % 360;
  val_1_TxPhase = val_1;
  convdeg2bin(val_1, bin);
  send2ps(bin, ps1_Tx);

  val_1 = (360 + val_1m + offset1_Rx) % 360;
  convdeg2bin(val_1, bin);
  send2ps(bin, ps1_Rx);

  // PS2
  int2bin(val_2, bin);
  val_2m = convbin2deg(bin);

  val_2 = (360 + val_2m + offset2_Tx) % 360;
  val_2_TxPhase = val_2;
  convdeg2bin(val_2, bin);
  send2ps(bin, ps2_Tx);

  val_2 = (360 + val_2m + offset2_Rx) % 360;
  convdeg2bin(val_2, bin);
  send2ps(bin, ps2_Rx);

  // PS3
  int2bin(val_3, bin);
  val_3m = convbin2deg(bin);

  val_3 = (360 + val_3m + offset3_Tx) % 360;
  val_3_TxPhase = val_3;
  convdeg2bin(val_3, bin);
  send2ps(bin, ps3_Tx);

  val_3 = (360 + val_3m + offset3_Rx) % 360;
  convdeg2bin(val_3, bin);
  send2ps(bin, ps3_Rx);

  // RX / BEAMFORMING (ângulo teta)
  if (switchState == 1) {
    strip.setBrightness(255);

    if (autoScanActive) {
      float stepMin = 0.1;
      float stepMax = 2.0;
      float step = stepMin + (stepMax - stepMin) * (pot2 / 1023.0);

      float spanMin = 20.0;
      float spanMax = 160.0;
      float span = spanMin + (spanMax - spanMin) * (pot3 / 1023.0);

      float tetaMin = 90.0 - span / 2.0;
      float tetaMax = 90.0 + span / 2.0;

      tetaAuto += dirScan * step;

      if (tetaAuto > tetaMax) {
        tetaAuto = tetaMax;
        dirScan = -1;
      } else if (tetaAuto < tetaMin) {
        tetaAuto = tetaMin;
        dirScan = 1;
      }

      teta = (int)tetaAuto;
    } else {
      teta = 180 * pot1 / 1023;
    }

  } else {
    strip.setBrightness(0);
  }

  // Potência recebida
  pot = 3.3 * det / 1023;
  pot = (-40.74 * pot) + 26.48;

  if (pot < minpot) {
    minpot = pot;
  } else if (pot > maxpot) {
    maxpot =  pot;
  }

  f1 = 70;
  f2 = -40;
  f3 = 30;

  // LED strip
  pos = 60 * (teta - 50) / 80;
  if (maxpot > minpot) {
    green = (int)(255 * pow((pot - minpot) / (maxpot - minpot), 6));
  } else {
    green = 0;
  }
  red = 255 - green;
  beamform();

  // MATLAB string
  str = String(val_1m) + " " + String(val_2m) + " " + String(val_3m) + " " +
        String(100 * red / 255) + " " + String(teta) + " " +
        String(f1) + " " + String(f2) + " " + String(f3) + " " +
        String(switchState);

  screen();           // assumo que está noutro ficheiro .ino
  Serial.println(str);

  delay(10);
}

//----------------------- FUNÇÕES AUXILIARES ------------------------//
void int2bin(int val, int array[]) {
  for (int i = 0; i < 6; i++) {
    array[i] = (val >> i) & 1;
  }
}

void send2ps(int bin[], int ps[]) {
  for (int i = 0; i < 6; i++) {
    digitalWrite(ps[i], bin[i]);
  }
}

int convbin2deg(int bin[]) {
  return ((bin[0] * 5) + (bin[1] * 11) + (bin[2] * 22) +
          (bin[3] * 45) + (bin[4] * 90) + (bin[5] * 180));
}

void convdeg2bin(int value, int bin[]) {
  for (int i = 0; i < 6; i++) {
    if (value >= pesos[5 - i]) {
      bin[5 - i] = 1;
      value = value - pesos[5 - i];
    } else {
      bin[5 - i] = 0;
    }
  }
}

void ISR_button() {
  if ((millis() - lastDebounceTime) > debounceDelay) {
    lastDebounceTime = millis();
    menu = (menu + 1) % 6;   // 0..5
  }
}

void beamform() {
  for (int i = 0; i < 60; i++) {
    if (i >= pos - 3 && i <= pos + 3) {
      int idx = abs(pos - i);
      if (idx > 2) idx = 2;
      uint8_t r = (uint8_t)(red   * arr[idx]);
      uint8_t g = (uint8_t)(green * arr[idx]);
      strip.setPixelColor(i, strip.Color(r, g, 0));
    } else {
      strip.setPixelColor(i, strip.Color(0, 0, 0));
    }
  }
  strip.show();
}
