---
title: 競技プログラミングにおける木構造（Tree）の網羅的解析：データ構造、計算量、および実装手法
emoji: 🌲
type: tech
topics:
- algorithm
- datastructure
- tree
published: false
---

**Key Points:**
*   競技プログラミングにおいて、木（Tree）構造はデータへの高速なアクセス、更新、および区間演算を実現するための極めて重要な概念であると考えられます。
*   セグメント木やフェニック木（BIT）などの完全二分木を応用した構造は、配列上の区間クエリを対数時間で処理する能力を持ち、現代のアルゴリズムコンテストにおいて必須の知識であると広く認識されています。
*   素集合データ構造（Union-Find）の計算量にはアッカーマン関数の逆関数 \(\alpha(N)\) が現れ、実質的な定数時間 \(\mathcal{O}(1)\) での集合の併合・判定が可能であることが数学的に証明されています。
*   ダブリングを用いた最小共通祖先（LCA）の特定や、トライ木（Trie）による文字列の接頭辞管理など、対象データの特性（グラフの形状や文字列の性質）に最適化された特化型の木構造が多数存在します。

本稿では、競技プログラミング（Competitive Programming）の領域において頻出かつ極めて重要な役割を果たす「木（Tree）」ベースのデータ構造群について、学術的かつ網羅的な視点から整理・解析を行います。アルゴリズムの設計段階において、適切なデータ構造を選択することは、時間計算量および空間計算量の制約（通常、実行時間2秒以内、メモリ数百MB以内）を満たす上で決定的な要因となります [cite: 1]。本解析では、各データ構造の理論的背景、操作の定義、計算量の数学的評価、およびC++を中心とした実用的な実装例を詳解します。

---

## 1. 主要なトピックの列挙

競技プログラミングにおいて習熟が求められる主要な木構造および関連アルゴリズムは、以下の通りです。

1.  **二分木、二分探索木、平衡二分探索木**（Binary Tree, BST, AVL木、赤黒木、Treap、RBSTなど）
2.  **セグメント木**（Segment Tree / 遅延評価セグメント木を含む）
3.  **フェニック木**（Binary Indexed Tree / Fenwick Tree）
4.  **Union-Find**（素集合データ構造 / Disjoint Set Union）
5.  **トライ木**（Trie / Prefix Tree / Binary Trie）
6.  **ヒープ**（Heap / 優先度付きキュー）
7.  **木の走査**（DFS: 深さ優先探索、BFS: 幅優先探索、オイラーツアー）
8.  **最小共通祖先**（LCA: Lowest Common Ancestor）
9.  **その他競技プログラミングで重要な木構造**（HLD: Heavy-Light Decomposition、Link-Cut Treeなど）

以降のセクションでは、これらの各トピックについて、理論的側面と実践的側面（計算量、実装コード）の両面から深く掘り下げます。

---

## 2. 各トピックの詳細解説

### 2.1 二分木、二分探索木、平衡二分探索木 (BST & Balanced Trees)

#### トピック・操作の説明
**二分木（Binary Tree）**は、各ノードが最大2つの子ノード（左の子、右の子）を持つ木構造です。これに順序の概念を導入したものが**二分探索木（Binary Search Tree: BST）**であり、「左の子孫のすべての値 < 親の値 < 右の子孫のすべての値」という制約を満たします [cite: 2]。これにより、特定の値の検索を木構造を辿ることで効率的に行うことが可能です。

しかし、単純なBSTでは、要素が昇順や降順で挿入された場合に木が一直線（リスト状）に偏り、探索効率が極端に悪化する欠点があります。これを防ぐために、木の高さを \(\mathcal{O}(\log N)\) に保つよう動的に構造を再編するものが**平衡二分探索木（Balanced Binary Search Tree）**です [cite: 2, 3]。代表的なものに AVL木、赤黒木、Treap、RBST（Randomized Binary Search Tree）などがあります [cite: 4, 5]。

競技プログラミングにおいては、C++の標準ライブラリである `std::set` や `std::map`（内部的には赤黒木として実装されることが多い）が頻繁に用いられます [cite: 3, 5]。しかし、「\(k\) 番目に小さい値の取得」や「区間反転」などの高度な操作が要求される問題では、標準ライブラリでは対応できないため、RBSTやImplicit Treapなどを独自に実装する必要があります [cite: 2, 4]。

#### 計算量 (Time & Space Complexity)

| 操作 | 時間計算量 (Time) | 空間計算量 (Space) | 備考 |
| :--- | :--- | :--- | :--- |
| 挿入 (`insert`) | \(\mathcal{O}(\log N)\) | \(\mathcal{O}(N)\) | 平衡木の場合。偏った通常のBSTでは最悪 \(\mathcal{O}(N)\) [cite: 2] |
| 削除 (`erase`) | \(\mathcal{O}(\log N)\) | - | 同上 |
| 検索 (`find` / `lower_bound`) | \(\mathcal{O}(\log N)\) | - | 値の存在確認、あるいは \(x\) 以上の最小要素の検索 [cite: 5] |
| \(k\)番目の取得 | \(\mathcal{O}(\log N)\) | - | 各ノードに部分木サイズを持たせた自作平衡木のみ対応 [cite: 2, 3] |

#### 実装例 (C++)
以下は、期待値的に木の高さを \(\mathcal{O}(\log N)\) に抑える **RBST (Randomized Binary Search Tree)** の基本的なノード構造と、部分木のサイズを管理する実装の骨格です [cite: 4]。

```cpp
#include <iostream>
#include <vector>
#include <random>

using namespace std;

// RBSTのノード定義
struct Node {
    int val;
    int size; // 部分木のサイズ（k番目の取得などに使用）
    Node* left;
    Node* right;
    Node(int v) : val(v), size(1), left(nullptr), right(nullptr) {}
};

// 部分木のサイズを取得するヘルパー関数
int get_size(Node* t) {
    return t ? t->size : 0;
}

// ノードの情報を更新（サイズ再計算など）
Node* update(Node* t) {
    if (t) {
        t->size = 1 + get_size(t->left) + get_size(t->right);
    }
    return t;
}

// 木 t を size が k になるように左右に分割 (左: [0, k), 右: [k, N))
pair<Node*, Node*> split(Node* t, int k) {
    if (!t) return {nullptr, nullptr};
    if (k <= get_size(t->left)) {
        auto s = split(t->left, k);
        t->left = s.second;
        return {s.first, update(t)};
    } else {
        auto s = split(t->right, k - get_size(t->left) - 1);
        t->right = s.first;
        return {update(t), s.second};
    }
}
```

