---
title: 時系列基盤モデルの潮流：TimeGPT・Chronos・Lag-Llama解説
emoji: 📊
type: tech
topics:
- machinelearning
- timeseries
- ai
- deeplearning
- forecasting
published: false
---

**Generated**: 2026-04-11 16:01 UTC
**Provider**: Gemini
**Research Duration**: 3 minutes 45 seconds
**Sources**: 47 verified web sources


---

# 時系列基盤モデル（Time Series Foundation Models）の最新潮流：アーキテクチャ、特徴、および応用展開

**Key Points:**
*   **パラダイムの移行:** 自然言語処理（NLP）分野での大規模言語モデル（LLM）の成功に触発され、時系列予測の分野においても「時系列基盤モデル（Time Series Foundation Models: TSFM）」が急速に台頭しています。
*   **ゼロショット予測の実現:** TSFMは、特定のタスクやドメインに依存しない数十億から数千億の時系列データポイントで事前学習されており、新しいデータセットに対して追加学習（ファインチューニング）なしで高精度な予測を行う「ゼロショット予測」を可能にしています。
*   **多様なアーキテクチャのアプローチ:** Amazonの「Chronos」、Nixtlaの「TimeGPT」、ServiceNow等の「Lag-Llama」、Googleの「TimesFM」など、主要なIT企業や研究機関から独自のトークン化戦略やTransformerアーキテクチャ（エンコーダ・デコーダ、デコーダのみ等）を採用した強力なモデルが続々とリリースされています。
*   **従来手法との比較と議論:** TSFMは強力な予測能力を持つ一方で、XGBoostやProphet、ARIMAなどの従来の統計手法や軽量な機械学習モデルが依然として特定のシナリオ（特に小規模データや計算リソースが限られた環境）において同等以上の性能を示す場合があり、適用領域の見極めが重要であると指摘されています。
*   **幅広い応用領域:** 小売業の需要予測、金融市場の分析、交通モビリティの予測、医療機関の来院数予測、インフラの異常検知など、多岐にわたる産業分野での実社会応用が進展しています。

**研究の背景と文脈**
時系列データの分析と予測は、過去のパターンから未来を推定し、企業の戦略的意思決定やインフラの最適化を支援する不可欠な技術です。従来はARIMAなどの統計モデルや、XGBoostに代表される機械学習、LSTMなどの深層学習が用いられてきましたが、これらはデータセットごとのモデル構築や特徴量エンジニアリングに膨大な時間を要するという課題がありました。近年、テキストデータに対して画期的な成功を収めた「基盤モデル（Foundation Models）」のアプローチを時系列データに適用する研究が加速しており、本レポートではこの最新の潮流について詳述します。

**本レポートの目的**
本レポートは、2024年から2026年にかけて発表・更新された最新の時系列基盤モデル（TimeGPT、Chronos、Lag-Llamaなど）に焦点を当て、その基盤となる技術的アーキテクチャ、トークン化の手法、および予測のメカニズムを学術的かつ包括的に解説します。さらに、各モデルの特徴を比較し、実際の産業や実務における応用事例、ならびに従来の機械学習手法とのベンチマーク結果を分析することで、時系列基盤モデルの真のポテンシャルと今後の課題を浮き彫りにすることを目的とします。

---

## 1. 時系列予測における基盤モデルの台頭と背景

### 1.1 時系列データの特性と従来の課題
時系列データとは、室温、電力消費量、来店者数、為替レート、株価など、時間の経過に沿って連続的または定期的に測定された数値データの系列を指します [cite: 1, 2]。時系列データは、一般的な表形式データとは異なり、データポイントの「順序」が極めて重要であり、順序をシャッフルすると重要な情報が失われるという特性（Sequential dependency）を持っています [cite: 3]。また、年間、月間、週間などで繰り返される「季節性（Seasonality）」、長期的な上昇・下降傾向を示す「トレンド（Trend）」、祝日やイベントなどによる「休日効果（Holiday effects）」など、複雑な変動パターンを含んでいます [cite: 3, 4, 5]。

伝統的に、これらの時系列データの予測には自己回帰モデル（AR）、移動平均モデル（MA）、およびこれらを組み合わせたARIMA（AutoRegressive Integrated Moving Average）モデルや指数平滑法などの統計的手法が広く用いられてきました [cite: 6, 7, 8]。これらの手法は実装が容易で解釈性に優れる反面、線形性の仮定が強く、複雑で非線形なパターンを捉えにくいという限界がありました [cite: 7, 8]。これに対処するため、近年ではXGBoostやLightGBMなどの勾配ブースティング決定木（GBDT）や、RNN（リカレントニューラルネットワーク）、LSTM（Long Short-Term Memory）、Transformerといった深層学習アルゴリズムが採用されるようになりました [cite: 7, 9, 10]。深層学習アプローチ、例えばDeepARやPatchTSTといったグローバルモデルは、複数の時系列データを横断して学習することで予測精度を劇的に向上させました [cite: 9, 11]。

しかし、これらの従来型機械学習および深層学習モデルには、「データセットごとに一からモデルを学習させる必要がある」という共通の課題が存在しました [cite: 7]。ドメイン特化型のモデルを構築するためには、大量の学習データの準備、専門家による緻密な特徴量エンジニアリング（Feature Engineering）、ハイパーパラメータの調整、および多大な計算コストが必要であり、データサイエンスの専門知識を持たない組織にとっては導入のハードルが高いものでした [cite: 12, 13, 14]。

### 1.2 NLP分野からのパラダイムシフトとTSFMの誕生
一方、自然言語処理（NLP）やコンピュータビジョン（CV）の分野では、膨大なデータを用いて事前に学習（プレトレーニング）された巨大なAIモデルである「基盤モデル（Foundation Models）」がパラダイムシフトを引き起こしました [cite: 1, 15, 16]。GPTやBERTに代表されるLLM（大規模言語モデル）は、一度の事前学習で言語の構造や一般的な概念を学習し、未知のタスクに対してもファインチューニングや、学習を全く行わない「ゼロショット（Zero-shot）」で高い性能を発揮します [cite: 15, 17, 18]。

この「大量のデータで事前学習し、多様なタスクにゼロショットで適用する」という概念を時系列予測に持ち込んだのが、時系列基盤モデル（TSFM: Time Series Foundation Models）です [cite: 3, 18, 19]。TSFMは、経済、気象、医療、IoTセンサー、ウェブトラフィックなど、さまざまなドメインから収集された数十億から数千億ポイントに及ぶ大規模な時系列データセット（実データおよび合成データ）を用いて事前学習されます [cite: 2, 11, 18]。

### 1.3 時系列基盤モデル（TSFM）の提供する価値
TSFMの登場により、従来の時系列予測では困難であった以下の3つの主要な機能が実現可能になりました [cite: 20]。

1.  **ゼロショット予測（Zero-shot Forecasting）:** 新しい未知の時系列データセットに対して、一切の追加学習（タスク特化型のトレーニング）を行うことなく、即座に高精度な予測を出力することができます [cite: 11, 20, 21]。これにより、モデル開発に必要な時間と計算コストが大幅に削減されます [cite: 13, 14]。
2.  **ファインチューニング（Fine-tuning / Transfer Learning）:** 対象となるドメインのデータが限られている場合でも、事前学習によって獲得した普遍的な時間的パターン（転移学習）をベースに、少量のデータでパラメータを微調整することで、ドメイン特化モデルと同等以上の精度を達成することが可能です [cite: 15, 20]。
3.  **時系列データの埋め込み表現（Embedding）の作成:** 文章や画像をベクトル化するように、時系列データをAIが処理しやすい数値ベクトル（埋め込み表現）に変換できます。これにより、時系列データの検索、クラスタリング、分類、異常検知などの下流タスクへの応用が容易になります [cite: 20]。

TSFMは、データサイエンティストがこれまで多大な時間を費やしてきたデータの前処理やモデルの微調整から解放し、より戦略的な意思決定に時間を割くことを可能にする「時系列分析の民主化」をもたらす技術として期待されています [cite: 8, 12]。

---

## 2. 時系列基盤モデルの主要なアーキテクチャ設計

NLPにおける言語モデルと異なり、時系列データは「連続的な数値」であり、サンプリング周波数（毎秒、毎時、毎日など）やスケール（桁数や変動幅）がデータセットごとに全く異なります。そのため、TSFMを構築する上では、Transformerアーキテクチャに時系列データをどのように入力するか（トークン化戦略）が極めて重要な設計要素となります。

### 2.1 トークン化（Tokenization）とデータ表現のアプローチ
基盤モデルが連続的な時系列データを理解するためには、データを離散的な「トークン（Token）」または適切なベクトル表現に変換する必要があります。現在、TSFMでは大きく分けて以下の2つのアプローチが主流となっています。

*   **パッチ化（Patching）:**
    連続する複数の時点データ（例えば12時間分のデータポイント）を1つの「パッチ」としてグループ化し、これを1つのトークンとして扱う手法です [cite: 2, 22]。このアプローチはコンピュータビジョンのVision Transformer（ViT）にヒントを得たものであり、系列長を短縮することでTransformerのAttentionメカニズムの計算量（系列長の2乗に比例する）を大幅に削減し、長期的な依存関係を効率的に学習できるという利点があります [cite: 22, 23]。
*   **スケーリングと量子化（Scaling and Quantization）:**
    AmazonのChronosなどで採用されている手法で、時系列データをテキストのように「離散的な語彙」にマッピングします。具体的には、時系列データの値をその絶対値の平均などでスケーリング（正規化）し、あらかじめ設定した「ビン（Bin）」に割り当てる（量子化）ことで、各値を特定のID（トークン）に変換します [cite: 2, 13, 24]。これにより、既存のLLM（言語モデル）のアーキテクチャを一切変更せずに、そのまま時系列タスクに転用できるという画期的な特徴があります [cite: 13, 24]。

### 2.2 Transformerアーキテクチャの分類
TSFMは、ベースとなるTransformerの構造によって以下の3つのカテゴリに分類されます [cite: 2]。

1.  **デコーダのみ（Decoder-only）モデル:**
    GPTやLLaMAと同様に、過去の系列から次のステップを自己回帰的（Autoregressive）に予測する構造です。TimesFM（Google）、Lag-Llama、TimeGPTなどがこのアーキテクチャを採用しています [cite: 2, 16, 25]。
2.  **エンコーダのみ（Encoder-only）モデル:**
    BERTのように双方向の文脈を考慮して表現を学習します。予測タスクよりも、時系列の一般的な理解、埋め込み表現の生成、異常検知（Anomaly Detection）や分類タスクに向いているとされます。SalesforceのMOIRAI（Encoderモデルとしての側面）などが該当します [cite: 2]。
3.  **エンコーダ・デコーダ（Encoder-Decoder）モデル:**
    T5アーキテクチャに代表される構造で、入力系列をエンコーダで表現に変換し、デコーダで未来の系列を生成します。AmazonのChronosがこのアプローチを採用しています [cite: 2, 11]。

### 2.3 予測の出力形式：点予測と確率的予測
時系列予測における出力形式には、単一の予測値を出す「点予測（Point Forecast）」と、不確実性を伴う分布を出力する「確率的予測（Probabilistic Forecast）」があります。

*   **点予測:** 未来の特定の時点における「最も確からしい単一の値」を出力します。実装がシンプルですが、予測がどの程度確かなのかという「不確実性」を評価できません [cite: 26]。
*   **確率的予測:** 未来の各予測ステップに対して、確率分布（例：平均と分散、あるいは分位数）を出力します。これにより、予測値が収まる信頼区間（Prediction Interval）を算出でき、「この範囲内に収まる確率は90%である」といったリスク評価が可能になります [cite: 1, 26]。ビジネスの意思決定（在庫管理や金融リスク評価など）においては、不確実性を定量化できる確率的予測が非常に重要視されます [cite: 27, 28]。確率的予測を実現する手法として、分布のパラメータを直接予測する手法（Lag-Llamaなど）や、Conformal Prediction（TimeGPTなど）といった枠組みが用いられています [cite: 25, 29]。

---

## 3. 主要な最新時系列基盤モデルの詳細分析

2023年後半から2026年にかけて、世界トップクラスのIT企業や研究機関から多様なTSFMがオープンソースまたは商用APIとしてリリースされました [cite: 2, 20]。ここでは、代表的なモデルである「TimeGPT」「Chronos」「Lag-Llama」を中心に、その技術的特徴を深掘りします。

