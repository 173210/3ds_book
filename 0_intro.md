#はじめに
違法コピー用のデバイス, Gateway 3DSが発売され,
その後3DS™ハックシーンはその解析や新たな発見により飛躍的な発展を遂げました.
私個人も, Gateway 3DSが発売されて以降,
aliak11氏とのオープンソースカーネルexploitプロジェクト_OSKA_や,
GPLのもとでライセンスされた_rxTools_の開発に携わってきました.
最近では, ハッキングカンファレンス_32C3_でplutoo氏によって公開された脆弱性の実証コードの作成を行っています.

本書ではその過程で暴かれた3DS™のセキュリティの実体について説明し,
読者の知的好奇心を満たすことを目的とします.

3DS™, DSi™, DS™, GBA™, Amiibo™は任天堂の商標です.

ARM946E-S™, ARM7™, ARM9™, ARM11™, MPCore™, CoreLink™はARMの商標です.

MIPS™はMIPS Technologiesの商標です.

PICA®はDMPの商標です.

FCRAM®, Fast Cycle RAM®は富士通の商標です.

TeakLite™はCEVAの商標です.

本書はCC-BY-SA 4.0の元で公開されます.
