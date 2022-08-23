# 利用ガイド

これは [av3-animator-as-code](https://github.com/hai-vr/av3-animator-as-code) を VRChat World SDK で使えるように実験的に改造したものです。

通常、Unity でのアニメーションの作成は Unity の UI 操作で行いますが、このツールを使うと C# プログラムコードとしてアニメーションを記述することができます。

テキストでの記述になるので以下の利点があります。

- 多数の遷移を持つアニメータを作るのが楽になる（GUI での作業では、抜け漏れのミスが起きやすい）
- 同じようだが少しだけ異なるアニメーションを多数作る場合に、より簡単に作れる
- 後から修正がしやすい

このフレームワーク "animator as code" の事を以下では短く AAC と表記します。


# Quick Start Guide

## 導入

1. [GitHub の リポジトリー](https://github.com/naqtn/world-sdk-animator-as-code) のページを開く
2. 右上の「Code」から「Download ZIP」を選び zip ファイルを入手。（GitHub をよく知っている人はお好きな方法で）
3. VRChat world 開発の通常の方法でプロジェクトを用意。
4. プロジェクトの Asset フォルダの中に適当な名前でフォルダを作って ZIP ファイルを展開する。

## Examples の見方（ないしは、使い方の簡単な説明）

基本的な使用例が Examples フォルダの中に置かれています。これをどこから見ていけばいいのかを、簡単にガイドします。

### サンプルシーンの構成

* Examples フォルダにあるシーン（`WorldAnimatorAsCodeExampleScene.unity`）を Unity で 開きます。
* `AnimatorHolder` 以下の GameObject が、作成しているワールドの一部だと思ってください。
  - ここには `Animator` コンポーネントが置かれていてコントローラとして `WorldAacExampleController` が設定されています。
  - この Examples では、このコントローラにアニメーションを設定します。
* Animator ウィンドウで `WorldAacExampleController` を開いてみてください。
  - 配布状態では、すでにプログラムコードから生成されたアニメーションがコントローラに対して設定された後の状態になっています。
  - `Example 0` をはじめとして、幾つかのレイヤーが定義されています。
  - またパラメタを開くと、いくつかのパラメタが定義されているのが確認できます。
* AAC の仕組みは `Examples` GameObject 以下に配置されています。
  - 子供の GameObject 一つ一つが、個別の使用例を示しています。また、これは前ステップで見たレイヤーと対応付いています。
    `Example0` GameObject の下に書いてある内容に従って生成されたものが `Example 0` レイヤーに収められます。
  - （ここの名前は分かりやすいように一致させているもので、GameObject の名前がレイヤーになるわけではありません）
  - `Example0` GameObject にはコンポーネント `GenExample0_ToggleGo` が置かれています。
  - この名前は「生成例０番：GameObject をトグルする」を意味していて、この例のために作られたコンポーネントです。（AAC フレームワークの一部ではありません）
* Animator ウィンドウでレイヤー定義を表示しつつ Inspector で `Example0` GameObject を開いて、
   `GenExample0_ToggleGo` コンポーネントの `Remove` ボタンを押してみてください。
  - これでレイヤー定義が消えます。
  - 続いて `Create` ボタンを押してみてください。レイヤー定義が追加されます。
  - （及びアニメーションパラメタとアニメーションクリップも合わせて生成・設定されています）

### サンプルプログラムの構成

* 次にプログラムコードを見ていきましょう。
  - `GenExample0_ToggleGo.cs` を開いてください。
  - 先頭の `#define WORLD_AAC` は、この world SDK 用の改造部分を記述するためのものものです。
  （これが定義されていない場合に有効になる箇所が、オリジナルの av3-animator-as-code での使用方法になります。）
* 見進めて（using や namespace などは飛ばして）まず最初の `public class GenExample0_ToggleGo : MonoBehaviour` が先ほど Inspector で見ていたコンポーネントです。
  - このコンポーネントは Inspector で値を設定するための public なメンバー変数を定義しています。
  このコンポーネントはそれだけのために定義されていて、より実質的な内容は次に続く custom editor に書かれています
* （以下掲載するコード片は、見やすい様にコメントを取り除くなどの若干の加工をしています）

```csharp
    public class GenExample0_ToggleGo : MonoBehaviour
    {
        public Animator animator;
        public AnimatorController assetContainer;
        public string assetKey;
        public GameObject item;
    }
```

* 次に続く `public class GenExample0_ToggleGoEditor : Editor` が実質的な AAC の利用プログラムコードになっています。
  - ただし使用例プログラムの共通部分は `AacExample.cs` に書かれているので、それと合わせて見る必要があります。
  - `GenExample0_ToggleGo.cs` の `public override void OnInspectorGUI()` は Inspector が画面を描画する際に呼び出すメソッドです。
  ここから実質的な動作が始まります。
  - （Inspector のためのプログラムの書き方はこの文書では解説しません。このサンプルをそのまま真似れば良いでしょう）
  - `AacExample.InspectorTemplate` の引数の `Create` と `Remove` はこのクラスで定義しているメソッドです。
  それぞれボタンが押された時に呼び出されます。
  - AAC におけるアニメーションの定義の仕方は、その `Create()` メソッドの中に書いてあります。

```csharp
        public override void OnInspectorGUI()
        {
            AacExample.InspectorTemplate(this, serializedObject, "assetKey", Create, Remove);
        }

        private void Create()
        {
```


### 初期化パラメタ

```csharp
        private void Create()
        {
            var my = (GenExample0_ToggleGo) target;
            var aac = AacExample.AnimatorAsCode(SystemName, my.animator, my.assetContainer, my.assetKey, AacExample.Options().WriteDefaultsOff());
```

* では、その `Create()` メソッドを見ていきます
* まず冒頭では `target` を、この example でのコンポーネント `GenExample0_ToggleGo` にキャストしています。設定値がここから取り出せるようにしています。
* 次に `AacExample.AnimatorAsCode` を呼び出して AAC 利用の入り口となる [AacFlBase 型](README-original.md#base-aacflbase) のインスタンスを得ます。
  - `AnimatorAsCode` メソッドにはオーバーロードがいくつかありますが、いずれにせよ `AacV0.Create` で `AacFlBase` を得ます。
  - この部分について、Examples を読んでいる今の段階で気に留めておくポイントは、初期化パラメータです。
    詳しくはオリジナルの README の [初期化の構成要素はここに](README-original.md#declare-an-animator-as-code-aac) 書いてあります。
    AAC を利用する際の構成を理解するのに必要なので、以下簡単に説明します。
* `system name` : これから記述する複数のアニメーションから構成される“システム”の名前。
  - 生成されるレイヤーや animation clip の名前などに使われます。他の記述と重複しない文字列を指定します。
  - `GenExample0_ToggleGo.cs` では `GenExample0_ToggleGoEditor.SystemName` で定義した値を使うようになっています。
* `Animator` : AAC で記述するアニメーションを設定する対象の Animator コンポーネント。
  - `GenExample0_ToggleGo.cs` では、`GenExample0_ToggleGo` コンポーネントに対して Inspector で指定したものを使うようになっています。
  - この Animator に対して設定されている Animator Controller アセットに、生成したものが書き込まれます。（クリップについては後述 `asset container` 参照）
  - （オリジナルの AAC ではここは `Animator` ではなく複数の Animator Controller を保持する `AvatarDescriptor` になっています。）
* `animator root` : アニメーションに書かれる GameObject の相対パスの基準となる root の Transform。
  - `GenExample0_ToggleGo.cs` では、前記 Animator の GameObject が使われるようになっています。
* `default value root` : アニメーションされるパラメタのデフォルト値を取得する root となる GameObject の Transform。
  - 現在 （2022/8）の AACの実装では定義だけされていて使われていません。なお、ここでのデフォルトは WriteDefault でのデフォルトの意味だと思われます。
* `asset container` : AAC で生成されたアニメーションクリップを収めるアセット。
  - `GenExample0_ToggleGo.cs` では、`GenExample0_ToggleGo` コンポーネントに対して Inspector で指定したものを使うようになっています。
  - `Animator Controller` 型の空のアセットを造り指定します。
  - AAC でレイヤーなどを生成した状態で（つまり `Create` ボタンを押した後） `Examples/WorldAacExampleContainer` を Project ウィンドウで開いてみると、
  サブアセットとして内部にアニメーションクリップが生成されているのを観察できます。
* `asset key`: AAC で自動生成されるアセットの命名に使われる文字列。他と重複がない値を設定するようにします。
  - `GenExample0_ToggleGo.cs` では、Inspector で指定していればそれを、そうでなければ（つまり空欄だったら）ランダムな文字列を使うようになっています。
* `defaults provider` : AAC が生成する各種アニメーション要素（例えば animation state）の設定値のデフォルトを与えるもの。
  - `GenExample0_ToggleGo.cs` ではフレームワークの [AacDefaultsProvider.cs](Framework/Editor/V0/AacDefaultsProvider.cs) の実装が使われます。
  - （WriteDefaults の設定だけは容易に変更できる仕組みがあり off を指定しています。）


### アニメーションの記述方法

* コードを見る前に、この `GenExample0_ToggleGo.cs` で作ろうとしているアニメーションの内容について説明しておきます。
  - 二つのステート "Hidden" と "Shown" を持つ。
  - "Shown" にいる間、アイテム GameObject を active にする。
  - "EnableItem" というパラメタで、二つのステートの遷移を制御する。
* では `Create()` メソッドの続きを見ていきます。

```csharp
            var fx = aac.CreateMainLayer();

            var hidden = fx.NewState("Hidden")
                .WithAnimation(aac.NewClip().Toggling(my.item, false));
            var shown = fx.NewState("Shown")
                .WithAnimation(aac.NewClip().Toggling(my.item, true));

            var itemParam = fx.BoolParameter("EnableItem");

            hidden.TransitionsTo(shown).When(itemParam.IsTrue());
            shown.TransitionsTo(hidden).When(itemParam.IsFalse());
```

* 前節で述べた `AacFlBase` インスタンスの `CreateMainLayer()` メソッドを呼び出し、レイヤー定義のためのオブジェクトを得ます。型は [AacFlLayer](README-original.md#layer-aacfllayer) です。
  - （オリジナルの AAC のサンプルではここは、アバターの FX レイヤーの Animator Controller を対象とするように書かれています。）
  - レイヤー名は引数ではなく、前節で説明した初期化時の `system name` によって決まるようになっています。
* `NewState(string)` メソッドで animation state を追加します。
  - `NewState` メソッドは animation state の定義を書き込む [AacFlState](README-original.md#state-aacflstate) 型のオブジェクトを返します。
  - そのまま続けて state のアニメーションクリップを `WithAnimation` メソッドで設定しています。
* アニメーションクリップの定義は `AacFlBase` の `NewClip()` メソッドで得た [AacFlClip](README-original.md#clip-aacflclip) に対して行います。
  - ここでは `Toggling` メソッドによって、GameObject の active を変化させるアニメーションを定義しています。
  - 対象となる GameObject は、コンポーネントの public メンバー変数に定義して、Inspector で設定したもの（`my.item`）になっています。
* 最期にステートの遷移を定義します。
  - まず [AacFlLayer](README-original.md#layer-aacfllayer) の `BoolParameter(string)` メソッドでアニメーションパラメタについて記述するオブジェクトを得ます。
  - `AacFlState` の [TransitionsTo](README-original.md#transitions) メソッドで遷移を記述するオブジェクトを得て `When` メソッドで遷移をするパラメタの条件を記述しています。
  - 遷移の定義は色々なバリエーションがあって複雑なので、オリジナルの README での[遷移についての記載](README-original.md#create-transitions-and-define-conditions) を参照するのをお勧めします。 
* （アニメーションの定義のためのメソッドはほかにも多数あります。詳しくは [オリジナルの README](README-original.md) を参照してください。）

### 定義の削除
* Inspector 上の `GenExample0_ToggleGo` コンポーネントの `Remove` ボタンを押すと、`GenExample0_ToggleGo.cs` に書かれている `Remove` メソッドが動作します。
  - `Remove` メソッド では `RemoveAllMainLayers` メソッドを呼び出して、関連するレイヤーを削除しています。
  - （パラメタ定義は残ります。パラメタは `system name` で区別されるシステムの境界を越えて共通に利用されるので削除できないためだと思われます。）