### 3.1 TimeGPT (Nixtla)
**概要と背景:**
TimeGPTは、時系列予測プラットフォームを提供するNixtla社によって開発された、時系列データ用の初の大規模生成事前学習トランスフォーマーモデルです [cite: 30]。経済、気象、医療、エネルギーなど、さまざまなドメインから収集された1000億（100B）以上のデータポイントを含む過去最大規模のデータセットを用いて事前学習されています [cite: 12, 30, 31]。

**アーキテクチャと特徴:**
*   **汎用的なゼロショット推論:** 追加のトレーニングなしで、新しいデータセットに対して高精度なゼロショット予測を実行できます [cite: 12, 31]。
*   **Conformal Predictionによる不確実性評価:** TimeGPTの際立った特徴の一つは、予測の不確実性を定量化するために「Conformal Prediction（共形予測）」と呼ばれる非パラメトリックな手法を採用している点です [cite: 29, 31]。特定のデータ分布（正規分布など）を仮定せず、クロスバリデーション等を用いて予測区間（Prediction Intervals）を構築するため、異常値が存在する状況下でも頑健（ロバスト）で信頼性の高い信頼区間を算出できます [cite: 27, 29]。
*   **共変量（Exogenous Variables）の統合:** 曜日、休日の有無、季節性データ、イベントフラグなどの補助情報（外生変数）を特徴量としてモデルに組み込むことができ、予測精度をさらに向上させることが可能です [cite: 31, 32]。
*   **異常検知（Anomaly Detection）機能:** 予測モデリング技術を応用し、時系列データの通常のパターンから外れた異常値や予期せぬイベントを迅速に特定する機能も備えており、サイバーセキュリティの不審なネットワークアクティビティの発見や、金融市場の急激な変化の検知に有用です [cite: 29, 32]。

### 3.2 Chronos および Chronos-Bolt (Amazon Science)
**概要と背景:**
Chronosは、Amazon Scienceによって2024年3月に発表されたモデルファミリーです [cite: 22, 24]。言語モデルがテキストを扱うアプローチを、そのまま時系列データに適用するという画期的なコンセプトに基づいています。

**アーキテクチャと特徴（オリジナルのChronos）:**
*   **スケーリングと量子化によるトークン化:** Chronosは時系列データを直接数値として入力するのではなく、コンテキストの値を絶対値平均で割ることでスケールを統一（Scaling）し、あらかじめ設定した4094個のビン（特殊トークン2つを加えて4096個の語彙）に割り当ててトークンIDに変換（Quantization）します [cite: 3, 13, 24]。推論時には逆の処理（DequantizationとUnscaling）を行い、元の実数値に戻します [cite: 24]。
*   **T5アーキテクチャの活用:** トークン化された時系列データは、NLPで実績のあるエンコーダ・デコーダアーキテクチャであるT5（Text-to-Text Transfer Transformer）に入力されます。言語モデルの構造を一切変更せずに、クロスエントロピー損失を用いて学習させます [cite: 2, 24, 33]。
*   **自己回帰的サンプリングによる確率的予測:** 推論時にはモデルからトークンを自己回帰的（1ステップずつ順番に）にサンプリングし、複数の軌跡（Trajectories）を生成することで、予測の確率分布を獲得します [cite: 21, 33]。

**Chronos-Bolt と Chronos-2 への進化:**
2024年後半には、アーキテクチャを大幅に改良した**Chronos-Bolt**がリリースされました [cite: 22, 33]。
*   **パッチ化と直接マルチステップ予測:** Chronos-Boltは、過去の時系列データを1点ずつではなく、複数のデータポイントからなる「パッチ（Patch）」に分割してエンコーダに入力します [cite: 11, 22]。デコーダはこれらの特徴表現を用いて、将来の複数時点の予測値（分位数）を1ステップずつではなく「一度に生成（Direct multi-step forecasting）」します [cite: 11, 22]。
*   **圧倒的なパフォーマンス:** このパッチベースのアーキテクチャにより、Chronos-BoltはオリジナルのChronosモデルと比較して**最大250倍高速**であり、**20倍のメモリ効率**を実現しました。さらに予測誤差も5%削減されています [cite: 21, 22, 33]。モデルサイズはTiny(9M)、Mini(21M)、Small(48M)、Base(205M)の4種類が提供されており、エッジデバイスやCPU上での推論も現実的となっています [cite: 11]。
*   2025年後半にリリースされた**Chronos-2**では、従来の単変量予測に加え、多変量予測（Multivariate）や共変量を考慮した予測（Covariate-informed forecasting）にもゼロショットで対応できるようになり、最新のベンチマーク（fev-benchやGIFT-Eval）で最高性能を達成しています [cite: 14, 18, 22]。

### 3.3 Lag-Llama (ServiceNow, Morgan Stanley, Mila 等)
**概要と背景:**
Lag-Llamaは、2023年10月に発表された、単変量確率的時系列予測のためのオープンソース基盤モデルです [cite: 16, 25]。大規模言語モデルであるLLaMAのアーキテクチャに着想を得ており、時系列データのラグ（遅延）特徴量を共変量として扱う点が特徴です [cite: 16, 25, 34]。

**アーキテクチャと特徴:**
*   **LLaMAベースのデコーダのみアーキテクチャ:** Lag-LlamaはデコーダのみのTransformerモデルであり、各アテンション層のクエリとキーの表現にRMSNorm（Root Mean Square Normalization）やRoPE（Rotary Positional Encoding）を組み込むことで、学習の安定性と位置情報の認識能力を高めています [cite: 25, 35, 36]。
*   **ラグ特徴量と日付・時間特徴量の統合:** 入力データのトークン化において、特定のサンプリング周波数に依存しない汎用的なアプローチを採用しています。過去の時系列値から構築された「ラグ特徴量（過去の時点の値）」と、秒、分、時、日、週、月、四半期といった「日付・時間特徴量（Static covariates）」を連結してモデルの入力ベクトルを構築します [cite: 25, 34, 35]。これにより、テスト時に初めて見るデータ周波数であっても柔軟に対応できます [cite: 25, 34]。
*   **分布ヘッド（Distribution Head）による出力:** Lag-Llamaの最終層は、抽出された特徴量を特定の確率分布のパラメータに射影する「分布ヘッド」となっています [cite: 25, 34]。初期の実験では、柔軟性が高く裾が広い「Studentのt分布（Student's t-distribution）」が採用されており、モデルは未来のデータポイントに対する「自由度、平均、スケール」の3つのパラメータを出力します [cite: 25, 37]。これにより、予測の信頼区間を直接的に生成することが可能です [cite: 34, 37]。
*   **値のスケーリング:** 入力データの数値の大きさが系列ごとに異なる問題に対処するため、単変量ウィンドウごとに平均値と分散を計算して系列を正規化し、さらにこれらの要約統計量をモデルに入力することで、データの統計的背景をモデルに提供しています [cite: 25, 37]。

### 3.4 その他の注目すべき基盤モデル
時系列基盤モデルの開発競争は激化しており、他にも多数の有力モデルが存在します [cite: 18, 19]。
*   **TimesFM (Google):** 1000億以上のデータポイントで事前学習されたデコーダのみのモデルです。時系列データを「パッチ」として扱う概念を普及させ、特にゼロショット性能においてTimeGPTやChronosと並ぶ高い評価を得ています。最新バージョンのTimesFM-2.5では、コンテキスト長を16Kに拡大しつつパラメータ数を半減させ、高い効率性を実現しています [cite: 2, 18]。
*   **MOIRAI (Salesforce):** エンコーダ・デコーダアーキテクチャ（エンコーダモデルとしての利用も重視）を採用し、大規模な事前学習データセット「LOTSA」を用いて学習されたモデルです。特に多変量予測や様々な周波数のデータに柔軟に対応する設計（Any-variate対応）が特徴です [cite: 2, 3, 20]。
*   **Granite Time Series TTM (IBM):** 非常に軽量（805Kパラメータなど）でありながら高いパフォーマンスを発揮するTiny Time Mixerモデルです。リソースに制約のある環境での運用に適しています [cite: 3, 20]。
*   **Sundial (Tsinghua University):** 清華大学が開発したモデルで、1兆（1 Trillion）時点という極めて巨大なデータセットでプレトレーニングを行っており、学術会議で高い評価を得ています [cite: 18]。
*   **Cisco Time Series Model:** 2026年初頭にリリースが予定されているオープンウェイトのモデルで、特にシステムのオブザーバビリティ（可観測性）やセキュリティ領域における運用・自動化に焦点を当てています [cite: 38]。

---

## 4. 従来モデル（統計手法・機械学習）とのベンチマーク比較

TSFMが画期的である一方で、「これまでのモデルを完全に置き換えるのか？」という問いについては、学術的・実務的な観点から活発な議論が行われています。ここでは、XGBoostやProphetといった従来の予測手法とTSFMの比較結果について詳述します。

### 4.1 精度評価（Accuracy and Zero-shot Generalization）
複数のベンチマークテスト（GIFT-Eval、fev-bench、その他学術研究）において、TSFMのゼロショット予測能力は総じて高い評価を受けています [cite: 18, 19]。
Amazonの評価では、Chronos（およびChronos-Bolt）は学習に使用していない完全に新しいデータセットに対しても、従来の統計手法（AutoARIMA, ETSなど）やドメイン特化の深層学習モデルを大きく上回る、あるいは同等の性能を示しました [cite: 11]。また、TimeGPTの研究チームの報告によれば、TimeGPT-1は他の基盤モデル（TimesFM、Chronos、Moirai、Lag-Llamaなど）や既存の機械学習手法と比較して、精度と推論速度の総合バランスでトップクラスに位置するとされています [cite: 39]。

しかし、NeurIPS 2025で開催されたTSFMに関するワークショップでは、より慎重な見解も示されました。「慎重に設計された軽量な教師あり学習のベースライン（XGBoostやLightGBMなど）が、依然としてTSFMの性能と同等か、場合によっては上回るケースがある」という報告です [cite: 18, 19]。例えば、特定の実世界データセット（レストランの売上データなど）においては、日付や天候、イベントフラグなどの特徴量を綿密に作り込んだXGBoostモデルや、季節性を考慮したSARIMAXなどの古典的手法が、強力なベースラインとして機能することが確認されています [cite: 10, 40]。

### 4.2 計算コストと推論速度（Latency and Resource Constraints）
モデルの実運用において、推論レイテンシ（遅延）と必要な計算リソースは決定的な要素です。
*   **統計手法・XGBoost:** 非常に軽量であり、推論はほぼ一瞬（例: SARIMAXが0.07秒、XGBoostが0.01秒）で完了し、CPUのみで効率的に動作します [cite: 23]。
*   **TSFM（大規模モデル）:** TimeGPTやオリジナルのChronosなどは、Transformerの大規模な行列演算を伴うため、推論に数秒〜数十秒かかる場合があり、運用にはGPU環境が推奨されます [cite: 11, 23]。例えば、ある病院の救急外来予測のベンチマークでは、TimeGPTの推論時間は1.5秒程度であり、運用上許容範囲内ではあるものの、従来手法に比べればレイテンシは高いと評価されています [cite: 23]。また、Amazon Chronosについては、古典的な統計モデルのトレーニングよりも精度が低く、500%遅いと指摘する批判的な見解も存在します [cite: 19]。
*   **効率化されたTSFM:** この課題に対し、Chronos-BoltやIBMのTTM、TimesFM-2.5などの最新モデルは、パッチ化やモデルの小型化により、CPUでも高速に動作するよう劇的に改善されています [cite: 3, 18, 21]。特にChronos-Boltは、元のChronosモデルと比べて250倍高速でありながら精度を向上させており、実運用におけるハードルを大幅に下げました [cite: 11, 21]。

### 4.3 実装と運用の容易さ（Ease of Use and Feature Engineering）
XGBoostなどの機械学習モデルの最大の欠点は、データサイエンティストによる「特徴量エンジニアリング（Feature Engineering）」が必須である点です。過去のラグ変数、移動平均、時間的特徴（曜日、時間帯の正弦/余弦変換など）、休日フラグなどを手動で設計しなければ高精度な予測は得られません [cite: 10]。
対照的に、TSFMは時系列データをそのまま（あるいは最小限のスケーリングで）入力するだけで、モデル自身が内部的にトレンドや季節性、複雑な依存関係を抽出して予測を行います [cite: 10, 12, 23]。この「構築とデプロイの手間の少なさ」こそが、TSFMが従来手法に対して持つ最大のビジネス上の優位性です [cite: 12, 14, 23]。

