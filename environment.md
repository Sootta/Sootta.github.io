# ESP32でのGPIO出力制御：PlatformIOによる環境構築から実装まで

# ESP32とPlatformIOの環境構築

## 1. PlatformIOのインストール

PlatformIOはVS Code上で動作する強力な組み込み開発環境です。以下の手順でインストールします。

- VS Codeをインストールします（まだの場合）
- VS Codeの拡張機能タブから「PlatformIO IDE」を検索してインストールします
- インストール完了後、VS Codeを再起動します

## 2. ESP32プロジェクトの作成

PlatformIOを使ってESP32プロジェクトを作成します：

1. VS Code左側のPlatformIOアイコンをクリックします
2. 「Home」ボタンをクリックしてPlatformIOホームを開きます
3. 「New Project」をクリックします
4. プロジェクト名を入力します（例：「ESP32_GPIO_Test」）
5. ボードとして「Espressif ESP32 Dev Module」を選択します
6. フレームワークとして「Arduino」を選択します
7. 「Finish」をクリックしてプロジェクトを作成します

```cpp
; platformio.ini の例
[env:esp32dev]
platform = espressif32
board = esp32dev
framework = arduino
monitor_speed = 115200
```

# ESP32のGPIO出力制御

## 1. 基本的なLED点滅プログラム

まずは基本的なLED点滅プログラムを作成します。src/main.cppを開き、以下のコードを記述します：

```cpp
#include <Arduino.h>

// LEDピンの定義
const int LED_PIN = 2;  // ESP32内蔵LEDはGPIO2に接続されています

void setup() {
	// シリアル通信の初期化
	Serial.begin(115200);
	Serial.println("ESP32 GPIO制御テスト");
	
	// LEDピンを出力モードに設定
	pinMode(LED_PIN, OUTPUT);
}

void loop() {
	// LEDをONにする
	digitalWrite(LED_PIN, HIGH);
	Serial.println("LED ON");
	delay(1000);  // 1秒待機
	
	// LEDをOFFにする
	digitalWrite(LED_PIN, LOW);
	Serial.println("LED OFF");
	delay(1000);  // 1秒待機
}
```

## 2. 複数のGPIOピンを制御する例

次に、複数のLEDを制御する例を示します：

```cpp
#include <Arduino.h>

// LEDピンの定義
const int LED_PINS[] = {2, 4, 5, 12, 13};  // 使用するGPIOピン
const int NUM_LEDS = sizeof(LED_PINS) / sizeof(LED_PINS[0]);

void setup() {
	// シリアル通信の初期化
	Serial.begin(115200);
	Serial.println("ESP32 複数GPIO制御テスト");
  
	// 全てのLEDピンを出力モードに設定
	for (int i = 0; i < NUM_LEDS; i++) {
	    pinMode(LED_PINS[i], OUTPUT);
		digitalWrite(LED_PINS[i], LOW);  // 初期状態はOFF
	}
}

void loop() {
	// 順番に点灯
	for (int i = 0; i < NUM_LEDS; i++) {
	    digitalWrite(LED_PINS[i], HIGH);
	    Serial.printf("LED %d ON\\n", i);
	    delay(500);
		digitalWrite(LED_PINS[i], LOW);
	}
  
	// 全て点灯
	Serial.println("全LEDをON");
	for (int i = 0; i < NUM_LEDS; i++) {
	    digitalWrite(LED_PINS[i], HIGH);
	}
	delay(1000);
  
	// 全て消灯
	Serial.println("全LEDをOFF");
	for (int i = 0; i < NUM_LEDS; i++) {
	    digitalWrite(LED_PINS[i], LOW);
	}
	  delay(1000);
}
```

## 3. PWM出力によるLED明るさ制御

ESP32のLEDCチャンネルを使用して、PWM出力でLEDの明るさを制御する例：

```cpp
#include <Arduino.h>

// PWM設定
const int LED_PIN = 2;       // PWM出力用のピン
const int FREQ = 5000;       // PWM周波数
const int CHANNEL = 0;       // LEDCチャンネル
const int RESOLUTION = 8;    // 解像度（8ビット：0-255）

void setup() {
	// シリアル通信の初期化
	Serial.begin(115200);
	Serial.println("ESP32 PWM制御テスト");
	
	// PWM設定
	ledcSetup(CHANNEL, FREQ, RESOLUTION);
	ledcAttachPin(LED_PIN, CHANNEL);
}

void loop() {
	// 明るさを徐々に上げる
	Serial.println("明るさを上げています...");
	for (int dutyCycle = 0; dutyCycle <= 255; dutyCycle++) {
		ledcWrite(CHANNEL, dutyCycle);
		delay(15);
	}
	
	// 最大明るさで少し待機
	delay(1000);
	
	// 明るさを徐々に下げる
	Serial.println("明るさを下げています...");
	for (int dutyCycle = 255; dutyCycle >= 0; dutyCycle--) {
		ledcWrite(CHANNEL, dutyCycle);
		delay(15);
	}
	
	// 消灯状態で少し待機
	delay(1000);
}

```


# ビルドとアップロード

## プロジェクトのビルドとアップロード

コードを書いたら、ESP32にアップロードします：

1. ESP32をUSBケーブルでPCに接続します
2. VS Codeの下部ステータスバーにある「→」（アップロード）ボタンをクリックします
3. コードがコンパイルされ、ESP32にアップロードされます

## シリアルモニタの確認

プログラムの動作を確認するために、シリアルモニタを開きます：

1. VS Codeの下部ステータスバーにある「コンセント」アイコン（シリアルモニタ）をクリックします
2. シリアル出力を確認し、プログラムが正常に動作しているか確認します

# トラブルシューティング

- **ポートが認識されない場合**：ドライバのインストールが必要な場合があります。ESP32の種類によって、CP210xまたはCH340/CH341ドライバをインストールしてください。
- **アップロードに失敗する場合**：ESP32のBOOTボタンを押しながらアップロードを開始するか、RST（リセット）ボタンを押してみてください。
- **GPIOピンが機能しない場合**：ESP32のピン配置を確認してください。一部のピンは特別な機能を持っています（例：GPIO0は起動モード選択に使用）。

# まとめ

このガイドでは、PlatformIOを使用したESP32の環境構築からGPIO出力の制御方法までを解説しました。基本的なLED制御から、複数のGPIO制御、PWMによる明るさ制御まで、様々な例を示しました。ESP32は非常に柔軟なマイコンであり、これらの基本的な知識を応用することで、様々なプロジェクトに活用できます。