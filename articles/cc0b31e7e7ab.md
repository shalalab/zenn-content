---
title: 競技プログラミングにおけるソートアルゴリズムの包括的研究
emoji: 🔀
type: tech
topics:
- algorithm
- sort
- datastructure
published: false
---

本稿では、競技プログラミングならびに計算機科学において極めて重要かつ基礎的な要素である**ソート（Sorting：整列）アルゴリズム**について、包括的かつ詳細に体系化して論じる。

**主要な論点（Key Points）**
*   **ソートの重要性と変遷**：ソートアルゴリズムはアルゴリズム学習の「Hello World」とも称される[cite: 1]。過去にはメモリ制約や言語仕様の限界から「どのソートを実装するか」が問われたが、近年ではC++の`std::sort`等のライブラリが充実しており、競技プログラミングにおいては「自作するか」よりも「どの場面でどう活用するか」が重視される傾向にある[cite: 1, 2]。
*   **計算量のブレイクスルー**：単純なソートアルゴリズム（挿入ソートやバブルソートなど）は時間計算量が \(O(N^2)\) であるが、分割統治法などの高度なアルゴリズム手法を用いることで \(O(N \log N)\) へと劇的に改善される[cite: 1]。さらに、比較ベースではないアルゴリズム（基数ソートなど）を用いることで、特定の条件下では線形時間 \(O(N)\) でのソートも可能となる。
*   **ハイブリッド手法の台頭**：現代の実用的なプログラミング言語（Python、Java、C++など）では、単一の学術的なアルゴリズムではなく、複数の手法を組み合わせたハイブリッドアルゴリズム（TimSortやIntrosortなど）が標準採用されており、現実世界のデータ特性（部分的に整列済みであるなど）に最適化されている[cite: 2, 3, 4]。

ソートアルゴリズムは、二分探索の前提条件、中央値などの統計量の算出、分割統治法の適用、貪欲法の事前処理など、競技プログラミングのあらゆる解法の基盤となる[cite: 1, 5]。また、文字列アルゴリズム（Suffix Array等）の構築など、より高度なデータ構造へと直結する技術でもある[cite: 6]。本稿では、競技プログラミングで使用される主要なソートアルゴリズムについて、その動作原理、計算量、安定性、および実装例を網羅的に解説する。

---

## 1. 主要なアルゴリズムの箇条書きによる列挙

競技プログラミングや一般的な計算機科学において学習・活用される主要なソートアルゴリズムは以下の通りである。

*   **基本的な比較ソート（計算量 \(O(N^2)\) クラス）**
    *   バブルソート（Bubble Sort）[cite: 5, 7]
    *   選択ソート（Selection Sort）[cite: 5, 7]
    *   挿入ソート（Insertion Sort）[cite: 5, 7, 8]
*   **高速な比較ソート（計算量 \(O(N \log N)\) クラス）**
    *   マージソート（Merge Sort）[cite: 1, 5, 7]
    *   クイックソート（Quick Sort）[cite: 5, 7, 8]
    *   ヒープソート（Heap Sort）[cite: 8]
*   **非比較ベースのソート（線形時間 \(O(N)\) クラス）**
    *   カウンティングソート（Counting Sort / 計数ソート）[cite: 1]
    *   基数ソート（Radix Sort）[cite: 1, 8]
    *   バケットソート（Bucket Sort / バケツソート）[cite: 1, 8]
*   **その他・競技プログラミングや標準ライブラリで使用される高度なアルゴリズム**
    *   TimSort（ティムソート：Python/Javaなどの標準）[cite: 9, 10, 11]
    *   Introsort（イントロソート：C++ `std::sort` の内部実装として標準的）[cite: 2]
    *   シェルソート（Shell Sort）[cite: 8]
    *   トポロジカルソート（Topological Sort：グラフ上の順序付け）[cite: 12]
    *   ボゴソート（Bogo Sort：理論・ネタ用アルゴリズム）[cite: 7]

---

## 2. 各アルゴリズムの詳細解説

以下に、列挙した各アルゴリズムの動作原理、時間計算量（最良・平均・最悪）、空間計算量、安定性（Stable/Unstable）、そしてC++を用いた実装例を詳細に記述する。

### 2.1 バブルソート（Bubble Sort）

#### アルゴリズムの説明と動作原理
バブルソートは、配列の隣り合う2つの要素の大小関係を比較し、順序が逆であれば入れ替える（スワップする）という操作を繰り返す極めて単純なアルゴリズムである[cite: 5, 7]。このスワップ操作を配列の末尾から先頭（または先頭から末尾）に向けてスキャンしながら行うと、1回の走査ごとに未整列部分の最大値（または最小値）が正しい位置へと「泡が水面に浮かび上がるように」移動していくため、この名が付けられている。

1.  配列の先頭から、隣接する要素 `A[i]` と `A[i+1]` を比較する。
2.  `A[i] > A[i+1]` （昇順の場合）であれば、両者を入れ替える。
3.  この比較と交換を配列の終端まで行うと、最大要素が配列の末尾に確定する。
4.  未確定の要素がなくなるまで、走査の範囲を1つずつ狭めながら繰り返す。

#### 計算量分析
*   **時間計算量**
    *   **最良（Best）**: \(O(N)\)。配列がすでに整列済みである場合、スワップが1回も発生しなかった時点で走査を打ち切る最適化（フラグ管理）を行うことで、1回の走査で終了できる。
    *   **平均（Average）**: \(O(N^2)\)。要素の初期位置からの移動距離の期待値に基づく。
    *   **最悪（Worst）**: \(O(N^2)\)。配列が逆順に整列されている場合、すべての隣接ペアで交換が発生する。比較回数は \(\sum_{i=1}^{N-1} i = \frac{N(N-1)}{2}\) 回となる。
