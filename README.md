# RasberyPi指定時間GPIO出力
## 概要
キャンパスのチャイム（Raspberry Pi）をGPIO出力させるシェルスクリプトファイル

## 必要なもの
- Raspberry Pi 3 Model B
- Raspberry Pi用ACアダプター
- Raspberry Pi用ケース
- 導線

## コマンド

1. 平日0時にシェルスクリプトを実行するためにcronを設定する（動かない場合は曜日指定を1-5にする）
```
XDG_RUNTIME_DIR=/run/user/1000
0 0 * * 1, 2, 3, 4, 5 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/nagito-hiroshima/RaspberryPi-GPIO/master/chime.sh)"
```
2. 動作確認用  

タイムセットコマンド
```
curl -fsSL https://raw.githubusercontent.com/nagito-hiroshima/RaspberryPi-GPIO/master/chime.sh | sh
```
今すぐ確認コマンド
```
echo out > /sys/class/gpio/gpio14/direction;sleep 2s;echo in > /sys/class/gpio/gpio14/direction
```
Atジョブ全削除
```
at -l | while read a b;do atrm $a;done
```
## 広島キャンパス接続図
GPIO14と5Vを接続する


## 解説
ラズベリーパイのGPIOを扱うにはC#/Python/シェルスクリプトなど沢山ありますが  
今回は　****「シェルスクリプト」**** で作成していきます。  
以下のコマンドはラズベリーパイ標準のコマンドプロントに入力実行することが可能で、そこからシェルで作成しcronなどで定期的に実行するなど応用ができます。   

(n)と例があるところは下のように使いたい番号に変換してください。  
``` echo (n)```→```echo 21``` 
  
  
ピン配置  
<img width="1169" alt="スクリーンショット 2021-05-26 13 43 03" src="https://user-images.githubusercontent.com/68215637/119608201-87252c00-be30-11eb-887c-4a5c4e50f123.png">  
- - -  

ラズベリーパイのGPIOを変化させるには
あるディレクトリ内にあるファイルを書き換えることで自動でGPIOの出力が変化します。  
![名称未設定のノート (1)-3](https://user-images.githubusercontent.com/68215637/119607901-1847d300-be30-11eb-9a24-8b018f758fd0.png)
![名称未設定のノート (1)-4](https://user-images.githubusercontent.com/68215637/119608260-9f954680-be30-11eb-9272-0b3e22460ee3.png)


そこでまずこのコマンドを実行します。
```
echo (n) > /sys/class/gpio/export
```
このコマンドでは(n)のピンを使うためのファイルを作成してくれます。
簡単に言えば「今からこのピン使いますよ」と宣言しています。

そしてそのピンを出力として扱うのか入力として扱うのかを決めます
今回のスクリプトではOUTとINを切り替えて操作しています。（後述）
出力として設定するには
```
echo out > /sys/class/gpio/gpio(n)/direction
```
入力として設定するには
```
echo in > /sys/class/gpio/gpio(n)/direction
```
で設定できます。

- - - 

このスクリプトでは使ってないですが出力を切り替え（HIGHかLOW）するには  
![名称未設定のノート (1)-5](https://user-images.githubusercontent.com/68215637/119608411-e4b97880-be30-11eb-965e-fa2156dd14ed.png)

  
HIGHにする
```
echo 1 > /sys/class/gpio/gpio(n)/value
```
LOWにする
```
echo 0 > /sys/class/gpio/gpio(n)/value
```
GPIOを使い終わったら最後GPIOに
```
sudo echo (n) > /sys/class/gpio/unexport
```
を実行して「もう使わないピンだよ」と宣言してください。  

- - -
なぜ、出力モードを切り替えてON・OFFしたのかですが
普通であればGPIOピンを出力モードにして、GNDに接続すればよいのですが
RaspberryPiの出力は3.3vかつ最大出力電流は16mAしか無いためリレーを駆動できません。そこで、５Vの出力を活用してスイッチをON/OFFしてGNDを接続するシンク電流で作成しました。 
![名称未設定のノート (1)-6](https://user-images.githubusercontent.com/68215637/119608568-2518f680-be31-11eb-9129-29924f74ab58.png)  

初期設定でdirectionをOUTに設定するとcronなどで読み込みをしたとき毎回電流が流れてしまうため  
(OUTにするとラズベリー側はGNDになって電流が流れ込み通電状態になるため）  
directionの設定を時間していしてOUT・INを切り替えて疑似的にON・OFF再現しています。
![名称未設定のノート (1)-62](https://user-images.githubusercontent.com/68215637/119608631-3cf07a80-be31-11eb-9e00-2cc365fdd553.png)
![名称未設定のノート (1)-63](https://user-images.githubusercontent.com/68215637/119608635-3f52d480-be31-11eb-9377-787c2fd87f0d.png)
