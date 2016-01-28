#ntrcardhax
ntrcardhaxは, ARM9™のNTRCARD (DS™ 用カートリッジ)読み込みコードの脆弱性を突くexploitです.
脆弱性はハードウェア上の仕様とソフトウェア実装の組み合わせが原因です.

#実装とその意義
NTRCARDとのやりとりはFIFOをポーリングして行われます. ポーリングはカートリッジがビジー状態である限り行われます.
ここで, NTRCARDとのやり取りはMMIO (メモリマップドI/O) で行われますが,
この領域はなぜかARM11™でもアクセスできます. そのため,
読み込み開始前に要求するデータの容量をARM11™から増加させることで,
バッファオーバーフローを引き起こせます.

#結果
ARM11™カーネルからのARM9™権限掌握

#Exploit
ここで問題になるのは, 専用のハードウェアが必要になる点です. NTRCARDとして振る舞えるデバイスが必要になります.
今回はMIPS™ CPUからNTRCARDをエミュレーションするFPGAを制御することによって実装されているFlashcart,
DSTWOを用います. DSTWO用のSDKが公開されており, これによりMIPS™ CPUで動くプログラムを作成できます.

1. DSTWOでコードを起動
DSTWOではプラグインとしてMIPS™ CPUで動くコードを起動できます. なお, この操作はTWL\_FIRM,
つまりDS™モードで行います.

2. TWL\_FIRMを終了しNATIVE\_FIRMへ移行
ARM11™ Kernel ExploitはNATIVE\_FIRMで実行するのでTWL\_FIRMを終了します.
この際にリセット操作が行われても無視します.

3. ARM11™ Kernel Exploit

MMIOにアクセスするには当然カーネル権限が必要です. memchunkhax2等でちゃちゃっと奪取します.

4. カートリッジに読み込み命令を発行するスレッドを作成
このスレッドはカートリッジがビジー状態になったのを確認すると, ついでMMIOのレジスタにアクセスし,
受け取れる最大容量の16384バイトのデータ転送を常に要求します.

なお, 「常に」といっても, カートリッジがビジー状態の間はARM9™は読み込みを行えないため,
適度に遅延を発生させます. CPU速度ではARM11™の方が圧倒的に早いため, 遅延の制御は容易ですが,
メモリバスクロックも考慮する必要があります. 具体的な数値は次のような計算で得られます.

NTRCARDがレディになったのを確認してから実際に読み込みを開始するまで約58命令あります.
ARM9™の上限は134MHzですが, ARM9™からアクセスできるタイマーが約67108864Hz (67MHz) であることを実測しており,
DSも同じクロックなのでこのクロックで動作していると仮定します. 更に, 単純に1命令1クロックと仮定します.
これらの仮定を元に計算すると, その間隔は約1.2MHzであることが分かります. つまり,
命令を発行する間隔は1.2MHzより速く, 67MHzより遅くする必要があります. 十分広いと言えるでしょう.

5. ARM9™にNTRCARD読み込みを要求
普通のAPIを使ってNTRCARD読み込みを要求します. NTRCARDは時々レディになり,
ARM9™コードの続行を許しますが, その後すぐにARM11™からNTRCARDに読み込み命令が発行され,
ARM9™の命令は無視されます. よって, NTRCARDは16384バイト分のデータを送ってきます.
ところが, ARM9™コードは通常の512バイト分のバッファしか用意していません. ここで,
バッファオーバーフローが発生します.

#問題点
##ダウングレードで間に合うかもしれない
このexploitが使えるFWではダウングレードができるので, それで間に合うといえば間に合います.
特に, OTPがReisyukaku氏によってダンプされ, arm9loaderによるemuNAND対策も無効になった今,
ダウングレードはその代わりとしての役目を十分に果たせます.