# STM32マイコンのタイマー機能解説

## 1. PWM生成

PWM（Pulse Width Modulation）は、デジタル信号のデューティ比（ONとOFF時間の比率）を変化させることで、アナログ的な制御を可能にする技術です。STM32マイコンではタイマーを使用してPWM信号を生成します。

### PWM生成の基本設定手順

- **STM32CubeMXでの設定:**
- ①適切なピンを選択し、TIMのPWMモードを設定
- ②Clock ConfigurationでHCLK（システムクロック）を設定
- ③Prescaler（分周比）とCounter Period（カウンタ周期）を設定して周波数調整
- ④タイマーモードをPWM Generation CHxに設定

### PWM周波数の計算方法

PWM周波数は以下の式で計算されます：

```
PWM周波数 = HCLK / ((Prescaler+1) × (Counter Period+1))
```

例えば、HCLK=42MHz、Prescaler=840、Counter Period=1000の場合：

```
PWM周波数 = 42MHz / ((840+1) × (1000+1)) ≈ 50Hz
```

### PWM出力のコード例

```c
/* PWM開始 */
HAL_TIM_PWM_Start(&htim3, TIM_CHANNEL_1);

/* PWMのデューティ比設定（0〜Counter Period） */
__HAL_TIM_SET_COMPARE(&htim3, TIM_CHANNEL_1, 500); // 50%のデューティ比（Counter Periodが1000の場合）
```

## 2. エンコーダー読み取り

エンコーダーは回転や位置の変化を電気信号に変換するセンサーです。STM32のタイマーをエンコーダーモードで使用することで、簡単に回転量や方向を検出できます。

### エンコーダー読み取りの設定手順

- **STM32CubeMXでの設定:**
- ①エンコーダーのA相とB相を接続するピンを選択（例：PA6, PA7）
- ②タイマー（例：TIM3）をEncoder Modeに設定
- ③「Combined Channels」でモードを「Encoder Mode TI1 and TI2」に設定
- ④必要に応じてフィルタ設定やカウント方向を調整

### エンコーダー読み取りコード例

```c
/* エンコーダー読み取り開始 */
HAL_TIM_Encoder_Start(&htim3, TIM_CHANNEL_ALL);

/* カウント値の読み取り */
uint32_t encoder_count = __HAL_TIM_GET_COUNTER(&htim3);
// または
uint32_t encoder_count = TIM3->CNT;
```

エンコーダーモードでは、A相とB相の位相差によって回転方向を判別し、自動的にカウンタ値が増減します。

## 3. タイマー割り込み

タイマー割り込みは、一定時間ごとに処理を実行したい場合に便利です。STM32のタイマーは、カウンタがオーバーフローしたときに割り込みを発生させることができます。

### タイマー割り込みの設定手順

- **STM32CubeMXでの設定:**
- ①タイマー（例：TIM7）を選択し、Activatedにチェック
- ②Prescalerと Counter Periodを設定して割り込み周期を調整
- ③NVIC Settingsタブで割り込みを有効化

例えば10msの割り込み周期を作るには（HCLK=72MHz）：

```
必要カウント数 = 10ms ÷ (1/72MHz) = 720,000
Prescaler = 64を選ぶと
Counter Period = 720,000 ÷ 64 - 1 = 11,249
```

### タイマー割り込みのコード例

```c
/* タイマー割り込み開始 */
HAL_TIM_Base_Start_IT(&htim7);

/* 割り込みハンドラの実装 */
void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
    if(htim == &htim7) {
        /* タイマー7の割り込み処理 */
        HAL_GPIO_TogglePin(LD2_GPIO_Port, LD2_Pin); // LEDをトグル
    }
}
```

## 実装例：サーボモーター制御

サーボモーターは一般的に50Hzの周波数で、パルス幅0.5ms〜2.4msの範囲で角度制御します。

```c
/* サーボモーターの角度を設定する関数 */
void setServoAngle(TIM_HandleTypeDef *htim, uint32_t channel, float angle)
{
    /* 角度を0〜180度の範囲に制限 */
    if(angle < 0) angle = 0;
    if(angle > 180) angle = 180;
    
    /* 角度をパルス幅（25〜125）に変換 */
    uint32_t pulse = 25 + (angle / 180.0) * 100;
    
    /* PWMのパルス幅を設定 */
    __HAL_TIM_SET_COMPARE(htim, channel, pulse);
}
```

## 実装の際の注意点

- PWM周波数とシステムクロック設定の関係を正しく理解する
- Counter Periodの値は0からカウントするため、設定値は「必要カウント数-1」となる
- エンコーダモードでは通常4逓倍（4倍の分解能）で使用される
- タイマー割り込みでは、処理時間が割り込み周期を超えないように注意する
- デバッグ中は一時停止するとタイマーも止まるため、動作確認に注意が必要