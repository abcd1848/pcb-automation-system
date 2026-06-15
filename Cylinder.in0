#include <TimerOne.h>
#include "DHT.h"

const int CONV_STEP = 9;
const int CONV_DIR  = 8;
const int CYL_RPWM  = 6;
const int CYL_LPWM  = 7;
const int BUZZER    = 10;

#define DHTPIN 2
#define DHTTYPE DHT11
DHT dht(DHTPIN, DHTTYPE);

int motorSpeed = 200;
volatile boolean motorState = LOW;
boolean isRunning = true;

boolean isWpfStopped = false;
boolean isStopped = false;

// cylState: 0=대기, 1=NG대기(6초), 2=전진, 3=전진후대기, 4=후진
int cylState = 0;
unsigned long cylStart = 0;
const int CYL_PWM      = 200;
const unsigned long CYL_FWD  = 5000;
const unsigned long CYL_WAIT = 300;
const unsigned long CYL_REV  = 5000;

boolean isDefectAlarm = false;
unsigned long defectAlarmStart = 0;

unsigned long lastSensorTime = 0;
const unsigned long SENSOR_INTERVAL = 5000;

String rxBuffer = "";

String calcChecksum(String payload) {
  byte xorVal = 0;
  for (int i = 0; i < payload.length(); i++) {
    xorVal ^= (byte)payload.charAt(i);
  }
  char hex[3];
  sprintf(hex, "%02X", xorVal);
  return String(hex);
}

void sendMsg(String command, String data = "") {
  String payload;
  if (data.length() > 0) {
    payload = command + "," + data;
  } else {
    payload = command;
  }
  String checksum = calcChecksum(payload);
  String msg = "<" + payload + "," + checksum + ">\n";
  Serial3.print(msg);
  Serial.print("[TX] "); Serial.print(msg);
}

void sendAck() {
  Serial3.print("<ACK>\n");
  Serial.println("[TX] <ACK>");
}

void sendNak() {
  Serial3.print("<NAK>\n");
  Serial.println("[TX] <NAK>");
}

void parseMessage(String raw) {
  if (raw == "ACK" || raw == "NAK") return;

  int lastComma = raw.lastIndexOf(',');
  if (lastComma < 0) return;

  String payload = raw.substring(0, lastComma);
  String checksum = raw.substring(lastComma + 1);
  checksum.trim();

  if (!calcChecksum(payload).equalsIgnoreCase(checksum)) {
    sendNak();
    Serial.println("[ERR] Checksum mismatch");
    return;
  }
  sendAck();

  int firstComma = payload.indexOf(',');
  String cmd, data;
  if (firstComma > 0) {
    cmd = payload.substring(0, firstComma);
    data = payload.substring(firstComma + 1);
  } else {
    cmd = payload;
    data = "";
  }

  Serial.print("[CMD] "); Serial.print(cmd);
  Serial.print(" [DATA] "); Serial.println(data);

  if (cmd == "START") {
    isWpfStopped = false;
    isRunning = true;
    Timer1.resume();
    noTone(BUZZER);
    Serial.println(">> Conveyor START");
  }
  else if (cmd == "STOP") {
    isRunning = false;
    Timer1.stop();
    Serial.println(">> Conveyor STOP");
  }
  else if (cmd == "ESTOP") {
    isWpfStopped = true;
    isRunning = false;
    Timer1.stop();
    analogWrite(CYL_RPWM, 0);
    analogWrite(CYL_LPWM, 0);
    cylState = 0;
    Serial.println(">> ESTOP from WPF");
  }
  else if (cmd == "RESUME") {
    isWpfStopped = false;
    isRunning = true;
    isStopped = false;
    Timer1.resume();
    noTone(BUZZER);
    Serial.println(">> RESUME");
  }
  else if (cmd == "RESULT") {
    if (data == "NG" && !isStopped) {
      startEject();
      Serial.println(">> EJECT (NG)");
    }
  }
  else if (cmd == "HEARTBEAT") {
  }
}