---

### 2.2 セグメント木 (Segment Tree)

#### トピック・操作の説明
**セグメント木（Segment Tree）**は、1次元配列の区間に対するクエリ（区間和、区間最小値、区間最大値など）と、要素の更新を共に対数時間で行うための完全二分木ベースのデータ構造です [cite: 6]。

セグメント木は、「モノイド（Monoid）」と呼ばれる代数的構造を持つ任意の演算に対して適用可能です [cite: 7]。モノイドは以下の2条件を満たす集合 \(S\) と二項演算 \(\cdot\) の組です [cite: 7]：
1.  **結合則**: 任意の \(a, b, c \in S\) に対して、\((a \cdot b) \cdot c = a \cdot (b \cdot c)\)
2.  **単位元の存在**: 任意の \(a \in S\) に対して、\(a \cdot e = e \cdot a = a\) となる単位元 \(e\) が存在する。

内部的には、要素数 \(N\) の配列の葉ノードを持つ完全二分木を構築します [cite: 6]。実用上は、配列のサイズを2の冪乗（\(2^{\lceil \log_2 N \rceil}\)）に拡張し、1-indexed または 0-indexed の1次元配列として木を表現します。親ノード \(k\) の子ノードは \(2k\) および \(2k+1\) に位置づけられ、ポインタを使わずに配列のインデックス計算のみで木を走査できるため、キャッシュ効率が高く高速に動作します [cite: 7, 8]。

#### 計算量 (Time & Space Complexity)

| 操作 | 時間計算量 (Time) | 空間計算量 (Space) | 備考 |
| :--- | :--- | :--- | :--- |
| 構築 (`build`) | \(\mathcal{O}(N)\) | \(\mathcal{O}(N)\) | 事前に配列が与えられている場合、葉から根に向けて一斉に計算可能 [cite: 7, 9] |
| 1点更新 (`update`) | \(\mathcal{O}(\log N)\) | - | 葉から根へ辿りながら値を再計算する [cite: 6] |
| 区間取得 (`query`) | \(\mathcal{O}(\log N)\) | - | 目的の区間を被覆する最大 \(\mathcal{O}(\log N)\) 個のノードを結合する [cite: 6] |

※空間計算量について、要素数 \(N\) に対してノード数は最大 \(4N\) 程度確保する必要があります（完全二分木にするため）[cite: 10, 11]。

#### 実装例 (C++)
RMQ（Range Minimum Query）等の任意のモノイド演算に対応した汎用的なセグメント木の実装例です [cite: 7]。

```cpp
#include <vector>
#include <functional>
#include <algorithm>

template<typename Monoid>
struct SegmentTree {
    using F = std::function<Monoid(Monoid, Monoid)>;
    int sz;
    std::vector<Monoid> seg;
    const F f;          // 二項演算
    const Monoid M1;    // 単位元

    // コンストラクタ
    SegmentTree(int n, const F f, const Monoid &M1) : f(f), M1(M1) {
        sz = 1;
        while(sz < n) sz <<= 1; // n以上の最小の2の冪乗
        seg.assign(2 * sz, M1);
    }

    // k番目の値をxに更新する
    void update(int k, const Monoid &x) {
        k += sz;
        seg[k] = x;
        while(k >>= 1) { // 親ノードへ遡りながら更新
            seg[k] = f(seg[2 * k], seg[2 * k + 1]);
        }
    }

    // 区間 [a, b) の演算結果を取得
    Monoid query(int a, int b) {
        Monoid L = M1, R = M1;
        for(a += sz, b += sz; a < b; a >>= 1, b >>= 1) {
            if(a & 1) L = f(L, seg[a++]);
            if(b & 1) R = f(seg[--b], R);
        }
        return f(L, R);
    }
};
// 使用例（区間最小値）:
// auto f = [](int a, int b) { return std::min(a, b); };
// SegmentTree<int> seg(N, f, INT_MAX);
```

---

### 2.3 フェニック木 (Binary Indexed Tree / Fenwick Tree)

#### トピック・操作の説明
**フェニック木（Fenwick Tree）**、または **Binary Indexed Tree (BIT)** は、配列の「1点加算（Point Add）」と「先頭からの区間和（Prefix Sum）」をそれぞれ \(\mathcal{O}(\log N)\) で計算できるデータ構造です [cite: 12, 13, 14]。

累積和（Prefix Sum array）を用いれば区間和の取得は \(\mathcal{O}(1)\) ですが、値の更新には \(\mathcal{O}(N)\) がかかります [cite: 12]。一方、BITはセグメント木の機能を「区間和」に限定して無駄を省いたものと見なすことができ、セグメント木と比べて実装が極めて短く、メモリ使用量も小さく、定数倍の実行速度が速いという圧倒的な利点があります [cite: 14, 15]。

BITの核心は、2進数表現とビット演算の活用にあります。各ノードは、配列のインデックスの **LSB（Least Significant Bit：最も右にある1のビット）** が表す長さの部分区間の和を保持します [cite: 14, 16]。
LSBは2の補数表現を利用して `i & -i` で取得できます [cite: 14]。
*   加算処理（`add`）: インデックス \(i\) に加算した後、`i += (i & -i)` で親ノードへ移動しながら値を更新します。
*   和の取得（`sum`）: インデックス \(i\) から始めて、`i -= (i & -i)` で部分的な和を足し合わせながら遡ります。

区間 \([L, R]\) の和は `sum(R) - sum(L-1)` として計算できるため、差分によって任意の区間和も取得可能です [cite: 15]。また、配列要素をソートしながらBITに挿入していくことで、「転倒数（Inversion Number）」を \(\mathcal{O}(N \log N)\) で求めるアルゴリズムは競技プログラミングの頻出テクニックです [cite: 14, 15]。

#### 計算量 (Time & Space Complexity)

| 操作 | 時間計算量 (Time) | 空間計算量 (Space) | 備考 |
| :--- | :--- | :--- | :--- |
| 初期化 (`init`) | \(\mathcal{O}(N)\) | \(\mathcal{O}(N)\) | セグ木と異なり、サイズはちょうど要素数分で済む [cite: 13, 14] |
| 1点加算 (`add`) | \(\mathcal{O}(\log N)\) | - | - |
| 先頭からの和 (`sum`) | \(\mathcal{O}(\log N)\) | - | - |
| 任意の区間和 | \(\mathcal{O}(\log N)\) | - | `sum(R) - sum(L-1)` で実現 [cite: 15] |