*   **空間計算量**: \(O(1)\)。要素の入れ替えに必要な一時変数のみを使用するため、追加のメモリをほとんど必要としない。

#### 安定性（Stable/Unstable）
**Stable（安定）**。隣接する等しい要素同士は交換されない（`A[i] > A[i+1]` のみで交換し、`>=` とはしない）ため、相対的な順序は保存される。

#### 競技プログラミングにおけるユースケース
競技プログラミングにおいて純粋なソート目的でバブルソートが使われることは、計算量の観点からほぼない（実行時間制限（TLE）に抵触するため）。しかし、「隣接要素の交換のみで配列を整列させるために必要な最小交換回数（転倒数 / Inversion Count）」を求める問題の概念的基盤として非常に重要である。転倒数の偶奇性（パリティ）に関する問題でも言及される。

#### 実装例（C++）
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

template <typename T>
void bubbleSort(std::vector<T>& arr) {
    int n = arr.size();
    bool swapped;
    for (int i = 0; i < n - 1; i++) {
        swapped = false;
        // 1回のパスごとに末尾からi個は整列済みとなるため走査範囲を狭める
        for (int j = 0; j < n - i - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                std::swap(arr[j], arr[j + 1]);
                swapped = true;
            }
        }
        // スワップが一度も行われなかった場合、既にソート完了
        if (!swapped) break;
    }
}
```

---

### 2.2 選択ソート（Selection Sort）

#### アルゴリズムの説明と動作原理
選択ソートは、未整列の部分から最小値（または最大値）を「選択」し、それを未整列部分の先頭要素と交換することで整列を進めるアルゴリズムである[cite: 5, 7]。

1.  配列全体から最小値を探し、先頭の要素と交換する。
2.  先頭1要素は整列済みとなるため、残りの未整列部分から再度最小値を探し、2番目の要素と交換する。
3.  これを未整列部分がなくなるまで繰り返す。

#### 計算量分析
*   **時間計算量**
    *   **最良（Best）**: \(O(N^2)\)
    *   **平均（Average）**: \(O(N^2)\)
    *   **最悪（Worst）**: \(O(N^2)\)
    *   いかなるデータの初期状態であっても、未整列部分の最小値を探すための走査が省略できないため、比較回数は常に \(\frac{N(N-1)}{2}\) 回となる。ただし、要素の**交換（スワップ）回数**は最大でも \(N-1\) 回に留まる。
*   **空間計算量**: \(O(1)\)。インデックスを保持する変数のみで実行可能であるインプレースソートである。

#### 安定性（Stable/Unstable）
**Unstable（不安定）**。離れた要素同士を交換する操作が含まれるため、値が同じ要素の相対的な順序が入れ替わる可能性がある。例えば `[4a, 4b, 1]` という配列の場合、最初のステップで `4a` と最小値 `1` が交換されて `[1, 4b, 4a]` となり、`4a` と `4b` の順序が逆転する。

#### 競技プログラミングにおけるユースケース
競技プログラミングにおいて、通常の配列ソートで使われることはない。しかし「要素の書き込み（スワップ）コストが、比較コストに対して極めて高い環境」をシミュレートする特殊な問題では、スワップ回数が \(O(N)\) である性質が利用されることがある。

#### 実装例（C++）
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

template <typename T>
void selectionSort(std::vector<T>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; i++) {
        // 未整列部分における最小値のインデックスを探す
        int min_idx = i;
        for (int j = i + 1; j < n; j++) {
            if (arr[j] < arr[min_idx]) {
                min_idx = j;
            }
        }
        // 最小値を未整列部分の先頭に移動する
        if (min_idx != i) {
            std::swap(arr[i], arr[min_idx]);
        }
    }
}
```

---

### 2.3 挿入ソート（Insertion Sort）

#### アルゴリズムの説明と動作原理
挿入ソートは「手に持ったトランプを並べ替える方法」に例えられる手法である[cite: 8]。配列を「整列済みの部分」と「未整列の部分」に分け、未整列部分の先頭要素を取り出し、整列済み部分の適切な位置に「挿入」していく[cite: 7, 8]。

1.  配列の先頭要素を「整列済み」とみなす。
2.  続く要素を取り出し、整列済み部分の要素を後方から順に比較する。
3.  取り出した要素より大きい要素は1つ後ろへズラし、適切な位置を見つけたら挿入する。
4.  これを配列の末尾まで繰り返す。

#### 計算量分析
*   **時間計算量**
    *   **最良（Best）**: \(O(N)\)。データが既に整列されている場合、比較が各要素につき1回で済むため。
    *   **平均（Average）**: \(O(N^2)\)。
    *   **最悪（Worst）**: \(O(N^2)\)。データが逆順に並んでいる場合、各挿入操作において整列済み部分の全要素を比較・移動する必要がある。
*   **空間計算量**: \(O(1)\)。インプレースソートである。

#### 安定性（Stable/Unstable）
**Stable（安定）**[cite: 8]。厳密に自分より大きい要素のみを後ろにずらすため、等しい要素同士の位置関係は変化しない。

#### 競技プログラミングにおけるユースケース
競技プログラミングで単独で利用されることは少ないが、「ある程度整列されたデータに対しては高速に動作する」という性質（Adaptive性）を持つ[cite: 8]。そのため、後述するTimSortや、Introsortにおける小さな部分配列に対する処理（定数倍のオーバーヘッドが少ないため）として、他の高度なアルゴリズムの内部処理として非常に重要な役割を果たす[cite: 11]。

#### 実装例（C++）
```cpp
#include <iostream>
#include <vector>

template <typename T>
void insertionSort(std::vector<T>& arr) {
    int n = arr.size();
    for (int i = 1; i < n; i++) {
        T key = arr[i];
        int j = i - 1;
        
        // keyより大きい要素を1つ後ろにずらす
        while (j >= 0 && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        // 正しい位置にkeyを挿入
        arr[j + 1] = key;
    }
}
```

