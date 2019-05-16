<style type="text/css">
<!--
.reveal h2 {
    text-transform: none;
    font-size: 42px;
}
.reveal p { font-size: 30px; }
.reveal ul li { font-size: 30px; }
-->
</style>

## Elmで制約プログラミングを学ぼう

---

## 自己紹介

ABAB↑↓BA
@ababaupdownba

- ドラクエ大好き
- マイナー言語好き
- WEBエンジニア

---

## 制約プログラミングを学ぶと何が良いのか？

- 自由に対する責任が何かを考えなくて良くなる
- フィードバックを早くし、改善のループを早く回せるようになる
- 面倒なことに対して、楽ができる

---

## 制約と婚活パーティー

制約がない婚活パーティーにはどんな問題があるか考えてみましょう

## 制約と婚活パーティー

- 年齢差がありすぎる
- 異性がいない
- 既婚者がいる

---

## 自由 <-> 制約

- 制約の反対は自由
- 自由は、自由が持つ問題を考えなければならない
- 制約の先に本当の自由がある

---

## 制約とプログラミング

プログラミングにおける制約とはなにか考えてみよう

---

## Elmの制約

これらの制約嬉しい？

- プラットフォームの制約 -> WEBフロントエンドしか書けない
- エラーハンドリングの制約 -> 例外を起こせない
- 状態の制約 -> アプリケーション全体で一つの状態しか持てない
- 構文の制約 -> 何かを書きたいときの構文が定まっている
- フレームワークの制約 -> The Elm Architecture

---

## 制約とフィードバック

- プログラミングは異常な振る舞いついて、考えなければならない
- 異常な振る舞いに対するフィードバックは早ければ早いほど、改善のループは早くなる
- フィードバックの種類はいくつか種類がある

フィードバックの種類を上げてみよう

---


## HTMLの問題

この「1」はなんでしょう？ 答えは後で発表します。

```HTML
<div id="a">1</div>
```

---

## HTML表現の自由と危険

問題です。画面には何と表示される？

```HTML
<body>
  <div id="a">1</div>

  <script>
    const a = document.getElementById("a");
    const value = a.innerHTML;
    a.innerText = value + 1;
  </script>
</body>
```

---

## HTML表現の自由と危険 ~こたえ~

「1」は、**DOMString**(文字列)です。```'1' + 1 = '11'```となります。