---

## 5. 時系列基盤モデルの応用事例とユースケース

時系列基盤モデルは、その高い汎用性とゼロショット学習能力を活かし、様々な産業分野での実用化が始まっています。ここでは具体的な応用事例を紹介します。

### 5.1 小売業・サービス業の需要予測（Retail and Demand Planning）
小売業における商品ごとの売上予測や、飲食店の来店者数予測は、在庫の最適化、食品ロスの削減、および人員配置の効率化に直結します [cite: 15, 41]。
*   **応用例:** ドイツのレストランチェーンのデータを用いた研究では、数年にわたる時間ごとの売上データに対して予測が行われました。Chronosなどの基盤モデルは、休日のフラグや天候などの補助変数（外部特徴量）を明示的に入力しなくても、時系列データの変動パターンから直接ゼロショットで非常に競争力のある予測精度を達成しました [cite: 10]。また、Chronos-Boltを用いたEコマースカタログの売上予測事例では、数千のアイテムに対する複数ステップの将来予測が、従来モデルに比べて圧倒的に短い時間で計算可能であることが示されています [cite: 11, 21]。

### 5.2 金融市場とマクロ経済予測（Finance and Macroeconomics）
金融市場における株価、為替レート、金利の予測、およびインフレ率やGDPなどのマクロ経済指標の予測は、投資戦略やリスク管理において極めて重要です [cite: 1, 42]。
*   **応用例:** 従来、経済予測にはベイジアンVAR（ベクトル自己回帰）などの線形モデルが主に使われてきました。現在、TimeGPT、Moirai、TimesFMなどの基盤モデルが経済予測に応用されています。これらは現時点で従来の計量経済学手法を完全に上回る水準には至っていない場合もありますが、非線形のダイナミクスを捉える柔軟性により、特定の期間や変数において精度の向上が確認されています [cite: 42]。
*   **リスク評価:** 株価予測において、NixtlaのTimeGPTやStatsForecastを用いた事例では、Conformal Predictionを利用して予測の信頼区間（Prediction Intervals）を生成することで、単なる価格の点予測ではなく、市場のボラティリティ（変動性）や不確実性を考慮した厳密なソルベンシー（支払余力）評価が可能になっています [cite: 27]。

### 5.3 交通・モビリティ予測（Mobility and Traffic Forecasting）
都市部における交通量、公共交通機関の混雑度、タクシーやシェアサイクルの需要予測は、都市交通の運用最適化に不可欠です [cite: 6]。
*   **応用例:** ニューヨーク市およびオーストリア・ウィーンの自転車シェアリングデータを用いたベンチマーク研究では、TimeGPTがゼロショットで適用されました。短期（1時間）、中期（12時間）、長期（24時間）のすべてのホライズンにおいて、TimeGPTはARIMAやST-ResNetといった従来の深層学習・統計アプローチと比較され、データが少ないシナリオ（データスパースな環境）において極めて有効な選択肢であることが実証されました [cite: 6]。
*   **タクシー乗車数予測:** ニューヨークのタクシー乗車数のデータセットを用いた検証において、ChronosはXGBoostと同等以上の予測精度（RMSEやMAEの観点）を示し、事前のモデル設計なしで市場の変動を的確に捉える能力が確認されています [cite: 41]。

### 5.4 医療・ヘルスケア（Healthcare Operations）
病院の運用において、救急外来（Emergency Department: ED）の来院患者数を正確に予測することは、医師や看護師の適切な人員配置、ベッドの確保、および医療崩壊を防ぐために重要です [cite: 23]。
*   **応用例:** 実世界の救急外来来院データを用いた研究で、TimeGPTがSARIMAX、Prophet、XGBoostと比較されました。TimeGPTは、追加の再学習なしのゼロショット環境において、SARIMAXと並んで常にトップクラスの精度（MASE, RMSE指標）を記録しました [cite: 23]。特に、XGBoostが特徴量エンジニアリングを行っても精度が低迷したのに対し、TimeGPTは少ない運用負荷で信頼性の高い予測を提供できることが示されました [cite: 23]。

### 5.5 エネルギー管理とインフラ監視（Energy and Infrastructure）
電力需要の予測、発電設備のメンテナンス時期の予測、IoTセンサーデータからの異常検知は、エネルギー効率の向上とシステムダウンタイムの回避に直結します [cite: 1, 41]。
*   **応用例:** TimeGPTやProphetを用いた電力使用量データ（PJME）の予測検証において、TimeGPTは季節性やトレンドの変動を精密に捉えました [cite: 43]。また、変圧器の温度予測タスクにおいて、Chronos-T5モデルが用いられ、過去の温度遷移のデータから未来の温度変化の分布を確率的に出力することで、機器の過熱や故障の予兆を事前に検知（予知保全）する取り組みが行われています [cite: 12, 44]。TimeGPTの異常検知機能は、通常のパターンから逸脱したセンサーデータをリアルタイムでフラグ付けし、インフラの安全性を高める用途にも活用されています [cite: 29]。

---

## 6. 時系列基盤モデルの課題と今後の展望

TSFMは飛躍的な進化を遂げていますが、本格的な商用普及やミッションクリティカルなシステムへの導入に向けては、いくつかの技術的・実務的な課題が残されています。

### 6.1 データ汚染（Data Contamination / Data Leakage）の懸念
基盤モデルの評価において最も深刻な懸念事項の一つは、評価に用いるテストデータ（ベンチマークデータセット）が、モデルの「事前学習データ（Pre-training data）」に既に含まれてしまっている可能性（データリーク）です [cite: 6, 11]。
例えば、多くの研究論文で予測精度を測定するために用いられるM-Competitionsのデータや、Wikipediaのトラフィックデータなどは一般に公開されており、ChronosやTimeGPTの大規模な学習コーパスに含まれている可能性が非常に高いと指摘されています [cite: 6, 11, 39]。これにより、モデルが純粋な「未知のデータ（ゼロショット）」に対して予測を行っているのか、単に「記憶したデータ」を再現しているだけなのかを厳密に区別することが難しくなっています [cite: 6]。今後の研究では、完全に未公開で新しいプライベートデータセットを用いた、厳格な汎化性能の評価が求められています [cite: 6]。

### 6.2 共変量（外生変数）のシームレスな統合
小売の売上予測における「プロモーションの有無」や、電力需要における「気温・湿度」など、対象となる時系列以外の外部要因（共変量：Covariates）は予測精度を劇的に左右します [cite: 10, 11, 15]。
初期のTSFM（Lag-Llamaや一部のChronos等）は、本質的に「単変量（Univariate）」の予測に特化しており、外部変数をプロンプトとして組み込むことが難しい、あるいは考慮できないという制約がありました [cite: 11, 15]。
これに対する解決策として、現在2つのアプローチが進行しています。
1.  **アンサンブル/回帰モデルとの組み合わせ:** AutoGluon-TimeSeriesなどのフレームワークでは、外部特徴量を用いた「共変量回帰モデル」でまず予測を行い、その残差（予測しきれなかった誤差）をChronos-Boltのような単変量TSFMで予測するというハイブリッド手法が実装されており、大幅な精度向上が報告されています [cite: 11]。
2.  **アーキテクチャレベルでの対応:** Chronos-2やSalesforceのMoirai、Lag-Llama（ラグ特徴量等としての統合）のように、モデルのアーキテクチャ自体に共変量や多変量（Multivariate）の情報を直接入力・処理できるメカニズム（グループアテンションと時間アテンションの二重構造など）を実装する動きが進んでいます [cite: 18, 22, 34]。

### 6.3 ファインチューニングの必要性と限界
TSFMは「ゼロショット」を謳っていますが、現実の複雑なビジネスシナリオにおいては、ドメイン特有の特殊な非線形性やノイズに対応するため、結局のところ「フル・ファインチューニング（Full fine-tuning）」が必要になるケースが多いとNeurIPS 2025のワークショップで指摘されています [cite: 19]。
NLPにおけるChatGPTのように「そのままプロンプトを投げるだけで完璧な出力が得られる（BERT Momentのようなブレイクスルー）」段階には至っておらず、実用化に際しては、自社のデータで軽量なファインチューニング（転移学習）を行うプロセス（例：SageMaker等を用いたChronos-Boltの追加学習）を組み込むことが推奨されています [cite: 11, 15, 19]。ファインチューニングを行うことで、基盤モデルの汎用的な知識とドメイン特有の知識が融合し、予測精度が飛躍的に向上することが確認されています [cite: 11, 26]。

### 6.4 コンフォーマル予測とデータ制約時の信頼性
データ量が極端に少ない（データスパースな）状況では、従来のXGBoostや統計モデルは有効に学習できず、不確実性（信頼区間）のキャリブレーションが不安定になります [cite: 9]。最新の研究では、TSFMをコンフォーマル予測（Conformal Prediction）の枠組みで用いることで、事前学習された知識を活用し、限られたデータでも非常に安定した、信頼性の高い予測区間を生成できることが示されており、これはTSFMの大きな強みとして認識されつつあります [cite: 9]。

### 6.5 エコシステムの発展と今後の展開
2025年から2026年にかけてのTSFMの潮流は、「モデルの巨大化」から、「推論効率の最適化（高速化・軽量化）」と「ツールキットへの統合による民主化」へと移行しています [cite: 14, 45]。
Amazon SageMaker JumpStart、AutoGluon、Hugging Faceなどのプラットフォームを通じて、数行のコードでChronos-BoltやTimeGPTを利用・デプロイできる環境が整備されました [cite: 11, 33, 46]。また、予測プロセスをAIエージェントと統合し、ユーザーからの自然言語での問い合わせ（例：「来月の売上予測と在庫の最適化を提案して」）に対して、TSFMがバックエンドで推論を行い回答を生成するといった、「エージェント的ワークフロー（Agentic workflows）」への組み込みも進んでいます [cite: 8, 38, 47]。

---

## 7. 結論

時系列基盤モデル（TSFM）は、自然言語処理分野におけるLLMの成功を時系列データ分析に波及させた、予測モデリングにおける重要なパラダイムシフトです。TimeGPT、Chronos、Lag-Llama、TimesFMといったモデルは、それぞれ「コンフォーマル予測の統合」「言語モデルのトークン化技術の応用」「ラグ特徴量を用いた確率分布出力」など独自のアプローチを通じて、ゼロショットでの強力な予測能力を証明しました。

これらのモデルは、小売の需要予測、金融市場の分析、インフラストラクチャの異常検知、医療現場の運用最適化など、予測の迅速性と適応性が求められるあらゆる産業分野において、即座に価値を提供するツールとなりつつあります。とりわけ、特徴量エンジニアリングの手間を省き、AIの専門知識を持たないアナリストでも高度な予測を利用可能にする「民主化」の意義は計り知れません。

一方で、ベンチマークにおけるデータ汚染の懸念、XGBoost等の軽量な従来手法との精度・コストのトレードオフ、外生変数の扱いといった課題も存在しており、TSFMがすべての従来手法を直ちに置き換える「万能薬」ではないことも事実です。実社会への適用においては、課題の性質、利用可能な計算リソース、データの特性（単変量か多変量か、共変量の有無など）を見極め、状況に応じてTSFMのゼロショット予測、ファインチューニング、あるいは従来の軽量モデルを適切に使い分けるハイブリッドな戦略が求められます。

今後、アーキテクチャのさらなる最適化（Chronos-Boltのような高速・軽量化）やマルチモーダル化の進展により、時系列基盤モデルは企業戦略や社会インフラの自律的かつ高度な意思決定システムの中核として、より一層不可欠な存在へと進化していくことが予想されます。

