# 機体下電装(Under)
## ファイル構造

```
.
└── 26th_Underside
    ├── 26th_Underside.ino : メインのarduinoスケッチ
    ├── BMP3xx.h/cpp : BMP390のコードをまとめたファイル
    ├── LED.h/cpp : XIAO RP2040内蔵のLEDを制御するためのファイル
    ├── NeoPixel.h/.cpp : XIAO RP2040内蔵のフルカラーLEDを制御するためのファイル
    ├── SD_Under.h/cpp : microSD書き込みを制御するファイル
    ├── UARTHelper_Under.h/cpp : UARTの処理をまとめたファイル
    ├── URM37.h/cpp : 超音波センサURM37の処理をまとめたファイル
    ├── Underside_config.h/cpp : 機体下基板のピン設定をまとめたファイル
    ├── calculate_altitude.h/cpp : 離陸判定や気圧高度計算の処理をまとめたファイル
    ├── parameters.h/cpp : センサ値など複数のマイコンでよく使う変数をまとめたファイル
    └── tsd20.h/cpp : LiDAR (TSD20)を制御するためのファイル
```
    
### WatchDogの設定
→エアデータBicoと全く同じ。

[L15-L17](https://github.com/torica-org/2026-fm-archives/blob/main/Software/Underside/26th_Underside/26th_Underside.ino#L15-L17)：WatchdogのライブラリインクルードとCore1生存確認用フラグの宣言
```c
// WatchDog用
#include "hardware/watchdog.h"
volatile bool core1_alive;  // core1の生存確認用フラグ
```

[L65-L66](https://github.com/torica-org/2026-fm-archives/blob/main/Software/Underside/26th_Underside/26th_Underside.ino#L65-L66)：Watchdogの設定・有効化
```c
watchdog_enable(3000, 1);  // WatchDogを有効化．
// 3000ms(=3s)経っても反応がない場合，システムが暴走したとみなして強制再起動
```

[L116-L121](https://github.com/torica-org/2026-fm-archives/blob/main/Software/Underside/26th_Underside/26th_Underside.ino#L116-L121)・[L145](https://github.com/torica-org/2026-fm-archives/blob/main/Software/Underside/26th_Underside/26th_Underside.ino#L145)：Watchdogに餌やり
- Core0側
```c
// WatchDogでCore1の生存確認
if (core1_alive == true) {
  watchdog_update();  // core1の生存を確認出来たらWatchDogTimerをアップデート

  core1_alive = false;  // core1の生存フラグをfalseに戻す
}
```
- Core1側
```c
core1_alive = true;
```



### ハードウェアタイマーの設定・100Hz周期で動作させる仕組み
これもエアデータBicoとまったく同じ．

[L19-L38](https://github.com/torica-org/2026-fm-archives/blob/main/Software/Underside/26th_Underside/26th_Underside.ino#L19-L38)：ハードウェアタイマーの宣言
```c
// ハードウェアタイマー設定
#include "pico/stdlib.h"
struct repeating_timer core0_timer;
struct repeating_timer core1_timer;

// 100Hz実行用フラグ
volatile bool core0_timer_triggered = false;
（中略）
bool core1_timer_callback(struct repeating_timer *t) {
  core1_timer_triggered = true;
  return true;
}
```

[L62-L63](https://github.com/torica-org/2026-fm-archives/blob/main/Software/Underside/26th_Underside/26th_Underside.ino#L62-L63)・[L73-L76](https://github.com/torica-org/2026-fm-archives/blob/main/Software/Underside/26th_Underside/26th_Underside.ino#L73-L76)：ハードウェアタイマーの初期化・起動
```c
// ハードウェアタイマー起動
add_repeating_timer_ms(-10, core0_timer_callback, NULL, &core0_timer);
```

```c
void setup1() {
  // ハードウェアタイマーの設定はコアごとに
  add_repeating_timer_ms(-10, core1_timer_callback, NULL, &core1_timer);
}
```

loop関数部分（実際に100Hz周期で処理を実行させる部分）
[L79-L81](https://github.com/torica-org/2026-fm-archives/blob/main/Software/Underside/26th_Underside/26th_Underside.ino#L62-L63)・[L73-L76](https://github.com/torica-org/2026-fm-archives/blob/main/Software/Underside/26th_Underside/26th_Underside.ino#L79-L81)
```c
void loop() {
  if (core0_timer_triggered == true /* 100Hz用フラグが立っていたら */) {
  core0_timer_triggered = false;  // フラグを戻す
  （ここに処理を書く）
  }
}
```
loop1()も同様．


### setup/setup1関数
I2CやSPI・センサー・Watchdog・ハードウェアタイマーの初期化を行う．

### loop関数
loop (Core0)ではUART送受信，BMP390による気温気圧測定，NeoPixelの制御を行う．センサとNeoPixelの制御については別記事に書いた．

### loop1関数
超音波高度・LiDAR高度の読み取りとSDの書き込みを行う．特に超音波に関しては取り扱い要注意。