#### 実装例 (C++)
BITの内部構造は 1-indexed を前提とするとビット演算の整合性が保ちやすく、実装が簡潔になります [cite: 7, 13]。

```cpp
#include <vector>

// 1-indexed Binary Indexed Tree
template <typename T>
struct FenwickTree {
    int n;
    std::vector<T> bit;

    FenwickTree(int n) : n(n), bit(n + 1, 0) {}

    // i番目 (1-indexed) に x を加算する
    void add(int i, T x) {
        for (; i <= n; i += (i & -i)) {
            bit[i] += x;
        }
    }

    // 先頭から i番目 (1-indexed) までの和を求める
    T sum(int i) {
        T s = 0;
        for (; i > 0; i -= (i & -i)) {
            s += bit[i];
        }
        return s;
    }
    
    // 区間 [l, r] (1-indexed) の和を求める
    T sum_range(int l, int r) {
        return sum(r) - sum(l - 1);
    }
};
```

---

### 2.4 Union-Find (素集合データ構造)

#### トピック・操作の説明
**Union-Find**（素集合データ構造、Disjoint Set Union: DSU）は、複数の要素がどのグループ（集合）に属しているかを管理し、以下の2つの操作を極めて高速に行うためのデータ構造です [cite: 17, 18, 19]：
1.  **判定（Find / Same）**: 2つの要素が同じグループに属しているかを判定する。
2.  **併合（Union / Merge）**: 2つの要素が属するグループ同士を1つのグループに統合する。

グラフの連結成分の管理、最小全域木を求めるクラスカル法（Kruskal's Algorithm）、ネットワークの動的連結性判定などで必須の構造です [cite: 18]。

初期状態では、各要素は自身を根とする独立した木（要素数1のグループ）を形成しています。
Union-Findを高速化するためには、以下の2つの決定的な最適化手法（ヒューリスティクス）を適用します [cite: 17, 20]：
*   **経路圧縮（Path Compression）**: `Find` 操作で根を探す際、探索経路上にあるすべてのノードを直接根に繋ぎ直します。これにより、次からの探索が \(\mathcal{O}(1)\) になります。
*   **ランク/サイズによる併合（Union by Size / Rank）**: `Union` 操作の際、要素数（または木の深さ）が少ないグループを、大きいグループの根に繋ぎます。これにより、木が一直線に伸びてしまう最悪ケースを防ぎます。

#### 計算量 (Time & Space Complexity)
この2つの最適化を組み合わせることで、各操作のならし計算量（Amortized Time Complexity）は \(\mathcal{O}(\alpha(N))\) となります [cite: 18, 20, 21]。
ここで、\(\alpha(N)\) は**アッカーマン関数（Ackermann function）の逆関数**を指します [cite: 17, 22, 23]。アッカーマン関数 \(A(n, n)\) は天文学的な速度で爆発的に増加する関数であるため、その逆関数である \(\alpha(N)\) は、宇宙の原子の数ほどの \(N\) であっても実質的に4以下の定数に収束します [cite: 22, 23]。したがって、競技プログラミングの文脈では事実上の定数時間 \(\mathcal{O}(1)\) として扱われます [cite: 17]。

| 操作 | 時間計算量 (Time) | 空間計算量 (Space) | 備考 |
| :--- | :--- | :--- | :--- |
| 初期化 | \(\mathcal{O}(N)\) | \(\mathcal{O}(N)\) | 親の配列（およびサイズ配列）を確保 [cite: 13, 20] |
| 根の検索 (`root/find`) | ならし \(\mathcal{O}(\alpha(N))\) | - | 実用上ほぼ \(\mathcal{O}(1)\) [cite: 17] |
| 併合 (`unite/merge`) | ならし \(\mathcal{O}(\alpha(N))\) | - | 同上 [cite: 17] |
| 連結判定 (`same`) | ならし \(\mathcal{O}(\alpha(N))\) | - | 同上 [cite: 17] |

※数理的な厳密な証明において、\(Q\) 回のクエリと \(N\) 頂点に対する全体の計算量は \(\mathcal{O}(N + Q\alpha(N))\) であることが示されています [cite: 22]。

#### 実装例 (C++)
配列 `parent_or_size` を用いて、根ノードには「グループサイズの負の値」を、それ以外のノードには「親ノードのインデックス」を格納することで、メモリ効率と実装の簡素化を図る典型的なテクニックです [cite: 24]。

```cpp
#include <vector>
#include <numeric>

struct UnionFind {
    // 親ノードのインデックスを保持。自分が根の場合は -(そのグループのサイズ) を格納する。
    std::vector<int> parent_or_size;

    UnionFind(int n) : parent_or_size(n, -1) {}

    // 経路圧縮を伴う根の検索
    int root(int x) {
        if (parent_or_size[x] < 0) {
            return x; // 自身が根
        }
        // 親を根に繋ぎ直す（経路圧縮）
        return parent_or_size[x] = root(parent_or_size[x]);
    }

    // 要素 x と y が同じグループに属するか判定
    bool same(int x, int y) {
        return root(x) == root(y);
    }

    // x と y を併合する (Union by Size)
    bool unite(int x, int y) {
        int rx = root(x);
        int ry = root(y);
        if (rx == ry) return false;

        // サイズが大きい方に小さい方を繋ぐ
        if (-parent_or_size[rx] < -parent_or_size[ry]) {
            std::swap(rx, ry);
        }
        parent_or_size[rx] += parent_or_size[ry]; // サイズの更新
        parent_or_size[ry] = rx;                  // 根の変更
        return true;
    }

    // x が属するグループのサイズを返す
    int size(int x) {
        return -parent_or_size[root(x)];
    }
};
```

---

### 2.5 トライ木 (Trie / Prefix Tree)

#### トピック・操作の説明
**トライ木（Trie）**、またはプレフィックス木は、文字列の集合を効率的に保存、検索するための木構造です [cite: 25]。根（Root）は空文字列を表し、木を下るごとに1文字ずつ文字が追加されていきます。文字列の共通する接頭辞（Prefix）を同じノードとして共有するため、無駄なデータ保持を防ぎ、高速な前方一致検索を実現します [cite: 25, 26]。

主な用途としては、多数の単語からの辞書検索、文字列の出現回数のカウント、オートコンプリート機能の実装などが挙げられます [cite: 26, 27]。
また、Trie木を拡張し、検索失敗時の遷移先（Failureリンク）をBFSで事前計算することで、複数パターンの同時検索を線形時間で行う **Aho-Corasick（エイホ・コラシック）法** も競技プログラミングの文字列アルゴリズムにおいて極めて重要です [cite: 28]。