---

### 2.4 マージソート（Merge Sort）

#### アルゴリズムの説明と動作原理
マージソートは、ジョン・フォン・ノイマンによって考案された、「分割統治法（Divide and Conquer）」に基づくソートアルゴリズムである[cite: 1]。配列を半分ずつに分割していき、これ以上分割できない（要素数1）状態になったら、今度は整列順序を保ちながらそれらを「結合（マージ）」していく[cite: 7]。

1.  **分割（Divide）**: 対象の配列を中央で2つの部分配列に分割する。
2.  **再帰（Recursion）**: 分割された部分配列に対して再帰的にマージソートを適用する。
3.  **結合（Merge）**: 2つの整列済み部分配列の先頭要素を比較し、小さい方を新しい配列に順次格納して1つの整列済み配列を構築する。

#### 計算量分析
*   **時間計算量**
    *   **最良・平均・最悪**: 全て \(O(N \log N)\)[cite: 1, 2]。配列を二分割する操作は \(\log_2 N\) 回の深さを持ち、各階層においてマージ処理にかかる合計コストが \(O(N)\) であるため。漸化式 \(T(n) = 2T(n/2) + O(n)\) はマスター定理により \(O(N \log N)\) となる。
*   **空間計算量**: \(O(N)\)[cite: 2]。マージ処理の際に、部分配列の要素を一時的に保持するための外部配列（テンポラリ配列）を必要とする。

#### 安定性（Stable/Unstable）
**Stable（安定）**[cite: 1, 2]。マージ処理において、左右の部分配列の先頭要素が等しい場合、「左側の部分配列の要素を優先して選ぶ」という規則を設けることで、元の相対順序を完全に保存できる。

#### 競技プログラミングにおけるユースケース
C++の `std::stable_sort` は仕様上アルゴリズムは規定されていないが、事実上ほぼ確実にマージソート（またはその変種）で実装されている[cite: 6]。
また、マージソートの「マージ処理」の途中で、「右側の配列から要素を取り出す際、左側の配列にどれだけ未処理の要素が残っているか」をカウントすることで、配列の**転倒数（Inversion Count / バブルソートの最小交換回数）を \(O(N \log N)\) で求める**ことができる。これは競技プログラミングにおける頻出の典型テクニックである。

#### 実装例（C++）
```cpp
#include <iostream>
#include <vector>

template <typename T>
void merge(std::vector<T>& arr, int left, int mid, int right) {
    int n1 = mid - left + 1;
    int n2 = right - mid;
    
    // 一時配列の作成
    std::vector<T> L(n1), R(n2);
    for (int i = 0; i < n1; i++) L[i] = arr[left + i];
    for (int j = 0; j < n2; j++) R[j] = arr[mid + 1 + j];
    
    int i = 0, j = 0, k = left;
    // マージ処理
    while (i < n1 && j < n2) {
        if (L[i] <= R[j]) { // <= にすることで安定性を保証
            arr[k++] = L[i++];
        } else {
            arr[k++] = R[j++];
        }
    }
    
    // 残った要素のコピー
    while (i < n1) arr[k++] = L[i++];
    while (j < n2) arr[k++] = R[j++];
}

template <typename T>
void mergeSort(std::vector<T>& arr, int left, int right) {
    if (left >= right) return;
    int mid = left + (right - left) / 2;
    mergeSort(arr, left, mid);
    mergeSort(arr, mid + 1, right);
    merge(arr, left, mid, right);
}
```

---

### 2.5 クイックソート（Quick Sort）

#### アルゴリズムの説明と動作原理
クイックソートは、トニー・ホーアによって考案された分割統治法に基づくアルゴリズムであり、実用上極めて高速な比較ベースのソートである[cite: 5, 7, 8]。適当な「基準値（ピボット）」を選び、それより小さい要素を前に、大きい要素を後ろに分配（パーティション）し、分割された領域に対して再帰的に処理を行う[cite: 7]。

1.  配列からピボットとなる要素を1つ選ぶ。
2.  ピボットを基準にして、要素を「ピボット未満」「ピボット以上」の2つのグループに分割する（Partition操作）。
3.  分割された2つの部分配列に対して、それぞれ再帰的にクイックソートを適用する。

#### 計算量分析
*   **時間計算量**
    *   **最良・平均**: \(O(N \log N)\)[cite: 1]。ピボットが毎回理想的に中央値を分割した場合、再帰の深さは \(\log_2 N\) となる。ポインタ移動とスワップのみで構成されるため定数倍係数（Constant Factor）が非常に小さく[cite: 11]、同じ \(O(N \log N)\) のマージソートやヒープソートより実測で高速に動作する。
    *   **最悪**: \(O(N^2)\)。ピボットとして毎回最大値または最小値が選ばれた場合（既にソート済みの配列で先頭要素をピボットにした場合など）、再帰の深さが \(N\) に達する。
*   **空間計算量**: 平均的にはコールスタックの消費が \(O(\log N)\) である。しかし、最悪の場合再帰が深く積まれると \(O(N)\) となり、スタックオーバーフローのリスクがある（末尾再帰最適化などを行わない場合）。

#### 安定性（Stable/Unstable）
**Unstable（不安定）**。パーティション操作において、離れた場所にある要素をピボットの反対側にスワップするため、同じ値の要素の順序関係が保持されない。

#### 競技プログラミングにおけるユースケース
標準的なC++の `std::sort` は、かつて純粋なクイックソートを採用していたが、最悪計算量 \(O(N^2)\) を突かれてハック（意図的に遅い入力ケースを与えられてTLEさせられること）される問題があった。現在ではそれを防ぐために後述の Introsort（クイックソートをベースにヒープソートに切り替える手法）が用いられている[cite: 2]。