void conveyorISR() {
  if (!isStopped && isRunning) {
    motorState = !motorState;
    digitalWrite(CONV_STEP, motorState);
  } else {
    digitalWrite(CONV_STEP, LOW);
  }
}

void startEject() {
  if (cylState != 0) return;
  cylState = 1;
  cylStart = millis();
  isDefectAlarm = true;
  defectAlarmStart = millis();
  Serial.println(">> NG 감지, 6초 후 배출");
}

void updateCylinder() {
  if (cylState == 0) return;
  unsigned long elapsed = millis() - cylStart;

  if (cylState == 1 && elapsed >= 4000) {
    analogWrite(CYL_RPWM, 0);
    analogWrite(CYL_LPWM, CYL_PWM);
    cylState = 2;
    cylStart = millis();
    Serial.println(">> EJECT start");
  }
  else if (cylState == 2 && elapsed >= CYL_FWD) {
    analogWrite(CYL_RPWM, 0);
    analogWrite(CYL_LPWM, 0);
    cylState = 3;
    cylStart = millis();
  }
  else if (cylState == 3 && elapsed >= CYL_WAIT) {
    analogWrite(CYL_RPWM, CYL_PWM);
    analogWrite(CYL_LPWM, 0);
    cylState = 4;
    cylStart = millis();
  }
  else if (cylState == 4 && elapsed >= CYL_REV) {
    analogWrite(CYL_RPWM, 0);
    analogWrite(CYL_LPWM, 0);
    cylState = 0;
    Serial.println(">> EJECT complete");
  }
}

void setup() {
  pinMode(CONV_STEP, OUTPUT);
  pinMode(CONV_DIR, OUTPUT);
  pinMode(CYL_RPWM, OUTPUT);
  pinMode(CYL_LPWM, OUTPUT);
  pinMode(BUZZER, OUTPUT);

  digitalWrite(CONV_DIR, HIGH);
  analogWrite(CYL_RPWM, 0);
  analogWrite(CYL_LPWM, 0);

  Timer1.initialize(motorSpeed);
  Timer1.attachInterrupt(conveyorISR);

  Serial.begin(9600);
  Serial3.begin(9600);
  dht.begin();

  Serial.println("=== PCB INSPECTION SYSTEM ===");
  Serial.println("Ready.");
}

void loop() {
  isStopped = isWpfStopped;

  while (Serial3.available()) {
    char c = (char)Serial3.read();
    rxBuffer += c;

    int startIdx = rxBuffer.indexOf('<');
    int endIdx = rxBuffer.indexOf('>');
    if (startIdx >= 0 && endIdx > startIdx) {
      String raw = rxBuffer.substring(startIdx + 1, endIdx);
      rxBuffer = rxBuffer.substring(endIdx + 1);
      parseMessage(raw);
    }
    if (rxBuffer.length() > 200) rxBuffer = "";
  }

  if (!isStopped) {
    updateCylinder();
  }

  if (isStopped) {
    static unsigned long lastBuzz = 0;
    static bool buzzOn = false;
    if (millis() - lastBuzz >= 500) {
      lastBuzz = millis();
      buzzOn = !buzzOn;
      if (buzzOn) tone(BUZZER, 1000);
      else noTone(BUZZER);
    }
  }
  else if (isDefectAlarm) {
    if (millis() - defectAlarmStart < 1000) {
      tone(BUZZER, 1500);
    } else {
      noTone(BUZZER);
      isDefectAlarm = false;
    }
  }

  if (millis() - lastSensorTime >= SENSOR_INTERVAL) {
    float h = dht.readHumidity();
    float t = dht.readTemperature();
    if (!isnan(h) && !isnan(t)) {
      String data = String(t, 1) + "," + String(h, 1) + ",0,0,0";
      sendMsg("SENSOR", data);
    }
    lastSensorTime = millis();
  }
}
