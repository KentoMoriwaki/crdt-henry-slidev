---
# try also 'default' to start simple
theme: seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Slidev Starter Template
  Presentation slides for developers.

  Learn more at [Sli.dev](https://sli.dev)
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

# 分かった気になる<br/>CRDT による共同編集

Presentation slides for developers

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    Press Space for next page <carbon:arrow-right class="inline"/>
  </span>
</div>

<div class="abs-br m-6 flex gap-2">
  <button @click="$slidev.nav.openInEditor()" title="Open in Editor" class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon:edit />
  </button>
  <a href="https://github.com/slidevjs/slidev" target="_blank" alt="GitHub"
    class="text-xl icon-btn opacity-50 !border-none !hover:text-white">
    <carbon-logo-github />
  </a>
</div>

<!--
The last comment block of each slide will be treated as slide notes. It will be visible and editable in Presenter Mode along with the slide. [Read more in the docs](https://sli.dev/guide/syntax.html#notes)
-->

---

# 背景とゴール

- アプリケーションを開発していると「この機能を共同編集にしたいな」っていう場面はよくある。
- 「でも作ったことないし、大変そうだな」とハードルが高くて、優先度を上げられない。
- 確かに簡単ではないが、心理的なハードルも大きい。

<br>
<br>

## なんか分かった気になって「自分も作れそう」と思ってもらうことがゴール

- WYSIWYG エディタを対象にした話ですが、他のアプリケーションにも適用できます
- サーバーサイドの話は今回はありません

<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---
layout: image-right
image: https://source.unsplash.com/collection/94734566/1920x1080
---

# 今作っているもの

Henry では、Web ベースの電子カルテを作っています。

- 多機能なブロックの埋め込み
- 編集履歴
- 複数人での共同編集
- Draft.js からの移行

---

# OT と CRDT: データの一貫性を保つ仕組み

共同編集の難しい点は、変更がコンフリクトしたときに、どうやって一貫性を担保するか。

（複数のユーザーが同時に同じ箇所に文字を打ったとき、ユーザーによって表示が異ならないようにする）

## OT と CRDT という二つの仕組みが有名

---

# OT

プレーンテキストを共同編集するのがベース。

簡単にいうと、何文字目にどういう操作をしたのかを蓄積して、この状態の 3 文字目ってことは、今の状態だと 4 文字目 ってことだな、というのを計算する方法。

---

# CRDT

コンフリクトしないように設計されたデータをもとにした技術。 

各 client がデータのコピーを持ってそれを編集し、それらをコンフリクトなく同期することができる。

CRDT にもいくつか派生があり、ここでは Yjs が用いている CvRDT の話。

---

# CRDT のデータ構造

エディタでは、文字列を操作しているように見えるが、実はその裏にある木構造を操作している。

この木構造がコアで、それぞれの要素が　Parent / Left / Right を持つ。

それらの操作を、クライアントごとの配列として集めたもの。

全く同じ場所に入れる場合は、clientID が小さい方が勝つ。

ただ文字列を操作しているように見えるが、それぞれの文字に ID が振られていて、裏側の木構造を操作していることを意識する。

---

# CRDT を使った事例

### CodeSandbox

コードエディタ部分は OT で、ファイルシステムなどは CRDT。わかりやすい使い分け。

### Figma

CRDT を使っている。 https://www.figma.com/ja/blog/how-figmas-multiplayer-technology-works/

### JupyterLab

今回紹介する Yjs を使っている

### Redis

地理分散システムに CRDT が使われている

---

# 登場するライブラリの紹介

- Yjs
- ProseMirror
- y-prosemirror
- lib0

---

# Yjs

CRDT の JavaScript 実装

```ts
import * as Y from "yjs";

const ydoc = new Y.Doc();
const map = ydoc.getMap("foo");
map.set("one", "1");
map.set("two", "2");
```

汎用的なデータ構造が用意されている。

使いこなすために知っておいてほしいことを後のページで紹介する。


---

# ProseMirror

WYSIWGY エディタのライブラリ。

エディタライブラリの選定基準は難しいが、筆者の経験値から選定。
何を使っても良いが、Yjs が公式に binding を用意してくれているものが便利。

## Tiptap

ProseMirror を React で使いやすくするための技術。

React Component を埋め込めるようにするために便利だが、 ProseMirror 自体を理解していないと使うのは難しい。

---

# y-prosemirror

Yjs と ProseMirror の binding

自動的に両者の状態を同期してくれるので、基本は ProseMirror 側だけで機能開発すれば良いが、ハマりポイントがある。

両者の操作ごとに相互に変換して適用するのではなくて、差分を検知して埋めるアルゴリズムのため、予期しない形で変更が適用されることがある。

また、ProseMirror と Yjs のデータ構造の都合で、互換性がないパターンがある。（例えば、同じ場所に同じ名前の mark を複数つけることはできない。）

---

# lib0

主に encoding と decoding に使われているライブラリ。効率よくデータを詰め込むために使われる。

ネットワークでやりとりされるデータや、永続化するデータは、これで作られた binary (Uint8Array) なので、パッとみたときにどういうデータがやりとりされているか分からなくて、デバッグに困ることがある。

なので、基本的な使い方と、使われるデータ構造とその意味、その中身の見方を覚えておけば得する。

---

# lib0

```js {all|4|9|all}
import encoding from "lib0/encoding";
import decoding from "lib0/decoding";

const data = ["foo", "bar", "baz"]; // encode したいデータ

const encoder = encoding.createEncoder();
encoding.writeVarUint(encoder, data.length);
for (const str of data) {
  encoding.writeVarString(encoder, str);
}
const message = encoding.toUint8Array(encoder);

const decoder = decoding.createDecoder(message);
const length = decoding.readVarUint(decoder);
for (const i = 0; i < length; i++) {
  const str = decoding.readVarString(decoder);
  assert(str === data[i]);
}
```

---

# 同期プロトコル

Client-Server 方式や、 P2P など幅広い選択肢が取れる。

二つの Client(Server) がある時、両者が同じデータを持っている状態にするプロトコル。

最低限のデータ量だけで同期できるように、両者の差分だけを送り合う仕組みがある。

---

# 同期プロトコルの図


図の簡略化のために一方向だけ書いたが、両者が同時に sync1 を送信する

同期が完了したら、あとはその都度変更を送信しあう。

---

# StateVector

わたしはこれだけの状態を持っています、を表すデータ。

---

# 削除の操作

追記していくだけと書いたが、パフォーマンスの観点から、削除は追記じゃない。

削除された要素に、削除フラグを立てる。

なので、削除されたデータは残り続けるので、Garbage collection する必要がある（後述）。

## DeleteSet の構造

ここからここは削除しました、というデータ。

---

# Snapshot

State の特定の地点を表すデータ。これがあれば、その地点の状態まで戻ることができる。

## StateVector との違い

StateVector は削除された範囲に関する情報を持っていないため、その状態に完全に戻ることはできない。

Snapshot = StateVector + DeleteSet

---

# Garbage Collection

先述した通り、削除の操作は削除フラグを付けるだけで、データはずっと残っている。
それにより Snapshot があれば、完全に特定の状態を再現することができる。

しかしそれだとデータがどんどん肥大化してしまい、転送が遅くなったり、操作が遅くなってしまう。
そこで、削除された要素の中身を消去して圧縮することができる。

```ts
const ydoc = new Y.Doc({ gc: true })
```

処理速度とメモリのトレードオフ

基本方針
- クライアントのようは揮発性の高いデータは、 GC を使わない。もちろんメモリがシビアな場合はやってもよい。
- サーバーサイドでそのまま永続化するようなデータに関しては GC しておかないと、どんどんゴミデータが溜まって肥大化する

---

# Garbage Collection

Snapshot も使いながら、 Garbage Collection するなら、オンデマンド実行がおすすめ。

```ts
function gc(ydoc: Y.Doc): Uint8Array {
  assert(!ydoc.gc);
  const tmpDoc = new Y.Doc({ gc: true });
  Y.applyUpdate(tmpDoc, Y.encodeStateAsUpdate(ydoc));
  return Y.encodeStateAsUpdate(tmpDoc);
}
```

保存する前など任意のタイミングで実行して、元のデータとは別の場所に保存しておく。

---

# Format v1/v2

二種類のメッセージのフォーマットのバージョンが存在している。

内部の Tree の状態などは同じで、ただ単に encode / decode するメッセージのフォーマットが違うだけ。
なので、同じ Doc に v1 と v2 のメッセージを適用しても問題ないが、そのメッセージがどちらのフォーマットなのかは知っておく必要がある。
メッセージ自体も相互変換は可能だが、コストがかかるので、どちらかに合わせる。
デフォルトは v1 なので、v2 の方が高パフォーマンスなので、v2 を使う。
何も考えずにサンプルとか、関連ライブラリを使うと v1 を使うことになるので気をつける。

```js
const v1 = Y.encodeStateAsUpdate(ydoc);
const v2 = Y.encodeStateAsUpdateV2(ydoc);
Y.applyUpdate(ydoc, v1);
Y.applyUpdateV2(ydoc, v2);
```

---

# その他の注意点

ユーザーの一般的な操作に操作に対して最適化されているので、変な操作をするロジックを書くとめっちゃ遅くなることがある。

例えば、一度の transaction で大量の範囲を書き換えるなど。

---

# Yjs の未来

基本的な機能はできているし、パフォーマンスも十分に高速。
特定の操作を高速化する改善などが進んでいる（要素の Move など）。

また多数の繰り返し処理が行われて v8 の最適化に依存して処理速度が大きく変わったりする。

安定して高パフォーマンスが出せたり、別の言語でも binding できるように Rust(WebAssembly)化が進んでいる。

Yjs のメッセージと互換性がある Rust 実装の yrs の開発が進んでいて、より多くの場面で採用できる幅が広がる。

---

# まとめ

自分が CRDT(Yjs) を使った開発を始める前に知っていたかった知識をまとめた

- 単純なテキスト以上の複雑なデータを共同編集したい場合や、 P2P にしたい場合は　CRDT が有力な選択肢になる
- CRDT は Parent / Left / Right への参照を持った木構造への操作を蓄積したもの
- State(Update) / ClientSet / DeleteSet / StateVector / Snapshot などのデータ構造と意味を理解しておく
- ProseMirror <-> Yjs は自動だが、予期せぬ挙動を起こす場合がある。
- Garbage Collection を有効にするかどうかは場面に応じて検討する。
- yrs に期待

---

# 最後に

- サーバーサイドも色々とトピックがあるので、どこかで話したい。
- ぜひお声がけください


---
layout: center
class: text-center
---

# Learn More

[Documentations](https://sli.dev) · [GitHub](https://github.com/slidevjs/slidev) · [Showcases](https://sli.dev/showcases.html)
