# TEST
THIS IS TEST DOCUMENT

# Wio_ExtFlashLoadスケッチをビルドする(platformio)
2022/2/3  


## 概要
「[Wio Ext Flash Load](https://macsbug.wordpress.com/2020/06/09/wio-ext-flash-loader/)」、  
「[External Flash Loader library for Wio Terminal](https://github.com/ciniml/ExtFlashLoader)」  
にあるWio_ExtFlashLoad(WriteSampleMenu.ino)スケッチをplatformioでビルドする。  
arduinoのサンプル・スケッチとして提供されているものだが、plaformioビルド環境でビルドする。  
そのplatformio.iniを提供する。  
Wio_ExtFlashLoadは、アプリを起動するメニューになっており、  
SDカードにビルドした.binを起き、それを選択して起動する。  


## WriteSampleMenu.ino
以下の手順を実行する：

```
# platformioの環境に入る

# プロジェクト・ディレクトリを作成する
mkdir Wio_ExtFlashLoad
cd Wio_ExtFlashLoad

mkdir src
cd src
wget https://raw.githubusercontent.com/ciniml/ExtFlashLoader/master/examples/WriteSampleMenu/WriteSampleMenu.ino
wget https://raw.githubusercontent.com/ciniml/ExtFlashLoader/master/examples/WriteSampleMenu/menu_data.h


cd ..

gedit platformio.ini
# 後節にあるplatformio.iniの内容を作成する

# ビルド＆書き込み
pio run -t upload

```

以上で、メニューが表示されるようになる。

## platformio.ini

wio-terminal用：
```
[env:seeed_wio_terminal]
platform = atmelsam
board = seeed_wio_terminal
framework = arduino
build_flags = -DWIO_TERMINAL
upload_protocol = sam-ba
monitor_speed = 115200
lib_ldf_mode = deep+

lib_deps = 
    https://github.com/Seeed-Studio/Seeed_Arduino_mbedtls/archive/dev.zip
    https://github.com/Seeed-Studio/Seeed_Arduino_rpcUnified/archive/master.zip
    https://github.com/Seeed-Studio/Seeed_Arduino_rpcBLE/archive/master.zip
    https://github.com/Seeed-Studio/Seeed_Arduino_rpcWiFi/archive/master.zip
    https://github.com/Seeed-Studio/Seeed_Arduino_FreeRTOS/archive/master.zip
    https://github.com/Seeed-Studio/Seeed_Arduino_FS/archive/master.zip
    https://github.com/Seeed-Studio/Seeed_Arduino_SFUD/archive/master.zip
    #
    https://github.com/Seeed-Studio/Seeed_Arduino_LCD/archive/master.zip
    #
    arduino-libraries/NTPClient@^3.1.0
    #
    ciniml/ExtFlashLoader@^0.1.2


```

## アプリ用.binを作成する
アプリ用SDの詳細は「[External Flash Loader library for Wio Terminal](https://github.com/ciniml/ExtFlashLoader)」に  
記述されているので、その中に入れる.binの作成方法について説明する。  

「pio run」でビルドすると  
以下に.binができる：
```
.pio/build/seeed_wio_terminal/firmware.bin
```
これをfirmware.binをapp.pinにリネームしてSDにコピーする。

これだけでメニューから起動できる。  
ただ、このままだとアプリ起動後、  
メニューに戻ることができないので  
setup()にボタンAを押してリセットしたときに  
メニューに戻るコードを埋め込む。  

TFT_eSPI対応アプリの場合、  
「[External Flash Loader library for Wio Terminal](https://github.com/ciniml/ExtFlashLoader)」の    
「アプリケーションのメニューアプリ対応」を参照のこと。  

LGFX対応アプリの場合は以下のコードを埋め込む：
```c++

以下のヘッダーを追加する：
///////////////////////////////////////////
// application menu header
#include <cstdint>
#include <ExtFlashLoader.h>
///////////////////////////////////////////

変更前：
void setup() { 

  lcd.init();
  
-------------------
→
変更後： 
void setup() { 
//// goto application menu /////////////// 
 lcd.init();
 pinMode(WIO_KEY_A, INPUT_PULLUP);
 if( digitalRead(WIO_KEY_A) == LOW) {
    lcd.print("Launching QSPI application\r\n"); // for LGFX
    ExtFlashLoader::ExtFlashLoader loader;
 }
////////////////////////////////////////// 
//
  //lcd.init();

```
platformio.iniには以下のライブラリを追加する：
```

lib_deps = 
    ...
        ＜省略＞
    ...
    #
    ciniml/ExtFlashLoader@^0.1.2


```

## 参照情報
LovyanGFX関連：
1. [LovyanGFX - Display (LCD / OLED / EPD) graphics library (for ESP32 SPI, I2C, 8bitParallel / ESP8266 SPI, I2C / ATSAMD51 SPI).
M5Stack / M5StickC / TTGO T-Watch / ODROID-GO / ESP-WROVER-KIT / WioTerminal / and more...](https://github.com/lovyan03/LovyanGFX)
1. [LovyanGFX LCD Graphics driver](https://macsbug.wordpress.com/2020/07/02/lovyangfx-lcd-graphics-driver/)

以上


