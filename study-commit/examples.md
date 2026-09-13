# Study Commit Examples

<!-- This file is auto-maintained by the study-commit skill. Max 30 entries. -->

- feat: ch02に因果マスクと出力変換付きAttentionモジュールを実装
- feat: ch02にマルチヘッドAttentionを実装
- feat: ch02にマルチヘッドAttentionをnn.Moduleクラスとして実装
- feat: codebotにマルチヘッドAttention(Dropout付き)モジュールを追加
- feat: ch02に層正規化(LayerNorm)モジュールを実装
- feat: ch02にGELU・FFN・Transformerブロックを実装
- feat: codebotにTransformerブロック構成要素を集約
- feat: ch02にGPTモデル本体(埋め込み・ブロック積層・出力層)を実装
- feat: codebotにGPTクラス(埋め込み・ブロック積層・出力層・save/load)を実装
- feat: ch03に事前学習の学習ループとデータ準備・モデル保存を実装
- feat: ch03の事前学習を実行し損失曲線を保存(.ptは除外)
- feat: ch03に事前学習済みGPTでテキスト生成する02_generateを実装
- feat: ch03にSFTデータをAlpaca形式に変換しトークン化する03_alpacaを追加
- feat: ch03に指示チューニング(SFT)の学習ループを実装し損失曲線を保存
- feat: ch03にSFTモデルと対話する04_chatとgenerate関数を追加
- feat: ch03に強化学習(GRPO)のデータ・報酬・損失関数を実装する09_grpoを追加
- feat: ch03にGRPOの損失関数と学習ループを実装し精度曲線を保存
- feat: ch04に出現頻度重み付きBPE学習の最適化版を実装
- feat: ch04にペア位置キャッシュ付きBPE学習を追加しtiny_storiesをgitignore
- feat: ch04に終了トークンを基準にファイルを分割する03_bpe_chunkを追加
- chore: storybotにtiny_storiesデータセットの取得スクリプトを追加
- chore: .gitignoreに.vscodeを追加
- feat: ch04のtrain_bpeをチャンク境界ごとにファイルを読み込む方式に変更
- feat: ch04にmultiprocessingで事前トークン化を並列化する04_bpe_parallelを追加
- exp: ch04にmultiprocessing.Poolの動作を確認するasync_testを追加
- chore: ch04の04_bpe_parallelをColab対応にしDriveマウントセルを追加
- feat: storybotにencode_file(並列エンコード・memmap保存)付きBPETokenizerを実装
- feat: ch04にマージ優先度でエンコードするBPETokenizerを実装
- feat: ch04にencodeを並列化する07_encode_parallelのひな形を追加
- fix: codebotのtrain_bpeの最頻出ペア選択を決定的なタイブレーク付きに修正
