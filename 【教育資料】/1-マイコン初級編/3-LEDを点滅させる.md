# 3 - LEDを点滅させる

マイコンを使うことで，電気が流れる／流れないを制御することができます．

ここでは，LEDのON/OFFを切り替えて，LEDを点滅させてみましょう！

## 必要なもの
- Raspberry Pi Pico
- ブレッドボード
- マイコンとPCを接続するためのUSBケーブル
- Arduino IDEがインストールされたPC
  - ボードマネージャ`Raspberry Pi Pico/RP2040/RP2350`がインストールされていること
 
## やり方
### 1. ブレッドボード上で配線する
以下の図のように配線します．**奥まで差し込むこと！**
<img width="800" height="450" alt="image" src="https://github.com/user-attachments/assets/c5f5a039-6c4e-4579-a1d2-9b18e14119fb" />



### 2. プログラムを準備する
Arduino IDEに以下のプログラムをコピーしましょう．

```cpp
const int led = 0;

void setup() {
  pinMode(led, OUTPUT);
}

void loop() {
  digitalWrite(led, HIGH);
  delay(1000);
  digitalWrite(led, LOW);
  delay(1000);
}
```

ここに出てきた`pinMode()`や`digitalWrite()`はArduino IDEでだけ使える特別な関数です．
[このページ](https://www.musashinodenpa.com/arduino/ref/)で確認してみてください．

### 3. プログラムをマイコンに書き込む
PCとマイコンをUSBケーブルでつなぎます．
Arduino IDEでボードの種類とCOMポートを選択します．

### 4. 動作確認
1秒ごとにON/OFFが切り替わる，つまり2秒ごとに点滅していれば完成です！

## 練習問題
### 1. LEDが0.2秒点灯，0.8秒消灯を繰り返すプログラムを作成し，動作を確認しなさい．
[ヒント] 
- `delay()`関数はミリ秒単位で設定します．

<details>
<summary>[解答]</summary>
<!--この下に1行空行を挟む-->
  
```cpp
const int led = 0;

void setup() {
  pinMode(led, OUTPUT);
}

void loop() {
  digitalWrite(led, HIGH);
  delay(200);
  digitalWrite(led, LOW);
  delay(800);
}
```
</details>

### 2. 「練習問題 1」の解答を用い，LEDの配線場所を`GPIO5`に変更したうえで動作を確認しなさい．
[ヒント]
- 変数`led`には「ピン番号（GPIO??）」が格納されています

<details>
<summary>[解答]</summary>
<!--この下に1行空行を挟む-->
  
```cpp
const int led = 5;

void setup() {
  pinMode(led, OUTPUT);
}

void loop() {
  digitalWrite(led, HIGH);
  delay(200);
  digitalWrite(led, LOW);
  delay(800);
}
```
</details>