#### 実装例（C++）
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

template <typename T>
int partition(std::vector<T>& arr, int low, int high) {
    // ランダムなピボット選択やMedian-of-3を行うことで最悪ケースを回避しやすい
    // ここではLomutoのパーティション法（末尾をピボット）を使用
    T pivot = arr[high];
    int i = low - 1;
    
    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            std::swap(arr[i], arr[j]);
        }
    }
    std::swap(arr[i + 1], arr[high]);
    return i + 1;
}

template <typename T>
void quickSort(std::vector<T>& arr, int low, int high) {
    if (low < high) {
        int pi = partition(arr, low, high);
        quickSort(arr, low, pi - 1);
        quickSort(arr, pi + 1, high);
    }
}
```

---

### 2.6 ヒープソート（Heap Sort）

#### アルゴリズムの説明と動作原理
ヒープソートは、データ構造の「ヒープ（二分ヒープ木）」を用いたソートアルゴリズムである[cite: 1, 8]。優先度付きキュー（Priority Queue）の仕組みを配列上でインプレースに展開したものと見なせる。

1.  対象の配列から「最大ヒープ（親ノードが常に子ノードより大きいという性質を持つ二分木）」を構築する。配列上で木構造を表現するため、インデックス `i` の子は `2i+1` と `2i+2` となる。
2.  最大ヒープの根（つまり最大値、配列の先頭 `A`）を取り出し、ヒープの末尾要素と交換する。これで最大値の位置が確定する。
3.  ヒープのサイズを1減らし、先頭に移動してきた要素を沈める（`heapify`）ことで、再び最大ヒープの条件を満たすように再構成する。
4.  これをヒープが空になるまで繰り返す。

#### 計算量分析
*   **時間計算量**
    *   **最良・平均・最悪**: 全て \(O(N \log N)\)。最初のヒープ構築は \(O(N)\) で行える（ボトムアップ構築）。その後の最大値の取り出しと再構築（深さ \(\log N\)）を \(N-1\) 回行うため、初期状態に関わらず常に \(O(N \log N)\) が保証される。
*   **空間計算量**: \(O(1)\)。木構造は配列上のインデックス計算で論理的に行われるため、追加のメモリ領域を必要としない。

#### 安定性（Stable/Unstable）
**Unstable（不安定）**。最大値を確定させるためにヒープの根と末尾を交換する際、配列内の相対順序が大きく破壊されるため。

#### 競技プログラミングにおけるユースケース
常に \(O(N \log N)\) かつ追加メモリ \(O(1)\) を要求される厳しい条件下で優れている。競技プログラミングでは、ヒープソートそのものを実装するよりも、最悪ケースを避けるためのフォールバックとして `std::sort` の中で暗黙的に呼び出される（Introsort）[cite: 2]か、`std::priority_queue` を用いたダイクストラ法などにヒープの概念が利用される[cite: 12]。

#### 実装例（C++）
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

template <typename T>
void heapify(std::vector<T>& arr, int n, int i) {
    int largest = i;       // 親ノード
    int left = 2 * i + 1;  // 左の子
    int right = 2 * i + 2; // 右の子

    if (left < n && arr[left] > arr[largest])
        largest = left;

    if (right < n && arr[right] > arr[largest])
        largest = right;

    if (largest != i) {
        std::swap(arr[i], arr[largest]);
        heapify(arr, n, largest);
    }
}

template <typename T>
void heapSort(std::vector<T>& arr) {
    int n = arr.size();
    
    // ヒープの構築 (O(N))
    for (int i = n / 2 - 1; i >= 0; i--)
        heapify(arr, n, i);

    // 要素を一つずつ取り出す (O(N log N))
    for (int i = n - 1; i > 0; i--) {
        std::swap(arr, arr[i]);
        heapify(arr, i, 0); // サイズを狭めて再構築
    }
}
```

---

### 2.7 カウンティングソート（Counting Sort / 計数ソート）

#### アルゴリズムの説明と動作原理
カウンティングソートは、要素同士の「大小比較」を行わないソートアルゴリズムである[cite: 1]。代わりに、入力配列に出現する各要素の値の頻度（出現回数）をカウントし、それを基に要素の最終的な出力位置を直接計算する。

1.  対象データが取り得る値の範囲（\(0\) から \(K\)）をカバーする度数分布表（カウント配列）を作成し、ゼロで初期化する。
2.  入力配列を走査し、各要素の出現回数をカウント配列に記録する。
3.  カウント配列の累積和を計算する。この累積和が、ソート後の配列における各要素の（最後の）位置を示す。
4.  入力配列を**後ろから前へ**走査し（安定性を保つため）、累積和配列を参照しながら出力配列に要素を配置し、累積和を1減らす。

#### 計算量分析
*   **時間計算量**
    *   **最良・平均・最悪**: \(O(N + K)\)。ここで \(N\) は要素数、\(K\) は取り得る値の最大値（範囲）。比較を行わないため、値の範囲 \(K\) が \(N\) に対して十分に小さい場合（例：英小文字のみの文字列など）は線形時間 \(O(N)\) となり極めて高速である。
*   **空間計算量**: \(O(N + K)\)。出力用の配列 \(O(N)\) と、カウント用の配列 \(O(K)\) が必要。

#### 安定性（Stable/Unstable）
**Stable（安定）**。入力配列を後ろから前へと走査して出力配列に書き込む際、同一値に対するインデックスも後ろから順に消費されるため、相対順序が維持される。

#### 競技プログラミングにおけるユースケース
各要素が整数値であり、かつ絶対値が極端に大きくない場合に最適である[cite: 1]。特に、アルファベット26文字からなる文字列のソートや、後述する Suffix Array（接尾辞配列）を線形時間で構築するためのSA-ISアルゴリズムの内部ソート（バケツソートの一種としての計数ソート）において必須の技術である[cite: 6]。

