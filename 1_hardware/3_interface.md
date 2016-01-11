#インターフェイス
##I²C
18個ほどのポートがあるようです.

###カメラ
ただのカメラ.

###MCU
Micro Control Unitだと思われます.

次の状態を得られます.

* 3Dスライダー
* 音量スライダー
* 電池残量
* GPUの何か
* RTC
* 電池状態
* 外部電源接続状態
* 電源ボタン
* HOMEボタン
* 無線LANスイッチ
* シェルの開閉
* LED通知状態

また, 次の動作を行えます.

* ハードウェアリブート
* ハードウェアシャットダウン
* LED通知

###LCDコントローラ
詳細不明.

###ジャイロスコープ
* InvenSense ITG-3200

InvenSenseの公式サイトにデータシート等があります.

http://www.invensense.com/products/motion-tracking/3-axis/itg-3200/

###UART
* NXP S750

赤外線通信に利用されます.

###加速度センサー
* ST LIS3DH

STの公式サイトにデータシート等があります.

http://www.st.com/web/catalog/sense_power/FM89/SC444/PF250725

###NFC
new 3DS™でAmiibo™等との通信に利用されます.

###QTM
new 3DS™で3Dの補正に利用されます.

##SD/MMCコントローラ
東芝製. SD/MMC/SDIOに対応しています. バグがあって私に恨まれています.
SDIOはデバッグにも利用?

###eMMC
* 3DS™: 東芝製, 1GB
* new 3DS™: Samsung製, 4GB

###無線LAN (SDIO)
* Atheros AR6014

##SPI
タッチスクリーンとデバッグ用?

##カード
###CTRCARD
3DS™のカードリーダー. CRCによるエラーチェック機能付き.

###NTRCARD
DS™のカードリーダー.

###SPICARD
SPIフラッシュとの通信に利用されます. Gateway 3DSで利用されました.
