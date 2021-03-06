第3回トレーニングで紹介したイベント・ソーシング設計パターンは永続化層への書き込みパフォーマンスを維持しつつ、

- 耐障害性
- アプリケーション層と永続化層の疎結合
- 履歴の保持によるトラブルシューティングの容易性

などを実現しました。しかし、イベント・ソーシングだけでは不便なこともあります。それはデータ読み込み側の処理です。

ここでまた伝統的な3層アーキテクチャと対比して考えましょう。このアーキテクチャではアプリケーション層をステートレスに保つため、状態をデータベースに全て退避するのでした。
データベースは書き込み時に最新の状態をテーブルに常に保持するので、データ読込処理はこれらのテーブルから抽出・集計できます。
リレーショナル・データベースのSELECT文はまさにこの用途に適しています。

<p align="center">
  <img width=640 src="https://user-images.githubusercontent.com/7414320/80338283-e663e600-8896-11ea-8a52-8e85d366d35b.png">
</p>

一方イベント・ソーシング「だけ」を採用したアプリケーションでは、永続化層はイベント履歴のみを保存するので、
データの最新状態を永続化層のデータベースだけで取得することは出来ません。
データの最新状態を読み込むにはアクターにメッセージを投げ、アクターが最新状態を含むメッセージを投げ返す必要があります。
これでは抽出や集計の際に困ってしまいます。データベースと違い、アクターは大量のデータから抽出、集計するのに向いていないからです。

<p align="center">
  <img width=640 src="https://user-images.githubusercontent.com/7414320/80296108-42553e80-87b3-11ea-851e-9a04cb638510.png">
</p>
<p align="center">
  <img width=640 src="https://user-images.githubusercontent.com/7414320/80296107-41bca800-87b3-11ea-8006-5271a1f8f527.png">
</p>

この問題への対処法は書き込み側はイベント・ソーシングを、読み込み側は別の設計を用いることです。
書き込み側(Command)と読み込み側(Read)に別々の設計パターンを用いると、CQRS - Command Query Responsibility Separationと呼ばれるパターンを実現できます。

<p align="center">
  <img width=640 src="https://user-images.githubusercontent.com/7414320/80296014-a3c8dd80-87b2-11ea-962e-a031e0f99248.png">
</p>

CQRSが書き込み側と読み込み側を疎結合にすることで生まれるメリットは、以下のMicrosoftの記事がよくまとめているので引用します。

[コマンド クエリ責務分離 (CQRS) パターン - Microsoft Azure](https://docs.microsoft.com/ja-jp/azure/architecture/patterns/cqrs) :

> - 読み取りと書き込みのワークロードが不均衡になりやすいため、パフォーマンスやスケールの要件が大きく異なってくる可能性があります。
> - 読み取りと書き込みのデータ表現が一致しないことがよくあります。具体的には、操作の一部としては必要ないものの、正しく更新しなければならない追加の列やプロパティなどです。

CQRSは大掛かりな設計パターンです。実装や運用の難易度が高いので、CRUDで十分な単純なシステムに適用するには向いていません。
あなたが作るシステムにCQRSが必要化は慎重に見極めてください。