さらに、競技プログラミング特有の応用として **Binary Trie（二分トライ木）** があります [cite: 29, 30]。これは文字列の代わりに、非負整数を「0と1からなる固定長のビット列」とみなしてTrie木に格納するものです。これにより、整数の集合に対して「\(k\) 番目に小さい値の取得」や「要素全体へのXOR演算」を効率的に処理できます [cite: 29]。

#### 計算量 (Time & Space Complexity)

| 操作 | 時間計算量 (Time) | 空間計算量 (Space) | 備考 |
| :--- | :--- | :--- | :--- |
| 文字列の挿入 | \(\mathcal{O}(\|S\|)\) | $\mathcal{O}(\sum \|S\| \times \Sigma)$ | $\|S\|$は文字列の長さ、$\Sigma$は文字の種類数 [cite: 25] |
| 文字列の検索 | \(\mathcal{O}(\|S\|)\) | - | 検索対象の文字数のみに依存し、登録語数に依存しない [cite: 25] |
| 整数の挿入/検索 | \(\mathcal{O}(d)\) | \(\mathcal{O}(N \times d)\) | Binary Trieの場合。\(d\) は最大ビット長（例: 32や64）[cite: 29] |

※Trie木最大の弱点は「空間計算量（メモリ使用量）」です。文字の種類数 $\Sigma$（英小文字なら26）分のポインタや配列を各ノードで保持するとメモリ消費が爆発するため、実行時エラー（MLE）に注意が必要です [cite: 27, 31]。連想配列（Map）を用いたり、遷移が存在するノードのみ確保するなどの工夫が行われます。

#### 実装例 (C++)
アルファベット小文字（26種類）に限定したTrie木の実装例です [cite: 26]。

```cpp
#include <string>
#include <vector>

struct TrieNode {
    std::vector<int> next; // 子ノードへのインデックス
    int count;             // このノードを末尾とする単語の数
    
    TrieNode() : next(26, -1), count(0) {}
};

struct Trie {
    std::vector<TrieNode> nodes;

    Trie() {
        nodes.emplace_back(); // 根ノードを追加
    }

    // 文字列の挿入
    void insert(const std::string& word) {
        int node_idx = 0;
        for (char c : word) {
            int char_idx = c - 'a';
            if (nodes[node_idx].next[char_idx] == -1) {
                // 子ノードが存在しなければ新規作成
                nodes[node_idx].next[char_idx] = nodes.size();
                nodes.emplace_back();
            }
            node_idx = nodes[node_idx].next[char_idx];
        }
        nodes[node_idx].count++; // 単語の終端にマーク
    }

    // 文字列の検索 (存在する場合はそのカウントを返す)
    int search(const std::string& word) {
        int node_idx = 0;
        for (char c : word) {
            int char_idx = c - 'a';
            if (nodes[node_idx].next[char_idx] == -1) {
                return 0; // プレフィックスが存在しない
            }
            node_idx = nodes[node_idx].next[char_idx];
        }
        return nodes[node_idx].count;
    }
};
```

---

### 2.6 ヒープ (Heap / 優先度付きキュー)

#### トピック・操作の説明
**ヒープ（Heap）**は、「親ノードの値が子ノードの値よりも常に小さい（または大きい）」という順序制約を満たす完全二分木です [cite: 32]。この性質により、根ノードには常に集合内の最小値（または最大値）が位置することになります。

競技プログラミングにおいては、要素の追加と最小（最大）値の取り出しを高速に行う **優先度付きキュー（Priority Queue）** の内部実装として使われます [cite: 33, 34]。ダイクストラ法（単一始点最短経路問題）における未確定ノードの探索など、常に最良の選択肢を取り続ける貪欲法アルゴリズムにおいて不可欠な役割を担います [cite: 26, 32]。

木構造はポインタを用いず、0-indexed の1次元配列で表現するのが一般的です。ノード \(i\) に対して、親は \((i-1)/2\)、左の子は \(2i+1\)、右の子は \(2i+2\) となります [cite: 32]。
注目すべき点として、空の状態から \(N\) 個の要素を一つずつ挿入すると \(\mathcal{O}(N \log N)\) かかりますが、既にデータが入った配列全体をヒープの条件を満たすように再構築する操作（**Heapify**）は、下から上へ木を整理することで \(\mathcal{O}(N)\) の線形時間で行うことができます [cite: 33]。

#### 計算量 (Time & Space Complexity)

| 操作 | 時間計算量 (Time) | 空間計算量 (Space) | 備考 |
| :--- | :--- | :--- | :--- |
| 最大/最小の参照 (`top`) | \(\mathcal{O}(1)\) | - | 根の値を参照するのみ [cite: 33] |
| 要素の挿入 (`push`) | \(\mathcal{O}(\log N)\) | \(\mathcal{O}(N)\) | 末尾に追加後、親と比較し上方へ浮上させる [cite: 33] |
| 最大/最小の削除 (`pop`) | \(\mathcal{O}(\log N)\) | - | 末尾要素を根に置き、子と比較し下方へ沈める [cite: 33] |
| 全体からの構築 (`heapify`) | \(\mathcal{O}(N)\) | - | 要素が揃っている配列をヒープ化する操作 [cite: 33] |

#### 実装例 (C++)
通常、C++では標準ライブラリの `std::priority_queue` を用います [cite: 32]。以下は標準ライブラリを利用して最小ヒープ（小さい順に取り出されるキュー）を構成する例です。

```cpp
#include <iostream>
#include <queue>
#include <vector>

using namespace std;

int main() {
    // std::priority_queue はデフォルトで最大ヒープ。
    // 最小ヒープにするには、内部コンテナ(vector)と、比較関数(greater)を指定する。
    priority_queue<int, vector<int>, greater<int>> min_heap;

    // 要素の追加: O(log N)
    min_heap.push(5);
    min_heap.push(2);
    min_heap.push(8);
    min_heap.push(1);

    // 要素の取り出し
    while (!min_heap.empty()) {
        cout << min_heap.top() << " "; // 最小値を参照: O(1)
        min_heap.pop();                // 最小値を削除: O(log N)
    }
    // 出力: 1 2 5 8
    return 0;
}
```

---

### 2.7 木の走査 (DFS, BFS, オイラーツアー)

#### トピック・操作の説明
木に対する基本的な走査（Traversal）アルゴリズムは、グラフ探索の基礎でもあり、あらゆる木構造問題の出発点です。