#### 実装例（C++）
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

void countingSort(std::vector<int>& arr) {
    if (arr.empty()) return;
    int max_val = *std::max_element(arr.begin(), arr.end());
    int min_val = *std::min_element(arr.begin(), arr.end());
    int range = max_val - min_val + 1;

    std::vector<int> count(range, 0);
    std::vector<int> output(arr.size());

    // 度数のカウント
    for (int i = 0; i < arr.size(); i++) {
        count[arr[i] - min_val]++;
    }

    // 累積和の計算
    for (int i = 1; i < count.size(); i++) {
        count[i] += count[i - 1];
    }

    // 安定性を保つために後ろから走査して配置
    for (int i = arr.size() - 1; i >= 0; i--) {
        output[count[arr[i] - min_val] - 1] = arr[i];
        count[arr[i] - min_val]--;
    }

    arr = output;
}
```

---

### 2.8 基数ソート（Radix Sort）

#### アルゴリズムの説明と動作原理
基数ソートは、計数ソートの「値の範囲 \(K\) が大きいとメモリと時間を大量に消費する」という弱点を克服するためのアルゴリズムである[cite: 1, 8]。桁ごと（基数ごと）にソートを繰り返すことで、巨大な整数や文字列全体をソートする。

1.  最下位桁（LSD: Least Significant Digit）から最上位桁（MSD: Most Significant Digit）へ向かって順に処理を進める（LSD方式）。
2.  各桁に注目し、その桁の値だけをキーとして、要素全体を**安定なソート（カウンティングソートなど）**を用いて整列させる。
3.  最上位桁までこの操作を繰り返すと、全体が完全に整列される。

#### 計算量分析
*   **時間計算量**
    *   **最良・平均・最悪**: \(O(d(N + K))\)。ここで \(d\) は最大桁数（ループ回数）、\(N\) は要素数、\(K\) は基数（例えば10進数なら10、ビットごとなら2または256など）。\(d\) が定数と見なせる場合、全体として線形時間 \(O(N)\) となる。
*   **空間計算量**: \(O(N + K)\)。内部で使用するカウンティングソートの空間計算量に依存する。

#### 安定性（Stable/Unstable）
**Stable（安定）**。内部で使用する桁ごとのソートが安定ソートでなければならない（そうでないと、下位桁でせっかく整列した順序が上位桁の整列時に破壊されてしまう）。

#### 競技プログラミングにおけるユースケース
非常に要素数が多い（\(10^7\) 以上など）で、各要素が整数値の場合、比較ベースソートの \(O(N \log N)\) すらボトルネックになることがある。このような場面において、基数を 256（8ビット） や 65536（16ビット） などに設定して基数ソートを行うと、キャッシュ効率も良く極めて高速に動作する[cite: 1]。これもSuffix Arrayの構築（SA-IS）など高度な文字列アルゴリズムに紐づく[cite: 6]。

#### 実装例（C++）
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

// 特定の桁(exp)に着目したカウンティングソート
void countSortForRadix(std::vector<int>& arr, int exp) {
    int n = arr.size();
    std::vector<int> output(n);
    int count[cite: 7] = {0}; // 基数を10とする場合

    for (int i = 0; i < n; i++) {
        count[(arr[i] / exp) % 10]++;
    }
    for (int i = 1; i < 10; i++) {
        count[i] += count[i - 1];
    }
    for (int i = n - 1; i >= 0; i--) {
        output[count[(arr[i] / exp) % 10] - 1] = arr[i];
        count[(arr[i] / exp) % 10]--;
    }
    for (int i = 0; i < n; i++) {
        arr[i] = output[i];
    }
}

void radixSort(std::vector<int>& arr) {
    if (arr.empty()) return;
    int max_val = *std::max_element(arr.begin(), arr.end());
    // 最下位桁から順にソート (exp は 10^0, 10^1, 10^2...)
    for (int exp = 1; max_val / exp > 0; exp *= 10) {
        countSortForRadix(arr, exp);
    }
}
```

---

### 2.9 バケットソート（Bucket Sort / バケツソート）

#### アルゴリズムの説明と動作原理
バケットソートは、入力データをある規則に基づいて複数の「バケツ（Bucket）」に分散させ、各バケツの中で個別にソートを行った後、それらを順番に結合する手法である[cite: 1, 8]。データの分布が一様（Uniformly distributed）である場合に非常に高いパフォーマンスを発揮する。

1.  データの値の範囲をカバーするように、複数の空のバケツ（リストや配列）を用意する。
2.  入力配列の各要素を、その値に対応する適切なバケツに振り分ける。
3.  バケツごとに要素をソートする（通常は挿入ソートや `std::sort` を使用）。
4.  順番にバケツを巡回し、要素を結合して最終的なソート結果を得る。

#### 計算量分析
*   **時間計算量**
    *   **最良・平均**: \(O(N + K)\)（あるいは \(O(N)\)）。データが一様に分布しており、各バケツに入る要素数が定数個に近い場合、バケツ内のソートは \(O(1)\) で完了し、全体の振り分けと結合に \(O(N + K)\) （\(K\) はバケツ数）しかかからない。
    *   **最悪**: \(O(N^2)\)。すべてのデータが1つのバケツに集中してしまった場合、バケツ内のソートに依存する（挿入ソートを用いた場合は \(O(N^2)\) になる）。
*   **空間計算量**: \(O(N + K)\)。バケツ自体のポインタ/リスト管理と、全要素の格納領域。

#### 安定性（Stable/Unstable）
**Stable（安定）**。要素をバケツに順番に格納し、バケツ内でのソートに安定ソートを用いれば、全体としても安定になる。