**Sources:**
1. [soracom.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFXwIsFH8jGCxQZs-fqlOIoLyZyF1fasFRpOpd0Btn3dDv4OIFWa29YmIgFXEci_5lorXa8IICW74J05w1dATdPSRYOTThvqr8pm9yGey3FdcsDKyp5EYqshtbnFXYk-IHVjQBBLifWIlYHet6EbSlWXc4vqmecDDvwEAgBczf5ViDGYPA=)
2. [medium.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQF0EsoRZU9Bjvp8QbVwwTAKjZSAuQxUxXGdd9RUtFviU43A5NekdxkQ69EHVV-2MCiZ_64F-VGvVzZcUOTYuAA8LfyzHCx7zLKob_JZU8EwRVsNxqi_3iAC3lrLRHpVFEfLw-gtBWFQbieWNK7wlNUdTCArLeGMTxbdHE3i4lDrDqUKcxB7DXZQl6TyXmDJvZVY)
3. [towardsai.net](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG-xunPqIQYFKqYPZyGEM3P4rA3NjcfniJpNrbZJMN7ja7DkP0qOJsQYpQxcS6mF5riWltW0ltpPBnvNCx_vtFWy06a47Bz9weAn6bpKdU_WwiS09jU2KsmmDljsd36Wh1mc0Cpe947WNS3ROu5rK22uHaIrnxhPNjzFGdwLE1bovASK56KdiugxfFGYr7KZQvgncHNwQ==)
4. [speakerdeck.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEuw16U0pijgrr9p0rk768ESHPyzMatE8C_dxTTypfEn_TmojG7HsPcmK1WSxOuueqludvmVSst019Pj3swlUC0qBUbQj1dFt2ZMTAP9keTr8Gil_NFGie78VFXKKE8S-fS-Xmcgd1ZF2w6IPdSoOxEqF1RYvF01YgtWZaLXao=)
5. [hatenablog.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHcwN25XDHofoHa1H4L3RdvoTsLaiSsnBZ8v_q1pYxrhcYmn1Ymkn59U5aNFuFEekONcaMRyg7AQpWQoNpHS-hTNZ-CLGmRgRddzdxT7mYO6kMnpan5UwCGe7LLz8PWz5Dv0HgttlPiYK5z6XgGwg==)
6. [arxiv.org](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH555QxmxHQt2OKDcFIUJ-ALqEuCUHGIQl_snjT1f4bC5gvLPDMZx_9KAAJNSMXv3r9KcXYCWuErdc6dOTELOLimE8JdPx_dPfVKpR30gR4DLcCm2wX5-FWyg==)
7. [zenn.dev](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH4Crkji0miUPkPBoX46wqW8Uz_u8QUv0DXip9gDnl41rspLRyw8kuREsJfFypdM_DPWdX1Ve8ZdynZbSKruKyXqa2W_OpF-VG1W7Obxv6TW3ezFLK669SL8WTqlLEDgl7TrA==)
8. [zenn.dev](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFweqkRb-WM-GjbT6jfJEkpA74pPXUBVmCmyTTSPgrAP2IxC47gnXEs9YSOheS0b_3Y02X55lQaTMmxdkAlSVUtN_u_YP_tmNkIoBxiRCCBJgnP5562VNyMxEnxslzwYxi211JUCW_AkA==)
9. [arxiv.org](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH9jH6qWzII45AdgEXMulBNgyg2mQh5O-4MXRrSXdD80NFBK4uZMqcbi6X67fET00PLkttZP3AIIhiSg_b06SxSJFiHxZxcuipOgcpfXFwlTM0l2LrBVszseg==)
10. [themoonlight.io](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQF1kIqA4FEN4wH-857dtIw-x15OwW5FG_L77qBKPIaQuqh7wajELRW4HKI2jfYYwxNw9XVbeSvS_CO3nvQWpOI6ShN9tKOCCZS-Q__ycqvJEWJNN7208arjAMiTYCOWUA8i4vMzwfBvttR02lXLF2WOl-PbwejdSyvoj2bGcK1UOY3rLJoXhQveC6Jn05dJLqubsoz-pmLJtygiUx6WdJ5T9LNpDDWoMprun5osFThOmVbQ_z1b-CpBTtRSzm3tv2G1fnzW-iKD9BIdqUaUDBKyWUaWGg==)
11. [amazon.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGAAzqNBQhAXoQTiSyShxhB8pNZXKu8laiNfnk3gwShEyn5PNmwl8EEC0g3fwGlywtJgRvMTEZz7Jem2dCMkMsytNRy3QedrSDX6QUvYL1_mQE7lKWcA66q6tN5tBvzwSeHB4HbSeIs1PPTvn6yBvVjowhp6vSm3TFcTaIBq3g8Ip6AghLr1O8h7Ty7qrBZGvFxV0zW-vNPbeKpmM-EZv21811_IQSsKVpX0Uhldf0m6Nfvv0E=)
12. [industrial-data-science.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGAVRMpPHVh7DMbGG2SwGmN_a_6V8OhlM0NoK8UIFIN2gTFpfVM-ftYn3dw3CNgpiuhGIb6WxLnlJ_3_nT7ONWdPtcYikCZT_QEDzVkMbTUUbTe4rDoMkZVlCH8m0d2lLYaQVBGx2bOPB54dR0i19HyukP-SrKcsJGIcauEFWkGhJ_ApShvriZGAbA27eTZNEZbrpdvEpRoz8rubTV5LCVSz4H-GGebeCr6kDrxsKOi2uavsN9i4RMAS2hxNdX_lhT6Qee22ZeKV3UrySkCwYpq0YRWGw==)
13. [amazon.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGuLXZhy9094RXhSZ_GkEP5E-8ezCK9RyrcRqq2RncBHDTwIYFz5fhtubsXX2f33r3F76lJeh-Bsi7XxTB0ZqhsoN7QCzve2GdtQXAZF1OnIC2eZK6fWgU1VA8BeX5ke1Oxw2zq4LJ_wBPub_qpljytKbwtNRn3raffEKoO0SSkjGAfLNqBk4pX1tT068hhv6rShqzO6QGsuqaDcH7TOxKWDLQ-X_gTLov-x9Cop3Y=)
14. [machinelearningmastery.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGNiZNg__ar4OT2bRvoBbTiHEnxMRddwA6osVSqHnsKkjI1GI8GBRde8nShw23doSbaHHaIbz5g0LfuRcNMJoFkuHgaAqtNoGJddl-KxGwv4voxSeDw1VvbbfMg9te7FLG_Pci5LkAnHntImQ4WDeDFoFZpf2vOKGEkCY64kDsNWGMQxOBhEnTp8k9l55kMSE6QB_c8mBvCbsPnfN-GTEtj2JXWwuM=)
15. [superlinear.eu](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHjF_9iTpepLvxqYM6ThM87hVqLxnow2GYGps6fWH41aHiV8koY5MMS6tjxXZ7KhsnPjZSGfPbLoddtbvYieg29nvKWCQZdwydHc-HWx5UFdd6KnhU663AYBDu1cXYzB5Jc2tM-gzSe3gAaK9vxzlJFsR_D8O0zWFxx-e8cTmjFr0lODow_JdRzdehkOvZq-MQxpp3o)
16. [huggingface.co](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFlUy-eM0in1ewsmiiZXQxPcpvAl2wAqK14r43LcMCDxK030lN16ldLXpv_jHd_FEo0vAbEuXZLVq6Ta0wXMRqAIB7OwlTQFNeEC69tf2e7rXLci7ChZBjCSezeLPEf)
17. [note.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEWbRLKm2NPS-iUJ3FNrd1K9Hbn7HRcU0KxB9QcwTJk5jX7TEmEEmn8QMnMI0ieVsR_PN8pW_-ojTXRsLehk6DEXegtRAf2_ef5D2dl7RF69_dj2dLoDQqdVHZ9xEnNOg==)
18. [zenn.dev](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEhNwfko9QCFbvhp4uNx_KkPL-XMDS7nuPwEeB_BRP-ZZnvWkPqBGAeHNjrbA8bk-AQBwdcF1lonI--Lx_bZQJVNvMU2Ds2IBg5bcyRbLZ8u1tZPCiUEPf9nBxlByCdsU-xbrpZ)
19. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGk4KyjADRuqCYttrSGlF_SkmBzVA-FvbKS2aEhW2uhCJja93wWaEOQmbQ-Iqa5qzuQqlnSW0SUQ-6wi-tH4FAnuYcE7G95D9_FV5EGn8fF2vh5tRvUiHbSL-w-6QVGLSaXt9eihj8wl8pZMjF0)
20. [dir.co.jp](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH0BDP7VQkm-hT4aO2lk70TbC4qxRxJE0d4E7uJqGk35lA6ZCbkiqb8uL5FRBbL0yN0mLgjHIZfiAbNUX8QnLVrxRsOSOfsulk1FpwaAi-yIIdjo9SOFVw12Dq2vDGFiFWIsP42n2XAAU4Ibuph5W6Z5BoUszRSjMeCemCkvdc=)
21. [medium.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG2q6m1hDVS8eEWbHCIKf35jmKcuI4He3RApaX4QJUe4Q1drKXaONrKhVnb8NYUvrkwWpPwZxuYJUj29CCRgCRXfPZFdM6MZx-eaVDhghmlOLAjx59JvK4sjXWHfmLrZbuOYhfU9eRiUfGz6URVZens8BL9Qg5m_HWc2a0-vwIYIrAGRUwmZU2WbSFOD5GeyXavbv-YkhW3)
22. [github.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHFPL6a5-Rq2MjLH31NqDM0L3Ugku4f4oyxgP2mMcZhMPi8uaMK8UZwjStnViOX8P-ycr1dB9GNxXBh2KTM2yODhK50NDblVAJeDuOkjTlEndou7_gpQ5dGRAMxW3LdAlf9rYvpfY2VmXL8ZA==)
23. [researchgate.net](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHAQeSVl6EyKEkU-gZ5TVLGKoGeAQ0cufKfoWNpFBsHYjQQIOtK2PbtLK_in_U8re-Cw-o4awezXJyAIzJwSIKckaXcfWRdbyWuYsBGENUls5IuHckTOFmgJnrl2IR-FLEiJ7xn5SsIWbraHAD0RE0IEZGkxTiVCzdF28kPXsZ16srFbMRZsDPGdOYSUCe3R_zHMO9sZ-Ur59FUaNs8yzhj5EE1z7cMNeOU3Y2u5HymNLwWr5gwYWc1T2P9DYc0UsWEZVWCww==)
24. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH6im7iBuIQfltpfdBhBpVHfiu87Q8Ij2_B9bnT5lTTSiw3QxkGYeOy1Q_n5HSyghzcLBVlSstAH-SbZaG8jH79_3m9p5sPH3ozQR1wQMticp4v-FStVUhkI39fhWT3YSmFbyUouzVNuJLQOUhI)
25. [note.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFp1UpPBeApFooQ2D5eb5z_oR9XieeB77D54MjPls02dNmCZ6-4zgbSbdUuYltDu73bCJjhjw5vowlI3n1kuK76Ut2XpGVpXAuNpHAGL6dbgb-Z9P4VGABgObXRnHZANrE0YJ8=)
26. [ibm.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGw75lHSU6XJn5h8aA6zCuTF3aO-d5-O9LUFO9k9iBPVL-74Yz0LOE4vk77vATXkq3sKbwfHx2iVwKrCmWl6W80hAQY4wkY9p8tNwx_8rti_4PVjFSWjVHd7-NpHzz08sJEsh8=)
27. [medium.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHUwFim6QGqS9s9ikxmbbm3H2G6Vldi7r8hwXNHaBmh7w_qI9ELsXbxGzK_thDxfbE5sWRVdoPb9eStaLkf28vOuosVmSyeT_kIhqWBdeyVKm18W8KsEdFEAnHNDIzdT3Ae2Wlxlin802CUuzIsjjzWTFzAIXtAOtdqixWCR_DlaxID-WnTDnYSjcEdV8CI08cVaFZdUkiTG6ouLYWyc1E=)
28. [nixtla.io](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGOoa70BygGK-VS3Ws23yR4ZXkO3KU8UX-qWcyaaoCU2myP3FMz-oEs28h5vYWZw1w-pGZGnC7zuCLgqkJ4RH6HtFH5oUlp-Km9mLjolS9NPLps8sKONZyBv6PLGmEBFw_F_qc0CGSyqHgGv4O5uZBZOUjq47blew==)
29. [nixtla.io](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFrmhymREWSMc_qilUFrt0FvSKDoWdVKyUsETqMNkxBOmrXvMlCOZzUMHjrNsO3YG0Fl5Rkllpycc2PdnlqxHRxT0-eoTy-3bXlkSvYlot7tRXxCxQqt0IyCoZpXf1y)
30. [reddit.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEcpvui9w5_JU5uy_BlqPlw3xTcH6X5VuWmqu6J0Idt5QdXNDRMYwMNAPr3yR_eGqq5NsOtz7xBK2HKLxrjIDHTqK9WPjugRy0S3c8NHFZCkW8w-QcUEPZwYyBNu-lmhdLb5ry16aVmlG-uyEApcpYSnTkxuOV_K_TaEaibraITkA4AUSNk7t52-bURQdAnnkjRFFmyEFKE75XF7qvyqNXw)
31. [note.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGV6_LTmqcOHl0AmcpKnOIpGSP5bmRl9fjbBpuoeuzni3VRzQ6lorPRYqTXkv1dDjLhffA24UboJaEHMhjg82lcQxaIiLOJ8wpohkRowDW3mDl9WFCCrJVgbt5vlyHih-Hw)
32. [hackernoon.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEgAq-z6zCYUBP7I2iQP7YQW3xWDicTMyd_MDD9G7qQIuWDAejeAe5aj9BlN2mjYUNvMB2wUG7VBulRvJV9GtabtcA-ORii2qBMIznlXIjcyKHycRysOWQ2l_oZotaG_vnEkQ6DJ15t7PDKmdf72xsuuJ6tR3FRgS4ZWmR-JE9imp5KB-fCbjh9gCfws4on5UCp17Ln8yxgfXon7HK-6CMvga8tUisPxsAp35X6bOaNy656lNy__WmRwvMchi43Z69afYmjv1ChZRyn5uxnYICc7ZYodR8RjgIFF_sYKZBUUj3-EDkCnUjQEc-C1x1T1_trcDveJg3qsqBJuChyOc42Ajw2NyWYPx9AAcGoWvjKyyPBKqba0HouijcoAunI0Q==)
33. [huggingface.co](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEt1ICwEu3JOpUtxfByIelrjE6QJypgbKFR1e2-GBGDLngylgJNoBRSehowpTc5I_9VBlTw1cr8VjakE8cmJ3lncutUPsLMuNMI-qf6LbLTA6t8JvGVu3pxu3gbdljYWH_8VQlt)
34. [medium.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEhS4qepcXCOz-ymVyWzQLin7GFBgVKKLHRmDOzpGeBip-z61qKBekcm6LCyz09j7MnalNyH2QQgX1hKhI_cBiVBZHkMEAnCHqgxxBihzg6nO4i0cWTT1qfOjbXA7pGYFPLNd9jlCuczCQwcJB36eMwqOzhp5fQSR486Ai9UcB0KFIi4QZNyLxU6FosDn3KbxmG8hZXAfWFCifu1FLfL_l2L2NS-CRSBA==)
35. [andlukyane.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGqP-DxFemlYtz0aU101_DKwzdqCCKurvUGvTbXaHAV1mcDcSAkAb-hk3jftiUmYkCcI8x7mgJX3RoQ2tSrkaK3Jd22808HHYOQaozsor4aX1vehg2Go6BLpcx-0_AIC4f9nykiGha2)
36. [irina-lab.ai](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEfPRlL4w6-DnWYaiyZLQLGb-0BqEV4Si0fvbB86BK6SPfSRZ5GrDqg6WiuKcJVncT_FE9ztT1b5ERIcWDy8NTI6E94XLJV-EluZOviEHL9y9t6Pw8_pI7DUaoikZ8=)
37. [wandb.ai](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEasPKCWcFfvprgVcivf2PnP-nsO9qRTMHaLVEqRlrH3n3gnPjmoW3fqjSy4tIQ8PobEgu-nbYnbG1WpkDRbAMmnyeDPVqDm_i7fPR26K354vGybEZ7IEstlxjdKAaljPEXJn-u99MHNSRg__Y8cU752w9T6TJsqCQtKc59NityDJhDSGi7dMIN4JaNKKpkZHwAVYIvmVaZgCbpZnXlUyy7o-0H5kqU68Teb5GLKD0bYKLtDmLyLdlWwp3F)
38. [splunk.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHA5ugy6QG5OUWl8tXFkD7ciyhXq_Q-4LRNn1UrTGLFJAsGwXpfIoeaQQW8fEUo-7BI0D-OxLgsbPhm3XpNzUsRwysjGNIsZ4JqCQJyVUwGaElpj-DP7ERQGXJsRW6i1M1eQt_pzWKqaGI0qbbV15R3M4HnhOHpPVA5sTE93sUJNODFLRPMcaF__rSta8vCg_H7SllLH4NS6V-kt7E=)
39. [reddit.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGhUL18dfTcCBobjUcxOxcZeRGyQlXmhcLVkYCmNfktdPca_RY9uay0eRAfviZGTMKKx0cHGVbXHWbFtrsVIH5T-uWiXZATQAYSXfOBOuzAvxpwAAQ_llraLXFth46hviW5ifwWfaOsYNhCn1V_6ODSDfjb57bXU2xwWtNcmo4se3vTm6uiMBd7MpCPhqFef4VK5sIhD7mDZ_IiPi2mwxfRXA==)
40. [medium.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGbvS493rJVcy3_ElNwvSNc51Hpjua1AtRHY7trjQfjCZZoleC7OLJIBrW8US-07jfiVHCJQ4Tqff-fwmv-NcwaZ3Qd571JPJEeAuhvMescZNMjBfHin4K4_FnRpA7i33Sd39YBlBOWKlGku_Nhq1UGtZCRCs720GBvZUcu4pdIool-9ALejy76mpvKXvd9)
41. [serverworks.co.jp](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFzZpx-sLjDXxB1xzid-V2VbtgAzoDR3lWLxb8skJUUxzumURXfZaeDupnvW8bO3FuQkYIsjWqJW7hm2j1qpaT5uOIeBU2Q4ZubmfOlpPbAj7bG5Mzk4PwqWJCXWKIsD1MuhA==)
42. [bloomberg.co.jp](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHwVINc48339OxVmvLRtV4MNLn60uNwM95bMx7gyosVAGM2EHQEpCSy2X7IHtSpdcjR0iJYsLWKjeulGyRpmc1fXj1Errr-96KJQJk9crFum-ez7ZQcoUJBTf6gzgGlobEvL9pHpZNSIdXD4_5QyvNaxVo1hJ6QE1x6BTLUkRl5cSH2NDjAr9vtaZVSs7WFar3qNG_M)
43. [industrial-data-science.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGNaStVm5gCYxFKhO_fMG9hsDwYZkZaDDS8_bUbSdwOdDSmIb-vQOdukSt1LoQhAvDR2-73T0BKot5uN6_eupCi7PbYH22qTWu_tqCVQqBfG9-RX68AkCvMoQNq1z1WTAN8VUkpKuDwWAP6utgmr26mQfJ7DNKbBewy37ngnnF1SdYBbWSJItKd3mWMfOqkBDNzm4awvOp9hIww0qmLjBqj0Up3pa7fy2nxV7VJlz4DykME58-wePqA1nSvqUUdgDen62oGnnPPTAivPtacYn19I6fQFRzDvAKChRggxWbaLdwDKWZAsurr6PB2TXSaQvtBzzhVfSIEDb6UIRv1NDu8Z9h_oTy5QYNOGvPMcskrMJy08FcYt3Iu4SZXNdsb6wrvLZUSur1Z25aUxldO6GvY)
44. [hamaruki.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGTks3hpE04upnuzmPLIBtjslBfU-6zCffotE4XH_dRMOPR4M9LzjnLGZ6VNRdP-yms4eH5N6QMYbzLJYOfAr_PJ4otJFJGl3v032X6mzo5-ZvtfzKDknaRa_IjIU8X1hYYVNZaBgZFUo8woN4svkjcrrc9i7JJaoS8LccAaN0wcQDO-lOfx7E_6Nr9p5sgWmP0)
45. [fiby.ai](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEDvwsG-qeZsfICNZEXiwBIU6IGOBy8p_rfuGdp96rq89jXBfVuZ14uUsq_T4cFXTE2X-4V5Ub8HQdDb4xMCStlw35faCuKFnMfJDngr_Vl)
46. [hatenablog.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH_4vJPG9PUv4Tx8piGOZCfP0myIwqAqIqcKBv3yNCojMlQBlzpjWg1uDPfID9xNmWXQptj4eoM11ipYn7NQp5mGRzHHjAFqa-8YBl_KjdhM2e4vzl-XB9LcZUKfL-mH0CdUViLra40BsG1jKs5q1LAsnM=)
47. [ibm.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFOMvAg6da-c_QCoe3aTkxZZ1jie_nVCz103Qr6xeyw8ZbH-OX6JCjdf5SUYtwDAC5L-rsvqQwCrCVXyWpfhA9Ymz9TSnogJk1o_oT2lhCLpEXcZBjjLKbA8qGbxibxD6JnPLmqs7Xz3nPnmLMJN-n9Dp2VBK47dDw-fw==)