*   **深さ優先探索（DFS: Depth-First Search）**: 根から出発し、可能な限り深く（子ノードの方向へ）探索を進め、行き止まりに達したら親へ戻る（バックトラック）手法です。再帰関数（スタック）を用いて極めて簡潔に実装できます [cite: 35]。
*   **幅優先探索（BFS: Breadth-First Search）**: 根から出発し、根からの距離（深さ）が等しいノードを階層ごとに順に探索する手法です。キュー（Queue）を用いて実装され、最短経路や最短手数を求める際によく用いられます [cite: 35]。

木特有の走査技術として重要なのが **オイラーツアー（Euler Tour）** です。これは、木に対するDFSを行い、「ノードを最初に訪れた時（行きがけ順：Pre-order）」と「ノードから離れる直前（帰りがけ順：Post-order）」の両方のタイミングを1次元配列に記録するテクニックです [cite: 35, 36]。
これにより、木構造における「あるノードを根とする部分木」が、1次元配列上の「連続した区間」として表現可能になります。この変換により、前述の**フェニック木（BIT）やセグメント木を用いて、部分木に対するクエリ処理を高速に行う**ことができるようになります [cite: 36]。

#### 計算量 (Time & Space Complexity)

| 探索手法 | 時間計算量 (Time) | 空間計算量 (Space) | 備考 |
| :--- | :--- | :--- | :--- |
| 深さ優先探索 (DFS) | \(\mathcal{O}(V + E)\) | \(\mathcal{O}(V)\) | 木の場合 \(E = V-1\) なので実質 \(\mathcal{O}(V)\)。空間は再帰スタック [cite: 35] |
| 幅優先探索 (BFS) | \(\mathcal{O}(V + E)\) | \(\mathcal{O}(V)\) | 同上。空間はキューが保持するノード数 [cite: 35] |
| オイラーツアー | \(\mathcal{O}(V)\) | \(\mathcal{O}(V)\) | DFSの拡張。要素数 \(2V\) 程度の配列を生成 [cite: 35, 36] |

#### 実装例 (C++)
オイラーツアーによって「行きがけ順」と「帰りがけ順」を記録する実装例です。

```cpp
#include <iostream>
#include <vector>

using namespace std;

int N; // 頂点数
vector<vector<int>> adj; // 隣接リスト表現の木
vector<int> in_time;     // 頂点に最初に入った時刻 (行きがけ)
vector<int> out_time;    // 頂点から完全に出た時刻 (帰りがけ)
int timer = 0;

// DFSによるオイラーツアー
void dfs_euler(int u, int p) {
    in_time[u] = timer++; // 行きがけを記録
    
    for (int v : adj[u]) {
        if (v != p) { // 親への逆流を防ぐ
            dfs_euler(v, u);
        }
    }
    
    out_time[u] = timer++; // 帰りがけを記録
}

int main() {
    // ... グラフの入力受け取りなどは省略 ...
    in_time.resize(N);
    out_time.resize(N);
    
    // 根を頂点0としてDFSを開始
    dfs_euler(0, -1);
    
    // これにより、頂点 u の部分木は、配列の区間 [in_time[u], out_time[u]) に対応する。
    return 0;
}
```

---

### 2.8 最小共通祖先 (LCA: Lowest Common Ancestor)

#### トピック・操作の説明
根付き木において、与えられた2つの頂点 \(u\) と \(v\) の両方の祖先であるノードのうち、最も深さが深い（根から最も遠い、すなわち \(u, v\) に最も近い）ノードを **最小共通祖先（LCA）** と呼びます [cite: 37, 38]。LCAは、木における任意の2頂点間の距離を求める際などに不可欠です（木上の2頂点 \(u, v\) 間の距離は `dist(u) + dist(v) - 2 * dist(LCA(u, v))` で計算可能）。

LCAを \(\mathcal{O}(\log N)\) で高速に求めるための標準的な手法が **ダブリング（Doubling）** です [cite: 36, 37, 39, 40]。
ダブリングでは、各頂点について「\(2^k\) 回親を辿った先の頂点（\(2^k\) 先の祖先）」を事前に計算し、テーブル（2次元配列）に保持しておきます [cite: 36, 41]。構築の漸化式は、`table[k+1][i] = table[k][ table[k][i] ]` （\(i\) から \(2^k\) 遡った頂点から、さらに \(2^k\) 遡ると、合計 \(2^{k+1}\) 遡った頂点になる）という動的計画法に基づきます [cite: 37]。

クエリ時のアルゴリズムは以下の通りです [cite: 37, 40]：
1.  深さが深い方の頂点を、ダブリングを用いて浅い方の頂点と同じ深さになるまで引き上げる（高さを揃える）。
2.  高さを揃えた時点で2つの頂点が一致していれば、それがLCAである。
3.  一致していなければ、上位ビット（大きなステップ）から順に「\(2^k\) 個上の祖先が互いに異なるか」を判定し、異なるならば2頂点を同時にその位置まで引き上げる処理を繰り返す。
4.  最終的に、両者はLCAの直下（1つ下）のノードに到達するため、その親がLCAとなる。

#### 計算量 (Time & Space Complexity)

| 操作 | 時間計算量 (Time) | 空間計算量 (Space) | 備考 |
| :--- | :--- | :--- | :--- |
| 前処理 (構築) | \(\mathcal{O}(N \log N)\) | \(\mathcal{O}(N \log N)\) | サイズ \(\log N \times N\) のテーブルを作成 [cite: 37, 41] |
| LCA取得クエリ | \(\mathcal{O}(\log N)\) | - | 各ビットごとに \(O(1)\) の判定 [cite: 37, 41] |

#### 実装例 (C++)
ダブリングを利用したLCAの標準的な実装例です [cite: 37]。