![](https://i.imgur.com/filZnbM.png)

コラム
[データ型の変換](https://developer.mozilla.org/ja/docs/Web/JavaScript/Guide/Grammar_and_types#Data_type_conversion)

---

## ElmのHTMLの制約

[(+)](https://package.elm-lang.org/packages/elm/core/latest/Basics#(+))は関数です。引数には文字列を取ることができず**コンパイルエラー**になります。これにより文字列と数値がどういう結果になるか、という**認識のズレ**が解消されます。


```elm
> "1" + 1
-- TYPE MISMATCH ----------------------------------------------------------- elm

I cannot do addition with String values like this one:

4|   "1" + 1
     ^^^
The (+) operator only works with Int and Float values.
```

---

## ElmのHTMLの制約

[text関数](https://package.elm-lang.org/packages/elm/html/latest/Html#text)は、**String**型しか受け取りません。これにより、Htmlの要素としてのtextは文字列であるという**認識のズレ**が解消されます。

```elm
> Html.text 1
-- TYPE MISMATCH ----------------------------------------------------------- elm

The 1st argument to `text` is not what I expect:

5|   Html.text 1
               ^
This argument is a number of type:

    number

But `text` needs the 1st argument to be:

    String
```

---

## ElmのHTML制約

数値は[String.fromInt](https://package.elm-lang.org/packages/elm/core/latest/String#fromInt)関数で文字列に変換してから、text関数に渡します。認識のズレが最小限に押さえられた状態で計算が行なえます。また、やりたいことに対する書き方が統一されます。

```elm
Html.text (String.fromInt (1 + 1))
```

コラム: 流れるようにパイプで括弧を減らそう
```elm
text <| String.fromInt <| 1 + 1
```

---

## ElmのHTML制約まとめ

- 型とコンパイラの制約により機械的に、認識のズレを無くす
- 強い制約は親切なエラーを出す

---

## 制約とテスタビリティ

ふだん自動テストを書いていますか？ 

---

## なぜ自動テストをするのか

- 生きているプログラムの振る舞い(仕様)を永久的に残すことができる
- プログラム変更時に高速で振る舞いが正しいかフィードバックをくれる
- 改善のループが回せる

---

## なぜ自動テストが難しいのか

ふだん自動テストが書けていないとするならば、「振る舞い」と「ロジック」、「状態の書き換え」が上手く分離できていないことが問題になっています。

この分離が中々難しい

---

## カウンタアプリの振る舞い

分離が出来ていない例

+ボタンを押すと、カウンタ(0)が1インクリメントされる。

![](https://i.imgur.com/sux1Cy9.png)

分離が出来ていないことに対する問題点

+ボタンを押すことは実行して かつ 押さなければ、確認が出来ない。カウンタの「状態」がわからなければ次いくつの数値になるかわからない。

実行しなければわからない -> フィードバックが遅い -> 改善のループの遅れ

---

## カウンタ問題

振る舞いを考える

- カウンタの初期値は「0」とする
- ~~+ボタンを押す~~ インクリメント時には、*Increment*というお手紙(Msg)を発行する
- 更新関数(update)は、「Msg」と「現在のカウンタの値」を受け取り、新しいカウンタの値を返す

[https://gist.github.com/ababup1192/843f7a30e1cc39c24bff599cfd7b50de](全体)

---

## カウンタ問題

分離が上手くいけば、まるで振る舞いをテストできる。

```elm
updateTest : Test
updateTest =
    describe "updateのテスト" <|
        [ test "0のとき Incrementされると 1になる" <|
            \() ->
                update Increment 0
                    |> Tuple.first
                    |> Expect.equal 1
        , test "1000のとき Incrementされると 1001になる" <|
            \() ->
                update Increment 1000
                    |> Tuple.first
                    |> Expect.equal 1001
        ]
```

---

## カウンタ問題

人間にできない(とても時間の掛かる)振る舞いのテストも簡単にできる

```elm
updateTestHelper : Int -> Int -> Int
updateTestHelper incCount crnt =
    if incCount == 0 then
        crnt

    else
        updateTestHelper (incCount - 1) (update Increment crnt |> Tuple.first)

updateTest : Test
updateTest =
    describe "updateのテスト" <|
        [ test "0のとき 1000回、Incrementされると 1000になる" <|
            \() ->
                updateTestHelper 1000 0
                    |> Expect.equal 1000
        ]
```

---

## カウンタ問題

アプリケーションの状態は**Model**と呼ばれる。`Model == カウンタ`として今回は定義する。

```elm
type alias Model =
    Int


init : () -> ( Model, Cmd Msg )
init _ =
    ( 0, Cmd.none )
```

---

## カウンタ問題

viewはModelを用いて組み立てられる。onClickイベントでは、modelを書き換えることはしない。*Increment*を発行するだけ。

```elm
view : Model -> Html Msg
view model =
    div [ class "container" ]
        [ button [ onClick Increment ] [ text "+" ]
        , p [] [ text <| String.fromInt model ]
        ]
```

---

## カウンタ問題

*Incement*を受け取り、modelを1足したものを返す。

```elm
update : Msg -> Model -> ( Model, Cmd Msg )
update msg model =
    case msg of
        Increment ->
            ( model + 1, Cmd.none )
```

---

## カウンタ演習問題

- JavaScriptバージョンのカウンタの改善をしてみよう
- デクリメントバージョンのカウンタを実装してみよう

ヒント

```elm
type Msg = Increment | Decrement
```

---

## まとめ


- 自動テストをすることにより、振る舞い(仕様)を機械的にチェックする
- 実行することをそもそも無くし、フィードバックのループを早くする
- 振る舞いの適切な分離を覚えることで、テストを書きやすくする

---

# 異常に対する型

- ユーザは常に期待通りの動きをしない
- 特に入力は異常を含めて考えなければならない
- 異常を型で表せれば...？

---

# 足し算問題

異常な振る舞いはいくつあるでしょうか？

[スクショ]

![](https://i.imgur.com/xvVy2i7.png)


https://gist.github.com/ababup1192/1397eb47add84390ff634f8c6429e637

---

# 足し算問題

- 左が数値ではない場合
- 右が数値ではない場合
- 左右が数値ではない場合

---

# 足し算問題 UI部分

```html
<div>
    <input id="left" value="0">
    +
    <input id="right" value="0">
</div>
<p id="result">0</p>
<button id="calc">calc</button>
```

---

# 足し算問題 スクリプト

文字 -> 数値 に変換。変換出来ないケースを考慮しなければならない。

```javascript=
const left = document.getElementById("left");
const right = document.getElementById("right");
const calc = document.getElementById("calc");

calc.addEventListener('click', (e) => {
  const leftValue = parseInt(left.value);
  const rightValue = parseInt(right.value);

  if (!Number.isNaN(leftValue) && !Number.isNaN(rightValue)) {
    result.innerHTML = String(leftValue + rightValue);
  } else {
    result.innerHTML = '数値だけを入力してください。';
  }
});
```

---

# 足し算問題 

```elm=
type alias Model =
    { left : String
    , right : String
    , result : String
    }


init : () -> ( Model, Cmd Msg )
init _ =
    ( { left = "0"
      , right = "0"
      , result = "0"
      }
    , Cmd.none
    )
```

---

# 足し算問題

```elm=
view : Model -> Html Msg
view { left, right, result } =
    div [ class "container" ]
        [ div []
            [ input [ value left, onInput UpdateLeft ] []
            , text "+"
            , input [ value right, onInput UpdateRight ] []
            ]
        , p [] [ text result ]
        , button [ onClick Calc ] [ text "calc" ]
        ]
```

---

# 足し算問題

[String.toInt](https://package.elm-lang.org/packages/elm/core/latest/String#toInt)は、成功と失敗 ２つの文脈を持つ型 Maybe型を返す。

```elm=
type Msg
    = UpdateLeft String
    | UpdateRight String
    | Calc


update : Msg -> Model -> ( Model, Cmd Msg )
update msg ({ left, right } as model) =
    case msg of
        UpdateLeft l ->
            ( { model | left = l }, Cmd.none )

        UpdateRight r ->
            ( { model | right = r }, Cmd.none )

        Calc ->
            case ( String.toInt left, String.toInt right ) of
                ( Just lValue, Just rValue ) ->
                    ( { model | result = String.fromInt <| lValue + rValue }, Cmd.none )

                ( _, _ ) ->
                    ( { model | result = "数値だけを入力してください。" }, Cmd.none )
```

---

# Maybe型とパターンマッチ

```elm=
type Maybe a
    = Just a
    | Nothing
```

パターンマッチは分岐だけでなく、構造の分解により値を直接取り出し、値も返す。

```elm=
case (String.toInt "1") of
    Just n -> n
    None -> -1
```

---

# まとめ

- 異常を型で表すことにより、考慮漏れが無くなる
- パターンマッチ(構造の分解)により、面倒な判定と値の取り出しが一度にできる

---

# パスワード一致問題

viewに対するテストもね！

https://gist.github.com/ababup1192/528ba858d1945f818a627d84a227fd9d

---

# Summary

01. 制約は本当の自由のために
02. 振る舞いとロジックを分離
03. 異常に対する型