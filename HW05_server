#include <BLEDevice.h>
#include <BLEServer.h>
#include <BLEUtils.h>
#include <BLE2902.h>

#define BUTTON_PIN 4 // GPIO 4번 핀 사용

BLECharacteristic *pCharacteristic;
bool deviceConnected = false;
bool oldButtonState = HIGH;

#define SERVICE_UUID        "b9e2a522-08cc-4ff0-9ff4-569377be33d8"
#define CHARACTERISTIC_UUID "2da79697-0701-4a05-bb6a-433c7424a15a"

class MyServerCallbacks : public BLEServerCallbacks {
  void onConnect(BLEServer* pServer) {
    deviceConnected = true;
    Serial.println("✅ Client connected");
  }

  void onDisconnect(BLEServer* pServer) {
    deviceConnected = false;
    Serial.println("⚠️ Client disconnected. Restarting advertising…");
    BLEDevice::getAdvertising()->start();  // 🔁 advertising 재시작
  }
};

void setup() {
  Serial.begin(115200);
  pinMode(BUTTON_PIN, INPUT_PULLUP);

  // BLE 초기화
  BLEDevice::init("ESP32-Button-Server");

  // 서버 생성 및 콜백 연결
  BLEServer *pServer = BLEDevice::createServer();
  pServer->setCallbacks(new MyServerCallbacks());

  // 서비스 생성
  BLEService *pService = pServer->createService(SERVICE_UUID);

  // 특성 생성
  pCharacteristic = pService->createCharacteristic(
                      CHARACTERISTIC_UUID,
                      BLECharacteristic::PROPERTY_READ |
                      BLECharacteristic::PROPERTY_NOTIFY
                    );

  // 클라이언트에서 notification 수신 가능하도록 디스크립터 추가
  pCharacteristic->addDescriptor(new BLE2902());

  // 서비스 시작
  pService->start();

  // 광고 시작
  BLEAdvertising *pAdvertising = BLEDevice::getAdvertising();
  pAdvertising->start();

  Serial.println("🚀 BLE Server ready and advertising…");
}

void loop() {
  bool buttonState = digitalRead(BUTTON_PIN);
  if (deviceConnected && buttonState != oldButtonState) {
    oldButtonState = buttonState;
    String message = buttonState == LOW ? "Pressed" : "Released";

    // 특성 값 설정 및 notification 전송
    pCharacteristic->setValue(message.c_str());
    pCharacteristic->notify();
    Serial.println("📤 Sent: " + message);
  }
  delay(100);
}