```cpp
#include <vector>
#include <cmath>

using namespace std;

struct DoublingLCA {
    int LOG;
    vector<int> depth;
    vector<vector<int>> table; // table[k][i] = 頂点iから2^k回親を辿った頂点
    
    DoublingLCA(const vector<vector<int>>& g, int root = 0) {
        int n = g.size();
        LOG = 1;
        while ((1 << LOG) < n) LOG++;
        
        depth.assign(n, 0);
        table.assign(LOG, vector<int>(n, -1));
        
        // DFSで深さと直近の親(2^0)を初期化
        dfs(g, root, -1, 0);
        
        // ダブリングテーブルの構築 O(N log N)
        for (int k = 0; k + 1 < LOG; k++) {
            for (int i = 0; i < n; i++) {
                if (table[k][i] == -1) table[k + 1][i] = -1;
                else table[k + 1][i] = table[k][table[k][i]];
            }
        }
    }
    
    void dfs(const vector<vector<int>>& g, int v, int p, int d) {
        table[v] = p;
        depth[v] = d;
        for (int u : g[v]) {
            if (u != p) dfs(g, u, v, d + 1);
        }
    }
    
    // LCAを取得 O(log N)
    int query(int u, int v) {
        if (depth[u] > depth[v]) swap(u, v); // vの方を深くする
        
        // 1. 高さを揃える
        for (int i = LOG - 1; i >= 0; i--) {
            if (((depth[v] - depth[u]) >> i) & 1) {
                v = table[i][v];
            }
        }
        if (u == v) return u;
        
        // 2. LCAの直下まで同時に遡る
        for (int i = LOG - 1; i >= 0; i--) {
            if (table[i][u] != table[i][v]) {
                u = table[i][u];
                v = table[i][v];
            }
        }
        // 最終的な u, v の親が LCA
        return table[u];
    }
};
```

---

### 2.9 その他競技プログラミングで重要な木構造

競技プログラミングの上級者帯（AtCoder水色〜赤色など）において、上述した基礎的な木構造に加えて、高度な概念と組み合わせて頻出する特化型データ構造がいくつか存在します。

#### Heavy-Light Decomposition (HLD: 重軽分解)
*   **説明**: 木構造におけるパス（経路）をいくつかの1次元配列に分割（直線化）するテクニックです。各頂点から、最も部分木のサイズが大きい子への辺を「Heavy edge」、それ以外を「Light edge」と定義します。これにより、木上の任意の2頂点間の経路が \(\mathcal{O}(\log N)\) 個の連続したHeavy pathに分解されます。各パスに対してセグメント木などを適用することで、木上の任意のパスに対するクエリ（更新・集約など）を \(\mathcal{O}(\log^2 N)\) で処理可能になります。
*   **時間計算量**: 構築 \(\mathcal{O}(N)\), パスクエリ \(\mathcal{O}(\log^2 N)\)。

#### Link-Cut Tree
*   **説明**: 動的木（Dynamic Tree）と呼ばれるデータ構造の代表例です。「辺の追加（Link）」「辺の削除（Cut）」を動的に行いつつ、パス上のクエリやLCAなどを高速に処理できます。内部的にはSplay Treeなどの平衡二分探索木を用いてHeavy-Light Decompositionを動的に維持するような構造を持っています。
*   **時間計算量**: 各操作のならし計算量 \(\mathcal{O}(\log N)\)。実装が非常に複雑であるため、競技プログラミングでは出題頻度は高くないものの、一部の難問では必須となります。

#### Wavelet Matrix (ウェーブレット行列)
*   **説明**: 元々は簡潔データ構造や文字列アルゴリズムで用いられる手法ですが、競技プログラミングでは「静的な配列に対する2次元的なクエリ」を処理する木構造として利用されます。「区間 \([L, R]\) の中で値が \([x, y]\) の範囲に収まる要素の数」や、「区間内の \(k\) 番目に小さい値」を高速に求めることができます。
*   **時間計算量**: 構築 \(\mathcal{O}(N \log \Sigma)\)、クエリ \(\mathcal{O}(\log \Sigma)\) （\(\Sigma\) は要素の最大値）。

---

## 結論

競技プログラミングにおける「木構造」は、単純なデータ表現の枠を超え、高度なアルゴリズムの基盤となる極めて強力なツールです。セグメント木やフェニック木（BIT）を通じた高速な区間演算の習得から始まり、Union-Findによる集合の動的管理、ダブリングによるLCA特定、さらにはTrie木を利用した文字列処理に至るまで、多角的なアプローチが求められます [cite: 6, 14, 17, 26, 37]。

これらのデータ構造は、独立して用いられるだけでなく、「オイラーツアー＋フェニック木」や「HLD＋セグメント木」のように、互いに組み合わされることで複雑な制約のクエリ問題を解決する鍵となります。理論的な計算量の理解と、キャッシュ効率や定数倍を意識した実装力の双方が、競技プログラミングにおける高度な問題解決において極めて重要であると言えます。

