---
title: "VoiceflowのAIアシスト機能を試してみた②Freestyle"
emoji: "🗣️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Voiceflow","LLM","VUI","チャットボット"]
published: false
---

## 「AIアシスト機能」とは？

スマートスピーカーやチャットボットなどの会話フローをGUIで作成できるサービス[Voiceflow](https://voiceflow.com/)に新しく「AIアシスト機能」がリリースされました。

AIアシスト機能は、かんたんにいうとLLM(Large Language Model)による会話モデル設計を支援する機能です。以下の2つの機能があります。

- Generative Tasks
- Freestyle

今回はFreestyleについて試してみます。

Generative Tasksについては前回の記事をご覧ください。
https://zenn.dev/kun432/articles/20230125-voiceflow-ai-assist-1

:::message
AIアシスト機能はベータリリースです。機能の内容等については今後変更される可能性があります。予めご留意ください。
:::

:::message
Freestyleは実験的な機能です。LLMによるレスポンスの生成は間違ったレスポンスを返す可能性があります。現時点では商用レベルでの使用は推奨されていません。
:::

### Freestyleでできること

**Freestyle**は、用意しておいたインテントにマッチしない場合、LLMにすべてお任せでレスポンスを自動生成します。これがどういうことなのか？実際にプロジェクトを使って確認してみましょう。

### プロジェクトの作成

前回と同じものを使ってもいいのですが、故あって新しく作成します。作成内容は前回と同じで。プロジェクト名は適当に変えて下さい。

![create assistant](/images/20230130_01.png)

### Freestyleの有効化

まず最初に、Free Styleはデフォルトで有効化されていませんので、これを有効化します。

左のメニューから"Settings"をクリックします。

![create assistant](/images/20230130_02.png =500x)

下にスクロールして、AI assistの箇所でFreestyleを有効化します。

![create assistant](/images/20230130_04.png)

免責事項を確認して同意します。これでOKです。なぜ免責事項への同意が求められるかは後述します。

![create assistant](/images/20230130_05.png =400x)

キャンバス画面に戻りましょう。

![create assistant](/images/20230130_06.png =500x)

### Freestyleを試してみる

Voiceflowでは"Choice"を使ってインテントごとに会話フローを分岐しますが、用意しておいたインテントのどれにもマッチしない場合は通常"No Match"で処理を行います。Freestyleはこの"No Match"と同じところで実行されます。

前回の記事では"No Match"用のレスポンスもGenerative Tasksで生成できることを説明しましたが、これが設定されているとFreestyleは動作しませんので削除します。

![create assistant](/images/20230130_03.png =550x)

これだけです。あと、この後のテストがわかりやすいように、前回作成したorder_drinkインテントも削除してしまいましょう。

:::message

- インテントが1つもないというのはあまり実践的な例ではありませんが、デモのため意図的にこうしています。
- というのは、使用しているNLUやインテント/サンプル発話の数にもよりますが、インテントベースのNLUではなるべくfallbackを避けて既存のインテントに紐付けようとするNLUが多いと個人的には感じています。つまり、どのインテントのサンプル発話にマッチしないような発話でもfallbackされずに既存のインテントで誤って処理される可能性があるということです。
- 意図的にfallbackさせるというのは結構難しいです。。。

:::

## まとめ