---

## Source Citations

1. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQF0EsoRZU9Bjvp8QbVwwTAKjZSAuQxUxXGdd9RUtFviU43A5NekdxkQ69EHVV-2MCiZ_64F-VGvVzZcUOTYuAA8LfyzHCx7zLKob_JZU8EwRVsNxqi_3iAC3lrLRHpVFEfLw-gtBWFQbieWNK7wlNUdTCArLeGMTxbdHE3i4lDrDqUKcxB7DXZQl6TyXmDJvZVY](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQF0EsoRZU9Bjvp8QbVwwTAKjZSAuQxUxXGdd9RUtFviU43A5NekdxkQ69EHVV-2MCiZ_64F-VGvVzZcUOTYuAA8LfyzHCx7zLKob_JZU8EwRVsNxqi_3iAC3lrLRHpVFEfLw-gtBWFQbieWNK7wlNUdTCArLeGMTxbdHE3i4lDrDqUKcxB7DXZQl6TyXmDJvZVY)

2. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFXwIsFH8jGCxQZs-fqlOIoLyZyF1fasFRpOpd0Btn3dDv4OIFWa29YmIgFXEci_5lorXa8IICW74J05w1dATdPSRYOTThvqr8pm9yGey3FdcsDKyp5EYqshtbnFXYk-IHVjQBBLifWIlYHet6EbSlWXc4vqmecDDvwEAgBczf5ViDGYPA=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFXwIsFH8jGCxQZs-fqlOIoLyZyF1fasFRpOpd0Btn3dDv4OIFWa29YmIgFXEci_5lorXa8IICW74J05w1dATdPSRYOTThvqr8pm9yGey3FdcsDKyp5EYqshtbnFXYk-IHVjQBBLifWIlYHet6EbSlWXc4vqmecDDvwEAgBczf5ViDGYPA=)

3. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG-xunPqIQYFKqYPZyGEM3P4rA3NjcfniJpNrbZJMN7ja7DkP0qOJsQYpQxcS6mF5riWltW0ltpPBnvNCx_vtFWy06a47Bz9weAn6bpKdU_WwiS09jU2KsmmDljsd36Wh1mc0Cpe947WNS3ROu5rK22uHaIrnxhPNjzFGdwLE1bovASK56KdiugxfFGYr7KZQvgncHNwQ==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG-xunPqIQYFKqYPZyGEM3P4rA3NjcfniJpNrbZJMN7ja7DkP0qOJsQYpQxcS6mF5riWltW0ltpPBnvNCx_vtFWy06a47Bz9weAn6bpKdU_WwiS09jU2KsmmDljsd36Wh1mc0Cpe947WNS3ROu5rK22uHaIrnxhPNjzFGdwLE1bovASK56KdiugxfFGYr7KZQvgncHNwQ==)

4. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEuw16U0pijgrr9p0rk768ESHPyzMatE8C_dxTTypfEn_TmojG7HsPcmK1WSxOuueqludvmVSst019Pj3swlUC0qBUbQj1dFt2ZMTAP9keTr8Gil_NFGie78VFXKKE8S-fS-Xmcgd1ZF2w6IPdSoOxEqF1RYvF01YgtWZaLXao=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEuw16U0pijgrr9p0rk768ESHPyzMatE8C_dxTTypfEn_TmojG7HsPcmK1WSxOuueqludvmVSst019Pj3swlUC0qBUbQj1dFt2ZMTAP9keTr8Gil_NFGie78VFXKKE8S-fS-Xmcgd1ZF2w6IPdSoOxEqF1RYvF01YgtWZaLXao=)

5. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHcwN25XDHofoHa1H4L3RdvoTsLaiSsnBZ8v_q1pYxrhcYmn1Ymkn59U5aNFuFEekONcaMRyg7AQpWQoNpHS-hTNZ-CLGmRgRddzdxT7mYO6kMnpan5UwCGe7LLz8PWz5Dv0HgttlPiYK5z6XgGwg==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHcwN25XDHofoHa1H4L3RdvoTsLaiSsnBZ8v_q1pYxrhcYmn1Ymkn59U5aNFuFEekONcaMRyg7AQpWQoNpHS-hTNZ-CLGmRgRddzdxT7mYO6kMnpan5UwCGe7LLz8PWz5Dv0HgttlPiYK5z6XgGwg==)

6. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFweqkRb-WM-GjbT6jfJEkpA74pPXUBVmCmyTTSPgrAP2IxC47gnXEs9YSOheS0b_3Y02X55lQaTMmxdkAlSVUtN_u_YP_tmNkIoBxiRCCBJgnP5562VNyMxEnxslzwYxi211JUCW_AkA==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFweqkRb-WM-GjbT6jfJEkpA74pPXUBVmCmyTTSPgrAP2IxC47gnXEs9YSOheS0b_3Y02X55lQaTMmxdkAlSVUtN_u_YP_tmNkIoBxiRCCBJgnP5562VNyMxEnxslzwYxi211JUCW_AkA==)

7. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH4Crkji0miUPkPBoX46wqW8Uz_u8QUv0DXip9gDnl41rspLRyw8kuREsJfFypdM_DPWdX1Ve8ZdynZbSKruKyXqa2W_OpF-VG1W7Obxv6TW3ezFLK669SL8WTqlLEDgl7TrA==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH4Crkji0miUPkPBoX46wqW8Uz_u8QUv0DXip9gDnl41rspLRyw8kuREsJfFypdM_DPWdX1Ve8ZdynZbSKruKyXqa2W_OpF-VG1W7Obxv6TW3ezFLK669SL8WTqlLEDgl7TrA==)

8. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH555QxmxHQt2OKDcFIUJ-ALqEuCUHGIQl_snjT1f4bC5gvLPDMZx_9KAAJNSMXv3r9KcXYCWuErdc6dOTELOLimE8JdPx_dPfVKpR30gR4DLcCm2wX5-FWyg==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH555QxmxHQt2OKDcFIUJ-ALqEuCUHGIQl_snjT1f4bC5gvLPDMZx_9KAAJNSMXv3r9KcXYCWuErdc6dOTELOLimE8JdPx_dPfVKpR30gR4DLcCm2wX5-FWyg==)

9. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQF1kIqA4FEN4wH-857dtIw-x15OwW5FG_L77qBKPIaQuqh7wajELRW4HKI2jfYYwxNw9XVbeSvS_CO3nvQWpOI6ShN9tKOCCZS-Q__ycqvJEWJNN7208arjAMiTYCOWUA8i4vMzwfBvttR02lXLF2WOl-PbwejdSyvoj2bGcK1UOY3rLJoXhQveC6Jn05dJLqubsoz-pmLJtygiUx6WdJ5T9LNpDDWoMprun5osFThOmVbQ_z1b-CpBTtRSzm3tv2G1fnzW-iKD9BIdqUaUDBKyWUaWGg==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQF1kIqA4FEN4wH-857dtIw-x15OwW5FG_L77qBKPIaQuqh7wajELRW4HKI2jfYYwxNw9XVbeSvS_CO3nvQWpOI6ShN9tKOCCZS-Q__ycqvJEWJNN7208arjAMiTYCOWUA8i4vMzwfBvttR02lXLF2WOl-PbwejdSyvoj2bGcK1UOY3rLJoXhQveC6Jn05dJLqubsoz-pmLJtygiUx6WdJ5T9LNpDDWoMprun5osFThOmVbQ_z1b-CpBTtRSzm3tv2G1fnzW-iKD9BIdqUaUDBKyWUaWGg==)

10. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH9jH6qWzII45AdgEXMulBNgyg2mQh5O-4MXRrSXdD80NFBK4uZMqcbi6X67fET00PLkttZP3AIIhiSg_b06SxSJFiHxZxcuipOgcpfXFwlTM0l2LrBVszseg==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH9jH6qWzII45AdgEXMulBNgyg2mQh5O-4MXRrSXdD80NFBK4uZMqcbi6X67fET00PLkttZP3AIIhiSg_b06SxSJFiHxZxcuipOgcpfXFwlTM0l2LrBVszseg==)

11. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGAAzqNBQhAXoQTiSyShxhB8pNZXKu8laiNfnk3gwShEyn5PNmwl8EEC0g3fwGlywtJgRvMTEZz7Jem2dCMkMsytNRy3QedrSDX6QUvYL1_mQE7lKWcA66q6tN5tBvzwSeHB4HbSeIs1PPTvn6yBvVjowhp6vSm3TFcTaIBq3g8Ip6AghLr1O8h7Ty7qrBZGvFxV0zW-vNPbeKpmM-EZv21811_IQSsKVpX0Uhldf0m6Nfvv0E=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGAAzqNBQhAXoQTiSyShxhB8pNZXKu8laiNfnk3gwShEyn5PNmwl8EEC0g3fwGlywtJgRvMTEZz7Jem2dCMkMsytNRy3QedrSDX6QUvYL1_mQE7lKWcA66q6tN5tBvzwSeHB4HbSeIs1PPTvn6yBvVjowhp6vSm3TFcTaIBq3g8Ip6AghLr1O8h7Ty7qrBZGvFxV0zW-vNPbeKpmM-EZv21811_IQSsKVpX0Uhldf0m6Nfvv0E=)

12. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGAVRMpPHVh7DMbGG2SwGmN_a_6V8OhlM0NoK8UIFIN2gTFpfVM-ftYn3dw3CNgpiuhGIb6WxLnlJ_3_nT7ONWdPtcYikCZT_QEDzVkMbTUUbTe4rDoMkZVlCH8m0d2lLYaQVBGx2bOPB54dR0i19HyukP-SrKcsJGIcauEFWkGhJ_ApShvriZGAbA27eTZNEZbrpdvEpRoz8rubTV5LCVSz4H-GGebeCr6kDrxsKOi2uavsN9i4RMAS2hxNdX_lhT6Qee22ZeKV3UrySkCwYpq0YRWGw==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGAVRMpPHVh7DMbGG2SwGmN_a_6V8OhlM0NoK8UIFIN2gTFpfVM-ftYn3dw3CNgpiuhGIb6WxLnlJ_3_nT7ONWdPtcYikCZT_QEDzVkMbTUUbTe4rDoMkZVlCH8m0d2lLYaQVBGx2bOPB54dR0i19HyukP-SrKcsJGIcauEFWkGhJ_ApShvriZGAbA27eTZNEZbrpdvEpRoz8rubTV5LCVSz4H-GGebeCr6kDrxsKOi2uavsN9i4RMAS2hxNdX_lhT6Qee22ZeKV3UrySkCwYpq0YRWGw==)

13. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGNiZNg__ar4OT2bRvoBbTiHEnxMRddwA6osVSqHnsKkjI1GI8GBRde8nShw23doSbaHHaIbz5g0LfuRcNMJoFkuHgaAqtNoGJddl-KxGwv4voxSeDw1VvbbfMg9te7FLG_Pci5LkAnHntImQ4WDeDFoFZpf2vOKGEkCY64kDsNWGMQxOBhEnTp8k9l55kMSE6QB_c8mBvCbsPnfN-GTEtj2JXWwuM=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGNiZNg__ar4OT2bRvoBbTiHEnxMRddwA6osVSqHnsKkjI1GI8GBRde8nShw23doSbaHHaIbz5g0LfuRcNMJoFkuHgaAqtNoGJddl-KxGwv4voxSeDw1VvbbfMg9te7FLG_Pci5LkAnHntImQ4WDeDFoFZpf2vOKGEkCY64kDsNWGMQxOBhEnTp8k9l55kMSE6QB_c8mBvCbsPnfN-GTEtj2JXWwuM=)

14. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGuLXZhy9094RXhSZ_GkEP5E-8ezCK9RyrcRqq2RncBHDTwIYFz5fhtubsXX2f33r3F76lJeh-Bsi7XxTB0ZqhsoN7QCzve2GdtQXAZF1OnIC2eZK6fWgU1VA8BeX5ke1Oxw2zq4LJ_wBPub_qpljytKbwtNRn3raffEKoO0SSkjGAfLNqBk4pX1tT068hhv6rShqzO6QGsuqaDcH7TOxKWDLQ-X_gTLov-x9Cop3Y=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGuLXZhy9094RXhSZ_GkEP5E-8ezCK9RyrcRqq2RncBHDTwIYFz5fhtubsXX2f33r3F76lJeh-Bsi7XxTB0ZqhsoN7QCzve2GdtQXAZF1OnIC2eZK6fWgU1VA8BeX5ke1Oxw2zq4LJ_wBPub_qpljytKbwtNRn3raffEKoO0SSkjGAfLNqBk4pX1tT068hhv6rShqzO6QGsuqaDcH7TOxKWDLQ-X_gTLov-x9Cop3Y=)

15. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFlUy-eM0in1ewsmiiZXQxPcpvAl2wAqK14r43LcMCDxK030lN16ldLXpv_jHd_FEo0vAbEuXZLVq6Ta0wXMRqAIB7OwlTQFNeEC69tf2e7rXLci7ChZBjCSezeLPEf](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFlUy-eM0in1ewsmiiZXQxPcpvAl2wAqK14r43LcMCDxK030lN16ldLXpv_jHd_FEo0vAbEuXZLVq6Ta0wXMRqAIB7OwlTQFNeEC69tf2e7rXLci7ChZBjCSezeLPEf)

16. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHjF_9iTpepLvxqYM6ThM87hVqLxnow2GYGps6fWH41aHiV8koY5MMS6tjxXZ7KhsnPjZSGfPbLoddtbvYieg29nvKWCQZdwydHc-HWx5UFdd6KnhU663AYBDu1cXYzB5Jc2tM-gzSe3gAaK9vxzlJFsR_D8O0zWFxx-e8cTmjFr0lODow_JdRzdehkOvZq-MQxpp3o](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHjF_9iTpepLvxqYM6ThM87hVqLxnow2GYGps6fWH41aHiV8koY5MMS6tjxXZ7KhsnPjZSGfPbLoddtbvYieg29nvKWCQZdwydHc-HWx5UFdd6KnhU663AYBDu1cXYzB5Jc2tM-gzSe3gAaK9vxzlJFsR_D8O0zWFxx-e8cTmjFr0lODow_JdRzdehkOvZq-MQxpp3o)

17. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEWbRLKm2NPS-iUJ3FNrd1K9Hbn7HRcU0KxB9QcwTJk5jX7TEmEEmn8QMnMI0ieVsR_PN8pW_-ojTXRsLehk6DEXegtRAf2_ef5D2dl7RF69_dj2dLoDQqdVHZ9xEnNOg==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEWbRLKm2NPS-iUJ3FNrd1K9Hbn7HRcU0KxB9QcwTJk5jX7TEmEEmn8QMnMI0ieVsR_PN8pW_-ojTXRsLehk6DEXegtRAf2_ef5D2dl7RF69_dj2dLoDQqdVHZ9xEnNOg==)

18. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEhNwfko9QCFbvhp4uNx_KkPL-XMDS7nuPwEeB_BRP-ZZnvWkPqBGAeHNjrbA8bk-AQBwdcF1lonI--Lx_bZQJVNvMU2Ds2IBg5bcyRbLZ8u1tZPCiUEPf9nBxlByCdsU-xbrpZ](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEhNwfko9QCFbvhp4uNx_KkPL-XMDS7nuPwEeB_BRP-ZZnvWkPqBGAeHNjrbA8bk-AQBwdcF1lonI--Lx_bZQJVNvMU2Ds2IBg5bcyRbLZ8u1tZPCiUEPf9nBxlByCdsU-xbrpZ)

19. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGk4KyjADRuqCYttrSGlF_SkmBzVA-FvbKS2aEhW2uhCJja93wWaEOQmbQ-Iqa5qzuQqlnSW0SUQ-6wi-tH4FAnuYcE7G95D9_FV5EGn8fF2vh5tRvUiHbSL-w-6QVGLSaXt9eihj8wl8pZMjF0](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGk4KyjADRuqCYttrSGlF_SkmBzVA-FvbKS2aEhW2uhCJja93wWaEOQmbQ-Iqa5qzuQqlnSW0SUQ-6wi-tH4FAnuYcE7G95D9_FV5EGn8fF2vh5tRvUiHbSL-w-6QVGLSaXt9eihj8wl8pZMjF0)

20. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH0BDP7VQkm-hT4aO2lk70TbC4qxRxJE0d4E7uJqGk35lA6ZCbkiqb8uL5FRBbL0yN0mLgjHIZfiAbNUX8QnLVrxRsOSOfsulk1FpwaAi-yIIdjo9SOFVw12Dq2vDGFiFWIsP42n2XAAU4Ibuph5W6Z5BoUszRSjMeCemCkvdc=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH0BDP7VQkm-hT4aO2lk70TbC4qxRxJE0d4E7uJqGk35lA6ZCbkiqb8uL5FRBbL0yN0mLgjHIZfiAbNUX8QnLVrxRsOSOfsulk1FpwaAi-yIIdjo9SOFVw12Dq2vDGFiFWIsP42n2XAAU4Ibuph5W6Z5BoUszRSjMeCemCkvdc=)

21. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG2q6m1hDVS8eEWbHCIKf35jmKcuI4He3RApaX4QJUe4Q1drKXaONrKhVnb8NYUvrkwWpPwZxuYJUj29CCRgCRXfPZFdM6MZx-eaVDhghmlOLAjx59JvK4sjXWHfmLrZbuOYhfU9eRiUfGz6URVZens8BL9Qg5m_HWc2a0-vwIYIrAGRUwmZU2WbSFOD5GeyXavbv-YkhW3](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG2q6m1hDVS8eEWbHCIKf35jmKcuI4He3RApaX4QJUe4Q1drKXaONrKhVnb8NYUvrkwWpPwZxuYJUj29CCRgCRXfPZFdM6MZx-eaVDhghmlOLAjx59JvK4sjXWHfmLrZbuOYhfU9eRiUfGz6URVZens8BL9Qg5m_HWc2a0-vwIYIrAGRUwmZU2WbSFOD5GeyXavbv-YkhW3)

22. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHFPL6a5-Rq2MjLH31NqDM0L3Ugku4f4oyxgP2mMcZhMPi8uaMK8UZwjStnViOX8P-ycr1dB9GNxXBh2KTM2yODhK50NDblVAJeDuOkjTlEndou7_gpQ5dGRAMxW3LdAlf9rYvpfY2VmXL8ZA==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHFPL6a5-Rq2MjLH31NqDM0L3Ugku4f4oyxgP2mMcZhMPi8uaMK8UZwjStnViOX8P-ycr1dB9GNxXBh2KTM2yODhK50NDblVAJeDuOkjTlEndou7_gpQ5dGRAMxW3LdAlf9rYvpfY2VmXL8ZA==)

23. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHAQeSVl6EyKEkU-gZ5TVLGKoGeAQ0cufKfoWNpFBsHYjQQIOtK2PbtLK_in_U8re-Cw-o4awezXJyAIzJwSIKckaXcfWRdbyWuYsBGENUls5IuHckTOFmgJnrl2IR-FLEiJ7xn5SsIWbraHAD0RE0IEZGkxTiVCzdF28kPXsZ16srFbMRZsDPGdOYSUCe3R_zHMO9sZ-Ur59FUaNs8yzhj5EE1z7cMNeOU3Y2u5HymNLwWr5gwYWc1T2P9DYc0UsWEZVWCww==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHAQeSVl6EyKEkU-gZ5TVLGKoGeAQ0cufKfoWNpFBsHYjQQIOtK2PbtLK_in_U8re-Cw-o4awezXJyAIzJwSIKckaXcfWRdbyWuYsBGENUls5IuHckTOFmgJnrl2IR-FLEiJ7xn5SsIWbraHAD0RE0IEZGkxTiVCzdF28kPXsZ16srFbMRZsDPGdOYSUCe3R_zHMO9sZ-Ur59FUaNs8yzhj5EE1z7cMNeOU3Y2u5HymNLwWr5gwYWc1T2P9DYc0UsWEZVWCww==)

24. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH6im7iBuIQfltpfdBhBpVHfiu87Q8Ij2_B9bnT5lTTSiw3QxkGYeOy1Q_n5HSyghzcLBVlSstAH-SbZaG8jH79_3m9p5sPH3ozQR1wQMticp4v-FStVUhkI39fhWT3YSmFbyUouzVNuJLQOUhI](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH6im7iBuIQfltpfdBhBpVHfiu87Q8Ij2_B9bnT5lTTSiw3QxkGYeOy1Q_n5HSyghzcLBVlSstAH-SbZaG8jH79_3m9p5sPH3ozQR1wQMticp4v-FStVUhkI39fhWT3YSmFbyUouzVNuJLQOUhI)

25. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFp1UpPBeApFooQ2D5eb5z_oR9XieeB77D54MjPls02dNmCZ6-4zgbSbdUuYltDu73bCJjhjw5vowlI3n1kuK76Ut2XpGVpXAuNpHAGL6dbgb-Z9P4VGABgObXRnHZANrE0YJ8=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFp1UpPBeApFooQ2D5eb5z_oR9XieeB77D54MjPls02dNmCZ6-4zgbSbdUuYltDu73bCJjhjw5vowlI3n1kuK76Ut2XpGVpXAuNpHAGL6dbgb-Z9P4VGABgObXRnHZANrE0YJ8=)

26. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGw75lHSU6XJn5h8aA6zCuTF3aO-d5-O9LUFO9k9iBPVL-74Yz0LOE4vk77vATXkq3sKbwfHx2iVwKrCmWl6W80hAQY4wkY9p8tNwx_8rti_4PVjFSWjVHd7-NpHzz08sJEsh8=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGw75lHSU6XJn5h8aA6zCuTF3aO-d5-O9LUFO9k9iBPVL-74Yz0LOE4vk77vATXkq3sKbwfHx2iVwKrCmWl6W80hAQY4wkY9p8tNwx_8rti_4PVjFSWjVHd7-NpHzz08sJEsh8=)

27. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGOoa70BygGK-VS3Ws23yR4ZXkO3KU8UX-qWcyaaoCU2myP3FMz-oEs28h5vYWZw1w-pGZGnC7zuCLgqkJ4RH6HtFH5oUlp-Km9mLjolS9NPLps8sKONZyBv6PLGmEBFw_F_qc0CGSyqHgGv4O5uZBZOUjq47blew==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGOoa70BygGK-VS3Ws23yR4ZXkO3KU8UX-qWcyaaoCU2myP3FMz-oEs28h5vYWZw1w-pGZGnC7zuCLgqkJ4RH6HtFH5oUlp-Km9mLjolS9NPLps8sKONZyBv6PLGmEBFw_F_qc0CGSyqHgGv4O5uZBZOUjq47blew==)

28. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHUwFim6QGqS9s9ikxmbbm3H2G6Vldi7r8hwXNHaBmh7w_qI9ELsXbxGzK_thDxfbE5sWRVdoPb9eStaLkf28vOuosVmSyeT_kIhqWBdeyVKm18W8KsEdFEAnHNDIzdT3Ae2Wlxlin802CUuzIsjjzWTFzAIXtAOtdqixWCR_DlaxID-WnTDnYSjcEdV8CI08cVaFZdUkiTG6ouLYWyc1E=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHUwFim6QGqS9s9ikxmbbm3H2G6Vldi7r8hwXNHaBmh7w_qI9ELsXbxGzK_thDxfbE5sWRVdoPb9eStaLkf28vOuosVmSyeT_kIhqWBdeyVKm18W8KsEdFEAnHNDIzdT3Ae2Wlxlin802CUuzIsjjzWTFzAIXtAOtdqixWCR_DlaxID-WnTDnYSjcEdV8CI08cVaFZdUkiTG6ouLYWyc1E=)

29. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFrmhymREWSMc_qilUFrt0FvSKDoWdVKyUsETqMNkxBOmrXvMlCOZzUMHjrNsO3YG0Fl5Rkllpycc2PdnlqxHRxT0-eoTy-3bXlkSvYlot7tRXxCxQqt0IyCoZpXf1y](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFrmhymREWSMc_qilUFrt0FvSKDoWdVKyUsETqMNkxBOmrXvMlCOZzUMHjrNsO3YG0Fl5Rkllpycc2PdnlqxHRxT0-eoTy-3bXlkSvYlot7tRXxCxQqt0IyCoZpXf1y)

30. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEcpvui9w5_JU5uy_BlqPlw3xTcH6X5VuWmqu6J0Idt5QdXNDRMYwMNAPr3yR_eGqq5NsOtz7xBK2HKLxrjIDHTqK9WPjugRy0S3c8NHFZCkW8w-QcUEPZwYyBNu-lmhdLb5ry16aVmlG-uyEApcpYSnTkxuOV_K_TaEaibraITkA4AUSNk7t52-bURQdAnnkjRFFmyEFKE75XF7qvyqNXw](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEcpvui9w5_JU5uy_BlqPlw3xTcH6X5VuWmqu6J0Idt5QdXNDRMYwMNAPr3yR_eGqq5NsOtz7xBK2HKLxrjIDHTqK9WPjugRy0S3c8NHFZCkW8w-QcUEPZwYyBNu-lmhdLb5ry16aVmlG-uyEApcpYSnTkxuOV_K_TaEaibraITkA4AUSNk7t52-bURQdAnnkjRFFmyEFKE75XF7qvyqNXw)

31. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGV6_LTmqcOHl0AmcpKnOIpGSP5bmRl9fjbBpuoeuzni3VRzQ6lorPRYqTXkv1dDjLhffA24UboJaEHMhjg82lcQxaIiLOJ8wpohkRowDW3mDl9WFCCrJVgbt5vlyHih-Hw](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGV6_LTmqcOHl0AmcpKnOIpGSP5bmRl9fjbBpuoeuzni3VRzQ6lorPRYqTXkv1dDjLhffA24UboJaEHMhjg82lcQxaIiLOJ8wpohkRowDW3mDl9WFCCrJVgbt5vlyHih-Hw)

32. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEgAq-z6zCYUBP7I2iQP7YQW3xWDicTMyd_MDD9G7qQIuWDAejeAe5aj9BlN2mjYUNvMB2wUG7VBulRvJV9GtabtcA-ORii2qBMIznlXIjcyKHycRysOWQ2l_oZotaG_vnEkQ6DJ15t7PDKmdf72xsuuJ6tR3FRgS4ZWmR-JE9imp5KB-fCbjh9gCfws4on5UCp17Ln8yxgfXon7HK-6CMvga8tUisPxsAp35X6bOaNy656lNy__WmRwvMchi43Z69afYmjv1ChZRyn5uxnYICc7ZYodR8RjgIFF_sYKZBUUj3-EDkCnUjQEc-C1x1T1_trcDveJg3qsqBJuChyOc42Ajw2NyWYPx9AAcGoWvjKyyPBKqba0HouijcoAunI0Q==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEgAq-z6zCYUBP7I2iQP7YQW3xWDicTMyd_MDD9G7qQIuWDAejeAe5aj9BlN2mjYUNvMB2wUG7VBulRvJV9GtabtcA-ORii2qBMIznlXIjcyKHycRysOWQ2l_oZotaG_vnEkQ6DJ15t7PDKmdf72xsuuJ6tR3FRgS4ZWmR-JE9imp5KB-fCbjh9gCfws4on5UCp17Ln8yxgfXon7HK-6CMvga8tUisPxsAp35X6bOaNy656lNy__WmRwvMchi43Z69afYmjv1ChZRyn5uxnYICc7ZYodR8RjgIFF_sYKZBUUj3-EDkCnUjQEc-C1x1T1_trcDveJg3qsqBJuChyOc42Ajw2NyWYPx9AAcGoWvjKyyPBKqba0HouijcoAunI0Q==)

33. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEt1ICwEu3JOpUtxfByIelrjE6QJypgbKFR1e2-GBGDLngylgJNoBRSehowpTc5I_9VBlTw1cr8VjakE8cmJ3lncutUPsLMuNMI-qf6LbLTA6t8JvGVu3pxu3gbdljYWH_8VQlt](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEt1ICwEu3JOpUtxfByIelrjE6QJypgbKFR1e2-GBGDLngylgJNoBRSehowpTc5I_9VBlTw1cr8VjakE8cmJ3lncutUPsLMuNMI-qf6LbLTA6t8JvGVu3pxu3gbdljYWH_8VQlt)

34. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEhS4qepcXCOz-ymVyWzQLin7GFBgVKKLHRmDOzpGeBip-z61qKBekcm6LCyz09j7MnalNyH2QQgX1hKhI_cBiVBZHkMEAnCHqgxxBihzg6nO4i0cWTT1qfOjbXA7pGYFPLNd9jlCuczCQwcJB36eMwqOzhp5fQSR486Ai9UcB0KFIi4QZNyLxU6FosDn3KbxmG8hZXAfWFCifu1FLfL_l2L2NS-CRSBA==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEhS4qepcXCOz-ymVyWzQLin7GFBgVKKLHRmDOzpGeBip-z61qKBekcm6LCyz09j7MnalNyH2QQgX1hKhI_cBiVBZHkMEAnCHqgxxBihzg6nO4i0cWTT1qfOjbXA7pGYFPLNd9jlCuczCQwcJB36eMwqOzhp5fQSR486Ai9UcB0KFIi4QZNyLxU6FosDn3KbxmG8hZXAfWFCifu1FLfL_l2L2NS-CRSBA==)

35. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEfPRlL4w6-DnWYaiyZLQLGb-0BqEV4Si0fvbB86BK6SPfSRZ5GrDqg6WiuKcJVncT_FE9ztT1b5ERIcWDy8NTI6E94XLJV-EluZOviEHL9y9t6Pw8_pI7DUaoikZ8=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEfPRlL4w6-DnWYaiyZLQLGb-0BqEV4Si0fvbB86BK6SPfSRZ5GrDqg6WiuKcJVncT_FE9ztT1b5ERIcWDy8NTI6E94XLJV-EluZOviEHL9y9t6Pw8_pI7DUaoikZ8=)

36. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGqP-DxFemlYtz0aU101_DKwzdqCCKurvUGvTbXaHAV1mcDcSAkAb-hk3jftiUmYkCcI8x7mgJX3RoQ2tSrkaK3Jd22808HHYOQaozsor4aX1vehg2Go6BLpcx-0_AIC4f9nykiGha2](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGqP-DxFemlYtz0aU101_DKwzdqCCKurvUGvTbXaHAV1mcDcSAkAb-hk3jftiUmYkCcI8x7mgJX3RoQ2tSrkaK3Jd22808HHYOQaozsor4aX1vehg2Go6BLpcx-0_AIC4f9nykiGha2)

37. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEasPKCWcFfvprgVcivf2PnP-nsO9qRTMHaLVEqRlrH3n3gnPjmoW3fqjSy4tIQ8PobEgu-nbYnbG1WpkDRbAMmnyeDPVqDm_i7fPR26K354vGybEZ7IEstlxjdKAaljPEXJn-u99MHNSRg__Y8cU752w9T6TJsqCQtKc59NityDJhDSGi7dMIN4JaNKKpkZHwAVYIvmVaZgCbpZnXlUyy7o-0H5kqU68Teb5GLKD0bYKLtDmLyLdlWwp3F](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEasPKCWcFfvprgVcivf2PnP-nsO9qRTMHaLVEqRlrH3n3gnPjmoW3fqjSy4tIQ8PobEgu-nbYnbG1WpkDRbAMmnyeDPVqDm_i7fPR26K354vGybEZ7IEstlxjdKAaljPEXJn-u99MHNSRg__Y8cU752w9T6TJsqCQtKc59NityDJhDSGi7dMIN4JaNKKpkZHwAVYIvmVaZgCbpZnXlUyy7o-0H5kqU68Teb5GLKD0bYKLtDmLyLdlWwp3F)

38. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHA5ugy6QG5OUWl8tXFkD7ciyhXq_Q-4LRNn1UrTGLFJAsGwXpfIoeaQQW8fEUo-7BI0D-OxLgsbPhm3XpNzUsRwysjGNIsZ4JqCQJyVUwGaElpj-DP7ERQGXJsRW6i1M1eQt_pzWKqaGI0qbbV15R3M4HnhOHpPVA5sTE93sUJNODFLRPMcaF__rSta8vCg_H7SllLH4NS6V-kt7E=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHA5ugy6QG5OUWl8tXFkD7ciyhXq_Q-4LRNn1UrTGLFJAsGwXpfIoeaQQW8fEUo-7BI0D-OxLgsbPhm3XpNzUsRwysjGNIsZ4JqCQJyVUwGaElpj-DP7ERQGXJsRW6i1M1eQt_pzWKqaGI0qbbV15R3M4HnhOHpPVA5sTE93sUJNODFLRPMcaF__rSta8vCg_H7SllLH4NS6V-kt7E=)

39. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGhUL18dfTcCBobjUcxOxcZeRGyQlXmhcLVkYCmNfktdPca_RY9uay0eRAfviZGTMKKx0cHGVbXHWbFtrsVIH5T-uWiXZATQAYSXfOBOuzAvxpwAAQ_llraLXFth46hviW5ifwWfaOsYNhCn1V_6ODSDfjb57bXU2xwWtNcmo4se3vTm6uiMBd7MpCPhqFef4VK5sIhD7mDZ_IiPi2mwxfRXA==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGhUL18dfTcCBobjUcxOxcZeRGyQlXmhcLVkYCmNfktdPca_RY9uay0eRAfviZGTMKKx0cHGVbXHWbFtrsVIH5T-uWiXZATQAYSXfOBOuzAvxpwAAQ_llraLXFth46hviW5ifwWfaOsYNhCn1V_6ODSDfjb57bXU2xwWtNcmo4se3vTm6uiMBd7MpCPhqFef4VK5sIhD7mDZ_IiPi2mwxfRXA==)

40. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGbvS493rJVcy3_ElNwvSNc51Hpjua1AtRHY7trjQfjCZZoleC7OLJIBrW8US-07jfiVHCJQ4Tqff-fwmv-NcwaZ3Qd571JPJEeAuhvMescZNMjBfHin4K4_FnRpA7i33Sd39YBlBOWKlGku_Nhq1UGtZCRCs720GBvZUcu4pdIool-9ALejy76mpvKXvd9](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGbvS493rJVcy3_ElNwvSNc51Hpjua1AtRHY7trjQfjCZZoleC7OLJIBrW8US-07jfiVHCJQ4Tqff-fwmv-NcwaZ3Qd571JPJEeAuhvMescZNMjBfHin4K4_FnRpA7i33Sd39YBlBOWKlGku_Nhq1UGtZCRCs720GBvZUcu4pdIool-9ALejy76mpvKXvd9)

41. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFzZpx-sLjDXxB1xzid-V2VbtgAzoDR3lWLxb8skJUUxzumURXfZaeDupnvW8bO3FuQkYIsjWqJW7hm2j1qpaT5uOIeBU2Q4ZubmfOlpPbAj7bG5Mzk4PwqWJCXWKIsD1MuhA==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFzZpx-sLjDXxB1xzid-V2VbtgAzoDR3lWLxb8skJUUxzumURXfZaeDupnvW8bO3FuQkYIsjWqJW7hm2j1qpaT5uOIeBU2Q4ZubmfOlpPbAj7bG5Mzk4PwqWJCXWKIsD1MuhA==)

42. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHwVINc48339OxVmvLRtV4MNLn60uNwM95bMx7gyosVAGM2EHQEpCSy2X7IHtSpdcjR0iJYsLWKjeulGyRpmc1fXj1Errr-96KJQJk9crFum-ez7ZQcoUJBTf6gzgGlobEvL9pHpZNSIdXD4_5QyvNaxVo1hJ6QE1x6BTLUkRl5cSH2NDjAr9vtaZVSs7WFar3qNG_M](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHwVINc48339OxVmvLRtV4MNLn60uNwM95bMx7gyosVAGM2EHQEpCSy2X7IHtSpdcjR0iJYsLWKjeulGyRpmc1fXj1Errr-96KJQJk9crFum-ez7ZQcoUJBTf6gzgGlobEvL9pHpZNSIdXD4_5QyvNaxVo1hJ6QE1x6BTLUkRl5cSH2NDjAr9vtaZVSs7WFar3qNG_M)

43. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGNaStVm5gCYxFKhO_fMG9hsDwYZkZaDDS8_bUbSdwOdDSmIb-vQOdukSt1LoQhAvDR2-73T0BKot5uN6_eupCi7PbYH22qTWu_tqCVQqBfG9-RX68AkCvMoQNq1z1WTAN8VUkpKuDwWAP6utgmr26mQfJ7DNKbBewy37ngnnF1SdYBbWSJItKd3mWMfOqkBDNzm4awvOp9hIww0qmLjBqj0Up3pa7fy2nxV7VJlz4DykME58-wePqA1nSvqUUdgDen62oGnnPPTAivPtacYn19I6fQFRzDvAKChRggxWbaLdwDKWZAsurr6PB2TXSaQvtBzzhVfSIEDb6UIRv1NDu8Z9h_oTy5QYNOGvPMcskrMJy08FcYt3Iu4SZXNdsb6wrvLZUSur1Z25aUxldO6GvY](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGNaStVm5gCYxFKhO_fMG9hsDwYZkZaDDS8_bUbSdwOdDSmIb-vQOdukSt1LoQhAvDR2-73T0BKot5uN6_eupCi7PbYH22qTWu_tqCVQqBfG9-RX68AkCvMoQNq1z1WTAN8VUkpKuDwWAP6utgmr26mQfJ7DNKbBewy37ngnnF1SdYBbWSJItKd3mWMfOqkBDNzm4awvOp9hIww0qmLjBqj0Up3pa7fy2nxV7VJlz4DykME58-wePqA1nSvqUUdgDen62oGnnPPTAivPtacYn19I6fQFRzDvAKChRggxWbaLdwDKWZAsurr6PB2TXSaQvtBzzhVfSIEDb6UIRv1NDu8Z9h_oTy5QYNOGvPMcskrMJy08FcYt3Iu4SZXNdsb6wrvLZUSur1Z25aUxldO6GvY)

44. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGTks3hpE04upnuzmPLIBtjslBfU-6zCffotE4XH_dRMOPR4M9LzjnLGZ6VNRdP-yms4eH5N6QMYbzLJYOfAr_PJ4otJFJGl3v032X6mzo5-ZvtfzKDknaRa_IjIU8X1hYYVNZaBgZFUo8woN4svkjcrrc9i7JJaoS8LccAaN0wcQDO-lOfx7E_6Nr9p5sgWmP0](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGTks3hpE04upnuzmPLIBtjslBfU-6zCffotE4XH_dRMOPR4M9LzjnLGZ6VNRdP-yms4eH5N6QMYbzLJYOfAr_PJ4otJFJGl3v032X6mzo5-ZvtfzKDknaRa_IjIU8X1hYYVNZaBgZFUo8woN4svkjcrrc9i7JJaoS8LccAaN0wcQDO-lOfx7E_6Nr9p5sgWmP0)

45. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEDvwsG-qeZsfICNZEXiwBIU6IGOBy8p_rfuGdp96rq89jXBfVuZ14uUsq_T4cFXTE2X-4V5Ub8HQdDb4xMCStlw35faCuKFnMfJDngr_Vl](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEDvwsG-qeZsfICNZEXiwBIU6IGOBy8p_rfuGdp96rq89jXBfVuZ14uUsq_T4cFXTE2X-4V5Ub8HQdDb4xMCStlw35faCuKFnMfJDngr_Vl)

46. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH_4vJPG9PUv4Tx8piGOZCfP0myIwqAqIqcKBv3yNCojMlQBlzpjWg1uDPfID9xNmWXQptj4eoM11ipYn7NQp5mGRzHHjAFqa-8YBl_KjdhM2e4vzl-XB9LcZUKfL-mH0CdUViLra40BsG1jKs5q1LAsnM=](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH_4vJPG9PUv4Tx8piGOZCfP0myIwqAqIqcKBv3yNCojMlQBlzpjWg1uDPfID9xNmWXQptj4eoM11ipYn7NQp5mGRzHHjAFqa-8YBl_KjdhM2e4vzl-XB9LcZUKfL-mH0CdUViLra40BsG1jKs5q1LAsnM=)

47. **Source**
   - URL: [https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFOMvAg6da-c_QCoe3aTkxZZ1jie_nVCz103Qr6xeyw8ZbH-OX6JCjdf5SUYtwDAC5L-rsvqQwCrCVXyWpfhA9Ymz9TSnogJk1o_oT2lhCLpEXcZBjjLKbA8qGbxibxD6JnPLmqs7Xz3nPnmLMJN-n9Dp2VBK47dDw-fw==](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFOMvAg6da-c_QCoe3aTkxZZ1jie_nVCz103Qr6xeyw8ZbH-OX6JCjdf5SUYtwDAC5L-rsvqQwCrCVXyWpfhA9Ymz9TSnogJk1o_oT2lhCLpEXcZBjjLKbA8qGbxibxD6JnPLmqs7Xz3nPnmLMJN-n9Dp2VBK47dDw-fw==)


---

*This report was generated using Gemini Deep Research API with automatic web search and verified citations.*