#### 競技プログラミングにおけるユースケース
連続な浮動小数点数の分布や、幾何問題（平面上の点を座標ごとにブロック分割して処理するなど）において、ソートの概念を拡張してバケツ（平方分割などのバケット法）として用いることで計算量を改善するテクニックに直結する。

#### 実装例（C++）
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

void bucketSort(std::vector<float>& arr) {
    int n = arr.size();
    if (n <= 0) return;

    // 0.0 ~ 1.0 未満の浮動小数点数を想定
    std::vector<std::vector<float>> buckets(n);

    // 要素を各バケツへ振り分ける
    for (int i = 0; i < n; i++) {
        int bucket_idx = n * arr[i];
        if (bucket_idx >= n) bucket_idx = n - 1; // 境界対応
        buckets[bucket_idx].push_back(arr[i]);
    }

    // 各バケツ内を個別にソート (ここでは標準ライブラリを利用)
    for (int i = 0; i < n; i++) {
        std::sort(buckets[i].begin(), buckets[i].end());
    }

    // 結合
    int index = 0;
    for (int i = 0; i < n; i++) {
        for (float val : buckets[i]) {
            arr[index++] = val;
        }
    }
}
```

---

### 2.10 その他競技プログラミングで使用される高度なソートアルゴリズム

競技プログラミングでは「自作のソートよりも標準ライブラリを使うのが基本」である[cite: 2]。現代の標準ライブラリは、単一のアルゴリズムではなく、上述したアルゴリズムの弱点を補い合うように組み合わせた「ハイブリッドアルゴリズム」を採用している。

#### 2.10.1 TimSort（ティムソート）
**アルゴリズムの概要**：
2002年にTim PetersによってPythonのために開発され、現在ではPython (`list.sort()`, `sorted()`) やJava（Java 7以降のオブジェクトのソート `Arrays.sort()`）、Android、V8（JavaScript）などでデフォルト採用されている極めて強力なハイブリッドアルゴリズムである[cite: 3, 4, 10]。
マージソートと挿入ソートの利点を組み合わせ、現実世界のデータに頻繁に存在する「すでに整列されている部分配列（Runs）」を自動的に検出し、それを活かしてソートを高速化する（適応的/Adaptive）[cite: 3, 4, 10]。

**動作原理**：
1.  **Runs（ラン）の検出**：配列をスキャンし、連続して昇順、あるいは厳密に降順になっている部分列（Run）を見つける[cite: 3, 4]。
2.  降順のRunが見つかった場合は、その場で反転させて昇順にする[cite: 3, 4]。
3.  **minrun（最小ラン長）**：Runの長さが所定の閾値（通常32～64要素。サイズ63以下の配列なら配列長と同じ）に満たない場合、後続の要素を取り込んで**挿入ソート**によって長さを `minrun` まで押し上げる[cite: 3, 4, 11]。挿入ソートは要素数が少ない場合に極めて定数倍が軽く高速（O(N^2)の係数が極小）であるため、この閾値設定がパフォーマンスの鍵となる[cite: 11]。
4.  **マージ**：確保した複数のRunをスタックで管理し、特定のバランス条件（フィボナッチ数列的なサイズ関係）を満たすようにマージソートの要領で結合していく[cite: 3]。
5.  **ギャロッピング（Galloping）と二分探索**：マージ処理中、一方の要素が他方よりも連続して選ばれる場合（偏りがある場合）、片方から1つずつ比較するのをやめ、二分探索によって「どこまで一気に挿入できるか」をまとめて探し出す（マージコストの削減）[cite: 3]。

**計算量と性質**：
*   **時間計算量**: 最良 \(O(N)\)（部分的に整列済みの恩恵を受けるため）[cite: 10]。平均・最悪 \(O(N \log N)\) [cite: 4, 10]。
*   **空間計算量**: \(O(N)\)（マージのためのオーバーヘッド。小さい配列をテンポラリメモリにコピーして結合する）[cite: 3]。
*   **安定性**: **Stable（安定）**[cite: 3, 4, 10]。JavaやPythonにおいて、オブジェクトを比較キーでソートした際に元の出現順序が維持されるのはTimSortの安定性によるものである。

#### 2.10.2 Introsort（イントロソート / C++ std::sortの内部実装）
**アルゴリズムの概要**：
C++の `std::sort` （および `std::ranges::sort` [cite: 13]）の内部実装として広く採用されているアルゴリズムである[cite: 2]。David Musserによって1997年に考案された。

**動作原理**：
クイックソート、ヒープソート、挿入ソートの3つを動的に切り替える。
1.  基本的には高速な**クイックソート**で分割（パーティション）を進める。
2.  しかし、再帰の深さ（スタックレベル）を監視し、一定の閾値（通常は \(2 \log_2 N\)）を超えた場合、クイックソートの最悪ケース \(O(N^2)\) に陥っていると判断し、残りの部分配列の処理を強制的に**ヒープソート**（最悪 \(O(N \log N)\) 保証）に切り替える。
3.  さらに、部分配列の要素数が一定以下（例えば16要素以下）に細分化された場合は、関数呼び出しのオーバーヘッドを避けるため、一括して**挿入ソート**によって仕上げを行う。

**計算量と性質**：
*   **時間計算量**: 最良・平均・最悪すべて \(O(N \log N)\) [cite: 2, 13]。
*   **空間計算量**: \(O(\log N)\)（再帰スタック）。
*   **安定性**: **Unstable（不安定）**。クイックソートとヒープソートをベースにしているため安定ではない。安定性が必要な場合はマージソートベースの `std::stable_sort` を使用する[cite: 2, 6]。

#### 2.10.3 トポロジカルソート（Topological Sort）
競技プログラミングにおいて「ソート」という言葉が指すもう一つの重要な概念が、グラフ理論におけるトポロジカルソートである[cite: 12]。
これは、閉路を持たない有向グラフ（DAG: Directed Acyclic Graph）の頂点を、すべての有向辺が「左から右」に向かうように一直線に並べる順序付けである。カーンのアルゴリズム（Kahn's Algorithm、入次数を用いたBFS）やDFSを用いて \(O(V+E)\) で実装され、タスクの依存関係の解消や、DAG上での動的計画法（DP）の計算順序の決定に不可欠である[cite: 12]。

---

## 3. 競技プログラミングにおける実践的なソートの活用法

競技プログラミングにおいて、上述のアルゴリズムの理論を理解した上で、実際にC++等でどのようにコーディングし、どのような応用問題を解くかを詳述する。

### 3.1 標準ライブラリとC++でのソート
C++では `<algorithm>` ヘッダをインクルードし、イテレータを用いて簡潔にソートを行う（あるいはC++20以降のRangesライブラリを使用する）[cite: 13, 14]。

**配列や `std::vector` の全要素昇順ソート**:
```cpp
std::vector<int> a = {3, 4, 1, 2, 5};
std::sort(a.begin(), a.end()); // 1 2 3 4 5 になる [cite: 14]
```
`std::ranges::sort(a)` を用いると、`begin()` と `end()` を省略でき、より直感的かつ安全な実装が可能である[cite: 13]。時間計算量は共に \(O(N \log N)\) である[cite: 13]。

**降順ソート（`std::greater`の活用）**:
大きい順にソートしたい場合は、第3引数に関数オブジェクト `std::greater<T>()` を渡す。
```cpp
std::sort(a.begin(), a.end(), std::greater<int>()); // 5 4 3 2 1 になる [cite: 14]
// vectorの場合は rbegin(), rend() を用いる手法もある
std::sort(a.rbegin(), a.rend()); [cite: 14]
```

### 3.2 独自の比較関数（カスタムコンパレータ）による柔軟なソート
競技プログラミングでは、単純な数値だけでなく、複数の属性を持つ構造体やペア（`std::pair`）をソートする機会が極めて多い[cite: 15]。`std::sort` は第3引数に「何らかの順序関係 (ordering relation) を前提とする Compare 型のオブジェクト」を取る設計になっている[cite: 15]。

この際、比較関数は**狭義の弱順序（Strict Weak Ordering）**を満たさなければならない。すなわち、`a == b` のときは必ず `false` を返す必要がある（`<=` 等を含めてはならない）。これを破るとSegmentation Faultや無限ループを引き起こす。

**例：`std::pair<int, int>` において、第1要素で昇順（辞書順）、第1要素が等しい場合は第2要素で「降順（逆辞書順）」にソートする病的で複雑なケース**[cite: 15]：
C++11以降ではラムダ式（Lambda expression）を用いてインラインで簡潔に記述する。
```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<std::pair<int, int>> vec = {{1, 2}, {1, 5}, {2, 3}};
    
    // ラムダ式を用いたカスタムソート
    std::sort(vec.begin(), vec.end(), [](const std::pair<int, int>& left, const std::pair<int, int>& right) {
        if (left.first != right.first) {
            return left.first < right.first; // 第1要素は昇順
        }
        return left.second > right.second;   // 第2要素は降順
    });
    // 結果: {1, 5}, {1, 2}, {2, 3} となる
}
```

### 3.3 ソートを用いた典型的なアルゴリズム適用
競技プログラミングでは、ソート自体が目的ではなく、ソートを行うことで他のアルゴリズムを適用できるようにするための前処理として使われる[cite: 1, 5]。

1.  **二分探索（Binary Search）の前提条件**：配列がソートされていることで、特定の要素の検索や「X以下の要素が何個あるか」といったクエリに \(O(\log N)\) で応答可能になる[cite: 5]。中央値を求める問題でも、「中央値がX以下である」という判定を二分探索に帰着させることができる[cite: 5]。
2.  **貪欲法（Greedy Algorithm）の最適化**：最小のコストで最大の利益を得るような問題では、要素をコスト対効果の順にソートし、最良のものから順番に選択していく手法（ソートと貪欲法）が定石である[cite: 7]。
3.  **座標圧縮（Coordinate Compression / 1D-Compression）**：巨大な座標値が与えられた際、その相対的な大小関係のみを保持して小さな整数値に変換するテクニック。要素を複製した上でソートし、重複を削除（`std::unique`）して、元の値がソート済み配列のどこにあるかを二分探索（`std::lower_bound`）することで \(O(N \log N)\) で実現する。
4.  **クラスカル法（Kruskal's Algorithm）**：最小全域木（MST）を構築するグラフアルゴリズム。全辺を重みの昇順にソートし、Union-Find（素集合データ構造）を用いて閉路ができないように辺を追加していく手法[cite: 12]。

---

## 結言

競技プログラミングにおけるソートは、単純な整列処理に留まらず、計算量理論の基礎（\(O(N^2)\) から \(O(N \log N)\)、そして \(O(N)\) へのブレイクスルー）やデータ構造の挙動を深く理解するための登竜門である。
古典的なバブルソートや挿入ソートが転倒数計算やTimSortの内部最適化として息づき、マージソートやクイックソートが大規模データの根幹を支え、基数ソートや計数ソートが特定の条件下で限界を超えた速度を叩き出す。
現代の競技プログラミングにおいては、標準ライブラリ（`std::sort` や `std::stable_sort`）の内部挙動をブラックボックスにせず、それぞれの特性（安定性や最悪計算量）を意識しながら、カスタムコンパレータを駆使し、貪欲法・二分探索・高度な文字列アルゴリズムへと繋げていく運用能力が求められている。

**Sources:**
1. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHMGiCdrbS5t0ax8Ki_ciF2BtOLew1awGSXW8Bv-D8rc0woqb436C0AWB8TaSQorvMl5v8Nn7-so9TYzjz_kVX6P2vwXWZ7EV2Hsg95_AAvVCyTWHGMXTXVRwXUXdKCFRkSDrI1VzJX)
2. [kentei.ai](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEIrGJBW4Kol5FkzhwhvDCNSWSk6zHPk6CRoD0YSyg1ox-j9nMKJG6wPiyK8NVpTzAgSfyng3aEObiSwCmHuBGwGcnIk4M33gpj6LvB_aoRTfDBwTaorIo1Eg==)
3. [wikipedia.org](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQF3TwvDlNF9OHtvwDLqQ4dKhJjuhXMq_6FYVY-DvQ-BBROujxGV6kPVhRZQXKdr6YRMkXqrwKvS0pTvPNnTI5ZWvfyJXa_Z3irEMEI9LULejWhCPFzjlOC9EAA=)
4. [skerritt.blog](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH0YloIKOxNjneMhiY-jCbt9vJvhoX4hEAcgzVebK0n8Sei_dnMQkcttOsxxzEmaQ15tRKNfYvUzQouPbVLh2wiKn8YPah1qfFj06mhxULlafyBtw==)
5. [algo-logic.info](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHv1epUn5zDiRikiYRvmZCJz0s-OGSqOKNDYCtwdh4EHofLGLnXVOGMjvapcftDZqw48L87RhmNIr-zzhe1XB2eE2KDeKGV-cctlKmYgwaGHLIwoenJ5MGTks3RK9w=)
6. [atcoder.jp](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGs5NXT4xdzofzGmrBRIlsGIPm1sWbZE3Hvq4n1tgWmaEY-w_n65fdKrAXsX5h0sp2F8aHhqXnHU4OSCIwCJIddwRXzN4XbNkvZUlw3JroFi7Xi1ULB6YE1CQ0QzGqMidiHAaQZrQJw_infxqgCRi4=)
7. [slideshare.net](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHG2dRVsOAWTeKXDTMJMdy9QcLf4-xtZzhqckjdOd6jMpeUoASywjBKQuDZxEA98PIBnbTxc-0z_0IcWZsE8-Td6ojFPaNpk0ohJ5nJInv4YAiKC1ivCgBBwEV8W8X1TW2eTfgK8R4PT1GbpR876--cLyk3)
8. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFHqkXE_oPYZNr50P1VlUmtQUPS7B3WK_32Zo1KXStNhCR-7ty-DhlxiSD6PuCIincrbThzs_87wN7vl99nE9G5GM3vhxkaGVM2eZLqEBz8iir38xDWpZD8Hxd8dBroTzPdzs9lz8PKhPc=)
9. [reddit.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHvJh5ciwQ-h4nSYOUy2ps8HFHcWaiPYqQg_0eMUPG-qcvgaEPlsfejeAJEL2Lrifc2Diq-lit2L0XZgg_56j8DU-fOdu8z1i6GseP9V3mfpsn5GCMqAzLaYk_xsI2h3bA_ZNKnVxswMbzeBdnKv_FI3bBLc3Eo4NYfs5ArMdTaHnUx7-0Q1hV05y4SUduY104NrtksGOQw2-m1YL5cUH8cVw==)
10. [geeksforgeeks.org](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGr5tgGMSbKwKrr4bAcYsXtk_l2GgUE6YVGl4wSWyQefPC0WyUicO1jC_8d8BpGwqC_p6AX21lOUywZxedTsBRHDAVz0KoDNdBwvxKyEeUhn4cy56QBjXiv97fzu1jKZQ==)
11. [youtube.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEemmBy3O-ZM8HTQWbbXt1OZqiITYAois9WwltaahqasNCAXZGVd5yA0bi-NsErML7fPweHZeIbf___XwNKfIxdYEBOkl7kobHMIgbfOfSJ4wOgR49-0KVKcLSPm2Ue_9E=)
12. [reddit.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFe1KDglP93NJdEgdZ0_adAPMZG-IUUC0pSMNfGNWia7yKkRuWLFu5B1FoOY6_3U9QDOPRm4GZoG9QbovihUGeNw5dtCqTK4Zr2gLPJcGyadVeAE6JSsVlnsJIdbYqySorLAZiL10_rW_cCNZ7GS6141IKzd08YL09ntrPS1LTnPi8Dnl0G8hAkq6E70VZYWr7ypcRP)
13. [zenn.dev](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFAI01zorCCFq4gxcvVSI8GglGIwyaERk3owTjQeLvV6rdCoWQ5XgwuFWrPIqPd2zRYm6moPlP_ekjujrrvBD_VLyQIvGu1U5b9hSxtd6eQto1iWzPPDexmAOk58At_fV4ktkP7YYGsbaiZ4HYKzjz6YYzpftE0irQ3iDpmCHb1YQqeseR46dxtDfuk6ZzxD8s=)
14. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQE_OjWqBPgAGaMrZTzwNONQHEA3cV8b-ompAdeikhbhaF7uwM4ihqV_mDxcuzSfv1qCpyMPOasruYJ8X9tUlrLVQUnaJ-jKTfYOXXzE334Vx4AUxXU70ZLpmLhVHloc-WbiB-PGmfepCQ==)
15. [netlify.app](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHkvFaKUa_wWTX51D4VpcCSDoSDQ7sNBG8mJWTwoVN6nrzy5voBRKZB2xXlHJPRkuKE4JWoXs21ATULTecPXA77XcGHa5ypMpCNobYv1U24lu413fhTnNu_O6T-wQJE-m9zP_5Q_Qr533TzXW-j7N2BBibtCzherOwjRop2ty3qJgc4461x3oeKturlAHqm5weJjQr6xA70CwIjuxXMojYMks141O6kijq3-1nEo1vntHkSwcJjkR9Zru0E_-KQ70xyGCE_7rKhQsM0C7WyvO8=)