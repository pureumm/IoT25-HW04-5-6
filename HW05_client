#include <BLEDevice.h>
#include <BLEUtils.h>
#include <BLEScan.h>
#include <BLEAdvertisedDevice.h>

#define SERVICE_UUID        "b9e2a522-08cc-4ff0-9ff4-569377be33d8"
#define CHARACTERISTIC_UUID "2da79697-0701-4a05-bb6a-433c7424a15a"
#define TARGET_NAME         "ESP32-Button-Server"  // 디바이스 이름 기준

BLEClient* pClient;
BLERemoteCharacteristic* pRemoteCharacteristic;
bool doConnect = false;
BLEAdvertisedDevice* myDevice;

class MyAdvertisedDeviceCallbacks: public BLEAdvertisedDeviceCallbacks {
  void onResult(BLEAdvertisedDevice advertisedDevice) {
    // 조건 1: 서비스 UUID가 일치하거나
    bool uuidMatch = advertisedDevice.haveServiceUUID() && advertisedDevice.getServiceUUID().toString() == SERVICE_UUID;

    // 조건 2: 디바이스 이름이 일치
    bool nameMatch = advertisedDevice.haveName() && advertisedDevice.getName() == TARGET_NAME;

    if (uuidMatch || nameMatch) {
      Serial.print("✅ Found target device: ");
      Serial.println(advertisedDevice.toString().c_str());

      myDevice = new BLEAdvertisedDevice(advertisedDevice);
      doConnect = true;
      BLEDevice::getScan()->stop();  // 스캔 중지
    }
  }
};

class MyClientCallback : public BLEClientCallbacks {
  void onConnect(BLEClient* pClient) {
    Serial.println("🔗 Connected to server.");
  }
  void onDisconnect(BLEClient* pClient) {
    Serial.println("❌ Disconnected from server.");
  }
};

bool connectToServer() {
  pClient = BLEDevice::createClient();
  pClient->setClientCallbacks(new MyClientCallback());

  if (!pClient->connect(myDevice)) {
    Serial.println("⚠️ Failed to connect.");
    return false;
  }

  BLERemoteService* pRemoteService = pClient->getService(SERVICE_UUID);
  if (pRemoteService == nullptr) {
    Serial.println("❌ Service not found.");
    return false;
  }

  pRemoteCharacteristic = pRemoteService->getCharacteristic(CHARACTERISTIC_UUID);
  if (pRemoteCharacteristic == nullptr) {
    Serial.println("❌ Characteristic not found.");
    return false;
  }

  if (pRemoteCharacteristic->canNotify()) {
    pRemoteCharacteristic->registerForNotify([](BLERemoteCharacteristic* pBLERemoteCharacteristic,
                                                 uint8_t* pData, size_t length, bool isNotify) {
      String value((char*)pData, length);
      Serial.print("📥 Received from Server: ");
      Serial.println(value);
    });
  }

  return true;
}

void setup() {
  Serial.begin(115200);
  BLEDevice::init("ESP32-Client");

  BLEScan* pBLEScan = BLEDevice::getScan();
  pBLEScan->setAdvertisedDeviceCallbacks(new MyAdvertisedDeviceCallbacks());
  pBLEScan->setInterval(100);
  pBLEScan->setWindow(99);
  pBLEScan->setActiveScan(true);
}

void loop() {
  if (!doConnect) {
    Serial.println("🔍 Scanning for BLE server...");
    BLEDevice::getScan()->start(5, false);  // 5초 단위 반복 스캔
  }

  if (doConnect) {
    if (connectToServer()) {
      Serial.println("✅ Connected and ready to receive notifications.");
    } else {
      Serial.println("⚠️ Connection failed.");
    }
    doConnect = false;
  }

  delay(1000);
}