**Sources:**
1. [githubusercontent.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGPsWAm5zapHvCJ4pP5kJsIRYPX04LxWOEhWx8VhxOMX-PXdl07xADI5B2ZYjp12X2DAbhkh72UsyhAWcfPT8C7JPhey2NcXJIlsvz3eMmXwtnEhjYVmh7CBebpZstB1Fk0eO_3Ww4JM9RcdM3D5zMzpjJIc4M-8l4=)
2. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQERCzNOQjq9HJXUE_StJ1ZW6weiUhrkYCXueJqhQS7AhAZ37WSm8T4FNm_HC9ZHfgYIseJ2Px41C7NuJzGonyfh-qGrj_56aToDlExTEwi4RfTUaHsii4srMC58TuTaHUKcd-YtUyv3lMc=)
3. [ikatakos.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFx4Pw5XVSb-u2GS3sxpcXveewuwInd4Lulhou3JvWAaiJasou6cUYXWxKGjbfExai89LjMibHZU5_TiyV1cXuLErMLFbsfBmFgjPufWx8WhjatcR8UiLUx4QPWQ3lAmOaKrGxkrG3xUL73bzHEajtBARo0hga4jyV3yZVq5B-UTFEBCwFTbWP8fGLltTB979dTkh0p8gMOi0_7)
4. [github.io](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGClQuRdZu24iLAH-9Fw8wtKASO3jJNFbwmWTZF8VG3f61waA8ijv6Ece2Gnh89ugNasnTXMdkC5KrVCCcbh3WAZb5YPZVwLMvrazFRw53fLlxq6ufuOs5Y16wOsouayCIW28F541D-xMxaJG0xdlAzKEvqxg==)
5. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFI-vHSvj2OV5ZeBF9lW5F7h3WkVF2kRzZUQwmYEifglooBFXeNVDMs_kv8v-1Ru83wi-GbnLdrsI_P8y5dtYZRhx5DECMuPOiCjRQycfs7nTAmdu16aYTKvq0GRfBJAJF_TFYVEDxlBCBm)
6. [algo-logic.info](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH-TPJg3w9boKkbtYdZhds_UaM04ODBPFLArXtr_6EvbNUGe97iOsdPampca-C7f7YwlTc9sl4ekLTdH0i8pOvwVZfrMQSMOcD2BYpocH6l4l66HC3pJt6BGeav)
7. [github.io](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGksIvZnMalEqqAzdtiNS9oVXj7yaJzmNLlSsVSHwdsFRCazniNdLalUwcpZcjRNnr95IQcNdRAMeyHYiQvztBqfgDazo_Ql-9OIpSU6v_NCyYriAqwW2FlDSZSg9HnssmpeqUa3nDSu1CVRyDMJBlzkBN6R5gJc9m8-uuH)
8. [slideshare.net](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHva_PvTKrx02wFq_xR4-cmM3_Q33HwogF_p0Tgi12WPLhj7X-Ds5Cod69KimDnr9ZLVaqjAc8tOeReS-QAaTDQ-SymaphRNBvn75iIVTOYp9zS5VhYBASeGjYFbEjUJYTtfouH0lRwHLIcdWIh)
9. [zenn.dev](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEtz-1qC4LcS4aS1PmNv11uiQXBkZ3fC8RfnE-3d_RVoCEzZ4zJLqEailPxoZ_e-TwFOON_R6kqBsqEPtpTAunClh1FJSy8OTbk20jNExZzHFng1x5mt5pSjLahep7JN15Sxm_YtKc=)
10. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQH9uzCFDI5qFGICPNDgKmSRLVEXThzFtQFb4ue7lLzznlIbnKqrfHOqvLnudpuzEh5k4roKfimOxCRv4ogK-Rdq1UKkVLPqsfWYwT6IislZ_FItUsmxvA_5gA9YXjkjhvB-0jaIfblK7ipQUMrF)
11. [hatenablog.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQF1NbRicHk93E47R1FiISeRM40qDdUnet8sk7uQ2Vg_BFCXVuGKkuvelcAaangJ2jy2fQ8sBlSMZS95maPnhjjQgfn5UeNoUAN93o1RgvukFyqkBtjjtcsPS7QyM-yTPulrYA5KKXRR4V--6-g_k5j60zhn)
12. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG2ZWRwnMUafKZ_YlcQqUigDopUp69gPMeozVQplJfcwmAhtIEMSVyLV3KEhAVcwzyesFyxDcWqw2hKkaoP_nT7t2wBC2p7iXoCLdInl2fRvZ9XWDveZ9dp3M8ko7mKyLmqdwJ8yx49hRea8w_8rGQV)
13. [zenn.dev](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEmwBeo58lIE4h4DMO-xhmo97iPujWpHVGufqGTpQJ2EP_bw8scXI2wx244zKA-hpd5tulTa9F6JP6qo7wXJ-Afc0e3bSkMMVC9LI9HE8pkVRX4DLUaf0mQOphLWVYw22uFF8kfdOyi3YtqQkgqYC0xQTBYWw==)
14. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHZacQj7LorKEfQgF3r3eq3z4hBfha15imqk-hK0g8M1jU-BqEVcASLrrKh1gWO0gWKMzvJ4ztJZr98A2iACbfqraAWdejVUJYwjRbXh-DxW22MUArMTnwHpn6Hag8DrRk6Vq2XufAjgne4WmM9Rg==)
15. [algo-logic.info](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEBkd6vaEPCoDGydYGfXZC-rPDJX_GHgXxDd6PR3fNqLECi9nyWyhNSQ3fo7No2l8AZwEATKkyZLqNRZZzAlchgS1QNw-Fyp13TWxIyx6jLuGGDmOcc36OHz7_FLcn5QCLPjg==)
16. [note.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHc_6tm_UztnomVIJHVbHAoQyOC5_LohG57abudBQPWvtkISnzhQ3cm0PlDO2yXy3_yUb8g3AF2AQ3LKcJWFuyHEtLCy5-pmMnLJpIJOh12DH_-7CqYKb6znEog3zKT6RxAcGrH)
17. [zenn.dev](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGfqVfEQcyfPeewhaV0kCaCJtbooQKs29xHZg2b3q6jZyOHcaP5U1UH2Tluf1bbVbW6UPXEHiZ3W7GhFPvhS8K4rH36qpmOwc9NB3Cvbe8xDLq6-IQxjcoi79SPzeT_nQspfIpb7vq4TG3J10OX7Q==)
18. [techgym.jp](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEri2UsCk2rqW8M6h0-wVoMLhq2cs5d78qMxSoFrGwUlRtuW9goxzlhp3IiP-ycUMaOJM3WQ-0Klr1jIjBgaUp3tTIbIGthym3DFTscw5SDMvfjh6Yr5KwisA-L)
19. [maximum.vc](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFzqD1z5cE71BVEXgTyyKdYGZb7trQzPyqJhiL89lYqPMJyiNWVuyx9LKst1Cg17AlUaEO3yMEF1U66Bcd2efLAt_xrH9DcrPgFHWD-YS2cFwUy1TaZwQvtDXMyE8li34rhpyzlIlSVLg==)
20. [hatenablog.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQE_6qFFiwhIyePL3iwwGBCZ5Gz6ITxhS8DfzanGPJSvRBHhKg8cRPYFP-oL84ZVLASrsKvQ6-YKGVLaqoil6VdMEAZI64pf_AyKsqSSgCARBIEyErTy8Qbx5-CEi3NTrf7riqGSHUn_1ju1)
21. [zenn.dev](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEJ2EVrMTBpNJ9-0BOq5V_3ZulWdxPCFDfK3Mz3aKfis00h_lV863Pszn1f_o_MVAhbtuyoNNikObym9UVawvHn1m4TGWcHenAkUe2TjtJ0B0TyFNdFDxczUukiOWQ66wnBFaEqhZ4=)
22. [atcoder.jp](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHp23LlCIycCiBgj0bYGKFoTs-wKPI3tbyrcbm9ae9-urUpyqkBELvKYT1q0uCgVGsQkJ6d4yAUHBH6OgYYb0s_GpkrixK--INCeW8_84weTXullYTqcOXhr79U6CY8-zlvBrUgoVcp_XY2jAdUlGzW4ccsX0GDx9-OY5A=)
23. [atug.tokyo](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG-pyE3EmC4I1rFB-qv2Vb4nkP6Q1sQDwP3LSdyAZ6Zsg02uveAAZ4ZVJ1G5t-CW6MNIrA8uwEc4pxXlS2kpLF4jqoom-j2iFDYt862zhvG)
24. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFbJWEUBp3erfqnCDkKWJCfjiYtmn4fIwtTDWcszPPpFyBil7zSU8ZmrMt96PlHccGwGu65dwlL2gaAHeGgu3Nehtmdb4TZYNgUiVjpgJuFszcY_oywdgKD5kdyGlqLCztEtPNBSdagHw5K)
25. [github.io](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG0eIJsI3RcTloZ0gabzMMtdRL45zQWqf7xCPzaz4y8qlzDktwXnMgKoec7PXtnVJw6-duEO4i2eTvrNCBr6AaH_zhOzmkuepPyZn5SZ56EHNZu-80kSb3Oqv1DXSqQjbDNLStr8Uqmjc-6pSzQY0_TS2St4A==)
26. [algo-logic.info](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQE6tRAgd8HCxsboxFjydeOLv9OEKALav95JIeiJVy4uUbE_RYngj9nY0zvMgL2N3yd8UFbt1mNQcPha2_59FF7pjtttOLlhD2y9gO1VuLECYmVmsAcm4zEB)
27. [zenn.dev](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGrZ61hpf1BRUzP7NQHwpa3hOb3Ud4OH_b5_g1sgOwZ56Wb9OAPZA0w6J2tatERzTEyyiXzHUpvLocvuHBl5b1k_RVOTr4kd2ZQT8CpEYOKWXRl9pzIq2FnvlirSUDdA6OKz3bLlvMVXg==)
28. [ikatakos.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGlRfx7as1uZdvNw6tDOi6Yq_lD3rHxryl8wyhUWuc4aQgzp4CseJrqRFtwhI7Ru_aiF5YmGpeJliEfunVx3BUNBHhbW26ywCrc5KsOK7P6jIGB4pa07Xow4n6Bc4iGzg26UwopHhG7v98COBN5vnwB35U=)
29. [ikatakos.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFymEw5gxFVDSC15kgcSH9wG3OWPKUwy54yJioSRCDMZe_PyOEe9jVBV27uhwcnc8wiKyDe7djCkr0dE7-Qt2HiGD1xX1U9Hx6N4lgw8aLDLs_fzBa2y8pytwnUFJ62O77CWNDNbSWXv7sFHuMTWTDggYvqTV5El3M=)
30. [hatenablog.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHyuMsVyk85p02Bw1tRN-8U8BKkiKxEfEirCvXuqViErjaFSv8TFMwX9gcO7cFTbqSalqbRU44uw3Codcq9PxZndFdPmYF1HP5yS-3pMcDRXfG1HtT0--uZSoIOhYwaP4YQgE5ZBS7kvCiIjrVy)
31. [note.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQG2UOFw7noDU86DmxFktAOR5TRhzun_E4V5KkZBWwtLr1fKfvyymbc1iAYPVGqqRmoJ-OGtQuIt7AGXkKneNB63_SycyxX7czqguzP4K49_0Ho_8-sW5w_N1_q8846Diw==)
32. [hatenablog.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQE8k9CHGgyiYwR9XgaolV-WMAqBjSJKL_yD8EZ3bYbkNqclcU3E5oqaok2tiVR01T8W8jJrD_kKh2_tV-CPOMBAPOYEuFioyUG92oDyxzJn9JXyUyQ5T5NTP-u8skcSVv3ZaHVqUUvosAaXS99kpSvCScpjiYhtrDc=)
33. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFyf7dYFf2bJgU6Rkyj07-p5XMERrj-icNFK22WqyLcHox4easSVPuxhJ22mSlTZtN0DJPiZQCTWsoUzuOuqHuoXkaX6X0VN0fBh7DJHqywHCth3zBy7mdtk3dLgXmw3-YT8r9xG-C9)
34. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHI4-nvo2rO0RbZ4s9PIuRvENHqHhNW48B1WSWcfD2SZx4jd5c3lioOb5jvvOJcAkGrDpr-099c1MrrmJZR9_DC5j9h7Hb3vTqFkbJmY-G0jbe1RWpzs9j0SHBk1ntOcInkYL35Gg60Aqk=)
35. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQEJy6ZW3DPhuhyFbuzwJDvoYxwgylA57BfJkmYztwTGKaioeEanGidGTQ-AWtzb8ze6SyMgMYHKU6NrU3Tae22Dnb-uJauOaApl_NVUtVl69443EtKP4JvQ_jKDbgGAIbmkq_ArbT-HgQ==)
36. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQGZA9xP819yYhyUmx5SQ_Q8E68RkNNtSVq0qjnNy759SI9F4gecrEaeHHYIErKMooHMLC3Gm28-Y2gSFFzvwr7iElfLRFM0Q0Ws-pHegyBYwqudecn8Ue0iRb8--wW1-8GbFNbRXgQfpQHWCw==)
37. [github.io](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFyd0llAJuxr42dY5U_AXioS3q6L16Ddk7EW7Ujt8Z8TwA7lcB7k7a5mAB2IO_RagXkvFNwgi3LuAYSiUOxY9oDP1SPfGs9WnwkWhPF7InP40LFkcvUejGu4MbrS2SY9-Oq30Yxj7DoKKmZeyc3U1bsNkpkhiYcblLMY75vdFwjD7KtlNnrEIot644=)
38. [note.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQFABR2yYfm0YUQo4fv6vssPufY1hfY8PR7ADk93ouneRf5YJc8_jw9l_FVzrd59XPNnTu7Phd0zqarvZB-QyKbw6vnZAPUT9eqt0DlnbW9nBzOS3f2O-vEl2dcQZVN1yqpr497v)
39. [qiita.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHTm6BzitZCFTQIAYGaXgS3FYfLg2LwJFCRLO4ZqQyGUZ30-sumLeW3XLqDAQG2_ewM2XcosP34WAfy1XoltaV7hTphu5d7XUm1mgNPRGK8aCugmtaPVYcUO4QziL5aLpAofXsxjII9eIj-)
40. [wantedly.com](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHznA0cQ8eUDYWvnyIayHRXRP3J_AGrCIO_8JJyIKKNvOFC7GImEVAAoUc1MgUQngxYgBVsrk6ionSB51HfCtNMzeXckGsyd1Ll9LjQOncoP4vxlaPdFQmmwMHrurf8kvt4ZVT8lDmSgaSNRdzMU2qvpJHO_DCg)
41. [algo-logic.info](https://vertexaisearch.cloud.google.com/grounding-api-redirect/AUZIYQHkMBLa8pgbhQjFa5vQ8VGzC6izK821k0-yMWlYsiyCARwdKicDzkjvXnd_g81OPnwR0WSzq64Pa6ZGyy8eevLHdh71M5RoOtCkeqlVAvw2982h)