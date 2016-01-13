#arm9loaderhax
arm9loaderhaxは, ARM9起動時に実行されるarm9loaderの脆弱性を突くものです.

#arm9loader
arm9loaderは暗号化されたARM9コードを復号し, 実行します.
arm9loaderはnew 3DSで新たに導入されました.

#実装とその意義
arm9loaderを含むFIRMは自己復号形式のFIRMです. すなわち,
ARM9セグメントにはarm9loaderと暗号化されたコードが含まれており,
エントリーポイントはarm9loaderを指しています. このため,
実行すると内蔵されているarm9loaderが自動的に暗号化された部分を復号化し, 実行します.

arm9loaderはeMMC内の特定のセクターをOTPのSHA-256ハッシュを使って復号化し,
それを鍵として暗号化されたコードを復号化します. その後, OTPへのアクセスを無効にします.
セクターは512バイト, AESのブロックは16バイトなので,
理論上512 / 16 = 32個の暗号鍵を選択できます.

以上により, 既存の暗号化に加え, ARM9コードに更新可能な暗号を施すことが可能になりました.

#リビジョンと欠陥
##初版
9.5.9未満のファームウェアに搭載されています.

ARM9コードを復号化するのに用いた鍵をそのまま暗号化エンジンに残しています.
このため, ARM9へのアクセスが得られれば容易に復号できます.

##第2版
ファームウェア9.5.9に搭載されています.

初版の欠陥を修正しました.

##第3版
9.5.9以降のファームウェアに搭載されています.

使用する暗号鍵を変更しました. すなわち, 初版の欠陥により漏洩した鍵を利用しなくなりました.

一方で, これまでの版で行われていた, __復号に使う鍵の確認を忘れる__という致命的な過ちを犯しました.
これにより, 鍵が格納されたセクターが破壊され, その結果鍵が不正なものになっても,
それを用いて復号化を試み, 実行してしまいます.

arm9loaderhaxはこの脆弱性をつくexploitです.

#結果
ARM9での, arm9loader起動直後の任意のコード実行

#Exploit
さて, お待ちかねのexploitです. 想定される障壁を回避するため, exploitはやや複雑になります.
その過程を順におって見てみましょう.

1. ARM9 Exploit

これにはARM9メモリとNANDへのファイルシステム抜きでのアクセスが必要になります.
これにはARM9 Exploitが必須です.

2. ハックの準備を行う

ARM9で次の処理を行うコードを実行します.

* 例外ベクタにインストールコードを書き込む

本来の例外ベクタはBootrom内に配置されていますが, Bootrom内の例外ベクタのコードは単にARM9メモリ先頭あたりにジャンプします.
この「ミラーされた」例外ベクタにインストールコードを書き込みます.

* ARM9セグメントが大きいFIRM0のヘッダを書き込む
* FIRM0のARM9セグメントに第2ペイロードを書き込む
* FIRM0のARM9セグメントの残りの部分に不正な命令を書き込む
* FIRM1に第3版のarm9loaderを含むFIRMを書き込む
* 鍵を格納しているNANDセクターを任意の値で破壊する

3. 再起動

再起動を行ってもメモリの内容は維持されます. ここで, 起動後は次の処理が行われます.

* FIRM0を読み込み, ARM9セグメントが不正になっていることを検知. FIRM1の読み込みを続行.

* FIRM1を読み込み, arm9loaderを実行
この時点でFIRM0から読み込んだコードがFIRM1で上書きされますが,
FIRM1より大きければ一部が残ります.

* arm9loaderがOTPのハッシュを使って鍵を格納しているNANDセクターの復号を試みる
暗号の特性から, NANDセクターを復号化したものはランダムな値に見えます.

* NANDセクターの復号の結果得られた不正な鍵でARM9コードを復号する
同様に, ARM9コードは擬似的にランダムなものに変化します.

* ARM9コードを実行する

ARM9コードはランダムなものに変化しているので, これは予測不能な結果を招きます.
無限ループに陥らない限り, 最終的には例外を発生します.

4. インストールコード実行

例外が発生すると, 例外ベクタに配置されたインストールコードが自動的に実行されます.
インストールコードは次の処理を行います.

* LRレジスタを確認

例外が発生すると, プログラムカウンタがLRレジスタに退避されます.
LRレジスタがFIRM0の内容が残っているはずの部分のアドレスを示す場合,
exploitは成功です. 次のステップに移ります.

そうでない場合, 例のNANDセクターをもう1度書き換え, 再起動します.
NANDセクターを書き込む直前に特定のレジスタにマジックナンバーをセットし,
再起動の直前にそのレジスタを確認します. もし異なる場合にはNANDセクター書き込み後の途中のコードから実行されている危険があるため,
マジックナンバーをセットする時点にジャンプします.

* 第1ペイロードをリロケート

LRレジスタが示すアドレスに第1ペイロードの先頭を合わせます. ペイロードがARM9セクターをはみ出てしまったら,
コードを途中で分割してLRレジスタが示すアドレスより前に残りのコードを書く必要があります.
その場合, ペイロード内の分岐命令を適切に変更します.

* 第1ペイロードをFIRM0の最後に書き込む
これにより, 次回以降の起動では必ず第1ペイロードが実行されるようになります.

* 第1ペイロードを実行

5. 第1ペイロード実行

FIRM1のARM9セグメントの容量とFIRM0のARM9セグメントの容量と差が第1ペイロードの最大容量となります.
そのため, 第1ペイロードはできるだけ小さくする必要があるため,
第2ペイロードを実行するだけの最低限のコードのみを搭載します.
第2ペイロードは既に初期化されているNAND内にあるため, 読み出しには最低限のコードしか必要ありません.

PC/AT互換機のブートストラップに似ています.

6. 第2ペイロード実行

第2ペイロードは次の処理を行います. U-Boot等のブートローダーがすることと同じです.

* SDMMCを初期化
* FATを初期化
* バイナリを読み込む
* バイナリを実行

#問題点
##arm9loader実行後である
起動時とはいえ, arm9loader実行後であるため, OTP等にアクセスできません.
そのため, 復号化されたFIRMが事前に用意されている必要があります.
これは9.2にダウングレードしてhomemenuhax, ARM11 Kernel Exploit,
ARM9 Exploitを組み合わせても__同様の結果を得られます__.

##危険
ランダムになったARM9コードが例外を発生させる前に無限ループに陥った場合, __3DSは文鎮になります__.
ハードウェアでeMMCを修復する必要があります.

##実装されていない
まだ実装がありません. homemenuhaxを使った安全な実装がある中で,
__arm9loaderhaxの実装を試みるものがいるかは疑問があります__.