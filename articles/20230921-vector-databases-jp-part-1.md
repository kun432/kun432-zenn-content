---
title: "(日本語訳) Vector databases (Part 1): What makes each one different?"
emoji: "🔡"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["vectordb","vectordatabase","vectorsearch","vectorstore"]
published: True
---

## 日本語訳： ベクトルデータベース(パート1): それぞれの違いは？

### 訳者前書き

ベクトルデータベースについていろいろと調査・検証していたところ、以下の記事を見つけて、内容が非常によくまとまっており、多くの人にとっても有用な記事ではないか感じました。もはや翻訳などはDeepLやChatGPTで簡単にできる時代になりつつありますが、まだまだ検索エンジンで検索されることも多いと思いますし、少しでも参照しやすくなればと考えて、作者であるPrashanth Rao氏に許可を頂いた上で日本語に翻訳したものとなっています。

**全4回**の記事の**第1回目**となります。

https://thedataquarry.com/posts/vector-db-1/

Prashanth Rao氏のGitHubアカウント
https://github.com/prrao87

Prashanth Rao氏のTwitterアカウント
https://twitter.com/tech_optimist

:::message alert

**翻訳に関して**

翻訳は、DeepLおよびChatGPTを使用しつつ、日本語としての読みやすさの観点等を踏まえて一部意訳した文章も含まれます。また、ベクトルデータベースを構成する概念については、私の理解も全く十分ではありません。
したがって、翻訳内容に不備がある可能性は否定しません。翻訳上の不備を見つけられた場合はコメントにてご指摘ください。

:::

-----

## データベースをめぐるゴールドラッシュ

2023年上半期、ベクトルデータベースに関するマーケティングが（そして残念ながら誇大広告も）盛んに行われた。これを読んでいるあなたは、なぜこれほど多くの種類が存在するのか？それぞれの違いは何なのか？に興味があるだろう。名目上は、ベクトルデータベースはどれも同じことをする（セマンティック検索を必要とする多くのアプリケーションを実現する）のに、それぞれについてどうやって根拠のある意見を持てるようになるのだろうか？🤔

この記事では、世の中にあるさまざまなベクトルデータベースの違いをできるだけ視覚的にわかりやすく取り上げる。さらに、私が比較したポイントも具体的に挙げて、全体的な理解を深める手助けをする。

### たくさんの選択肢🤯

過去数ヶ月間にわたり、私はさまざまなベクトルデータベースやその内部構造を研究し、またPythonのAPI経由でアクセスしたりしてきたが、以下のような共通の問題点に遭遇した。

1. 各データベースのベンダーは（当然のことではあるが）自社の機能を売り込みながら、同時に競合他社のものを低く見せる。このため、どこを見るかによってあなたの視点にバイアスがかかりやすい。
2. **多数**のベクトルDBのベンダーが存在し、全体像やその背景に存在する基盤技術を理解するためには、複数かつ大量の情報源を読み漁って、点と点を繋げていく必要がある。
3. ベクトル検索に関しては、考慮すべきトレードオフが多数存在する。
    - **ハイブリッドかキーワード検索か？**<br>キーワード検索＋ベクトル検索のハイブリッドが最良の結果をもたらす、これを認識している各ベクトルデータベースのベンダーは、独自のカスタムハイブリッド検索ソリューションを提供している。
    - **オンプレミスかクラウドネイティブか？**<br>多くのベンダーはインフラが世界で最も大きな課題であるかのように「クラウドネイティブ」を強調するが、長期的にはオンプレミスのデプロイメントの方が経済的かつ効果的なことが多い。
    - **オープンソースか？フルマネージドか？**<br>ほとんどのベンダーは基本的な方法論として、ソースが入手可能なこと、またはオープンソースのコードベースの上に成り立っており、パイプラインのデプロイとインフラ部分（フルマネージドSaaSという形）で収益化している。それらの多くをセルフホスティングすることも能だが、それには追加の人手や社内のスキルが必要となる。

私は、異なるベクトルDBベンダーの主要な特徴やマイルストーンを記録するために、[スプレッドシートを使って追跡](https://docs.google.com/spreadsheets/d/1tUxif_MQYByprhFArZ2XqssvlMVfn5mKjsPjYTOO08o/edit?usp=sharing)している。私が見逃しているものも多くありそうなので、コメントで教えてほしい！

## 様々なベクトルデータベースの比較

2023年6月現在、私は8つのベクトル検索特化型データベースと、さらにベクトル検索を追加機能として持つ3つの既存データベースを確認している。

### 本社所在地および資金調達

既存のベンダー（つまり、確立されたベンダー）はさておき、各ベクトルデータベースのスタートアップ各社の資金調達のマイルストーン[^1]を追跡することから始めよう。

|会社|本社所在地|資金調達|
|---|---|---|
|Weaviate|🇳🇱アムステルダム|6800万ドル シリーズB|
|Qdrant|🇩🇪ベルリン|1100万ドル シード|
|Pinecone|🇺🇸サンフランシスコ|13800万ドル シリーズB|
|Milvus/Zilliz|🇨🇳/🇺🇸レッドウッドシティ|11300万ドル シリーズB|
|Chroma|🇺🇸サンフランシスコ|2000万ドル シード|
|LanceDB|🇺🇸サンフランシスコ|ベンチャー|
|Vespa|🇳🇴/🇺🇸インディアナポリス|Yahoo!|
|Vald|🇯🇵東京|Yahoo!Japan|

ベクトルデータベースに関しては、カリフォルニアのベイエリアで明らかに"多く"の活動が行われている！また、資金調達と評価の額には大きなばらつきがあり、データベースの能力と資金調達額に相関関係がないことは明らかである。

### プログラミング言語の選択

高速でレスポンスが良くスケーラブルなデータベースは、近年、GolangやRustのようなモダンな言語で書かれることが一般的である。ベクトル特化ベンダーの中で、Javaで書かれているのはVespaだけである。Chromaは、C++で構築されたOLAPデータベースであるClickhouseとオープンソースのベクトルインデックスである[HNSWLib](https://github.com/nmslib/hnswlib)の上に構築されたPython/TypeScriptラッパーとなっている。

![](https://thedataquarry.com/posts/vector-db-1/vector-db-lang.png)

興味深いことに、Pineconeも[^2]、LanceDBのベースとなるストレージフォーマットであるLance[^3]も、元々はC++で書かれていたにも関わらず、Rustで完全に一から書き直されている。明らかに、データベースコミュニティの多くがRust🦀を受け入れてきている！💪

### 年表

それぞれのベクトルデータベースはいつ頃から存在しているのだろうか？

![](https://thedataquarry.com/posts/vector-db-1/vector-db-timeline.png)

Vespaは、当時主流だったBM25キーワードベースの検索アルゴリズムに加えて、ベクトル類似性検索を取り入れた最初期のベンダーの1つである（面白い事実：Vespaの[GitHubリポジトリ](https://github.com/vespa-engine/vespa)には現在約75000のコミットがある🤯）。2018年末にWeaviateがオープンソースのベクトル検索特化型データベースを開始し、2019年にはMilvus（こちらもオープンソース）のリリースとともに、この分野での競争が激化した。なお、年表にも記載されているZillizは、Milvusの（商用の）親会社であり、Milvusを基盤として構築されたフルマネージドのたクラウドソリューションを提供するため、分けずに掲載した。2021年には、Vald、Qdrant、Pineconeという3つの新しいベンダーが参入した。Elasticsearch、Redis、PostgreSQLのような既存ベンダーはこの時点まで目立っておらず、意外なことにベクトル検索を提供し始めたのは2022年以降のことである。

### ソースコードの入手

ここに挙げた選択肢の中で、完全にクローズドソースなのはただ1つ：Pineconeだけである。Zillizもクローズドソースのフルマネージド商用ソリューションだが、Milvusをベースにすべて構築されており、Milvusの親会社とみなすことができる。他の選択肢はすべて、少なくともコードベースに関してはソースが入手可能であり、特定のライセンスによってコードの許容範囲とデプロイ方法が決定される。

![](https://thedataquarry.com/posts/vector-db-1/vector-db-source-available.png)

> 💡**ヒント**
>開発者として、各データベースのオープンソースGitHubレポジトリにあるIssues、PR、Releasesを追跡することで、そのロードマップで何が優先され、何が対処されているのかについてをかなり理解することができる。GitHub Stars⭐️ は、そのプロジェクトに対するコミュニティの関心を示す良い指標だが、個人的な経験に勝るものはなく、私はできるだけ多くのオープンソースデータベースを試してみるようにしている。

### ホスティング方法

データベースベンダーが提供する典型的なホスティング方式には、セルフホスト（オンプレミス）とマネージド／クラウドネイティブがあり、いずれもクライアント-サーバーアーキテクチャを踏襲している。3つ目の、より最近の選択肢は、*組み込み*モードで、データベース自体がサーバーレス方式でアプリケーションコードと緊密に結合している。現在、組み込みデータベースとして利用できるのはChromaとLanceDBのみである。

![](https://thedataquarry.com/posts/vector-db-1/vector-db-hosting.png)

図中の丸で囲んだ部分は特に興味深く感じる。

- Chromaは、組み込みモード（デフォルト）、クライアント-サーバーアーキテクチャに従ったマネージドのクラウドネイティブサーバー[^4]、クラウド上でサーバーレス・コンピューティングを可能にするクラウド型分散システムを備えた、オールインワンのサービスを構築しようとしている。
- 執筆時点で最も新しいベクトルデータベースであるLanceDBは、AI向けの組み込み型マルチモーダルデータベースを、分散型サーバーレス・コンピューティング環境と共にフルマネージドクラウドとして提供[^5]するという野心的な目標を掲げている。このデータベースは、ベクトル操作とMLのために高速で効率的なルックアップを実行するモダンなカラム型フォーマットである[Lanceデータフォーマット](https://lancedb.github.io/lance/)をベースとして使用している。

### インデックス作成方法

ほとんどのベンダーは、キーワード検索とベクトル検索を様々な方法で組み合わせるハイブリッドなベクトル検索手法を使用している。しかし、各データベースが使用しているる基本的なベクトルインデックスは、かなり大きく異なる場合がある。

![](https://thedataquarry.com/posts/vector-db-1/vector-db-indexes.png)

大半のデータベースベンダーは、HNSW（Hierarchical Navigable Small-World graphs）のカスタム実装を採用している。ベクトルインデックスをディスクに永続化することは、メモリよりも大きなデータセットを扱うために、急速に重要な目的になりつつある。2019年のNeurIPSのDiskANN[^6]に関する論文では、数十億のデータポイントを検索するための最も高速なオンディスクインデックスアルゴリズムである可能性が示された。Milvus、Weaviate、LanceDBのようなデータベースは、すでにDiskANNの独自バージョンを実装しており（あるいは積極的に実装中）、この分野では注目すべきものである。

## 主なポイント

このセクションでは、各データベースの主なポイント及び長所と短所をリストアップする。これらの中には、様々な手段（ブログ、ポッドキャスト、研究論文、ユーザー／共同研究者との会話、そして私自身のコード）を通じて得た情報に基づいた私の考えが含まれる。

### Pinecone

- **長所**<br>非常に簡単にセットアップして実行できる（ホスティングの負担がなく、完全にクラウドネイティブ）。ユーザーがベクトル化やベクトルインデックスについて何も知らなくても問題ない。彼らのドキュメント（これもかなり良い）通り、ただ**動く**だけである。
- **短所**<br>完全にプロプライエタリであり、GitHubで進捗を追うことができないため、内部で何が行われているのか、何がロードマップにあるのか、を知ることは不可能。また、[一部のユーザの経験](https://twitter.com/garywupx/status/1631238355634495488)では、から、完全に外部のサードパーティホスティングサービスに依存する危険性と、データベースのセットアップや運用に関して開発者の立場からの完全なコントロールの欠如が浮き彫りになっている。フルマネージドかつクローズドソースのソリューションに依存することのコストへの影響は、オープンソースでセルフホスティングの代替手段の数が非常に多いことを考慮すると、大きなものになる可能性がある。
- **私の見解**<br>ベクトルデータベースが注目されていなかった2020-21年、Pineconeは他のベンダーよりも進んでおり、他のベンダーが提供していない便利な機能を開発者に提供していた。2023年になって、正直言って、Pineconeが現在提供しているもので他のベンダーが提供していないものはほとんどなく、他のほとんどのベンダーは、少なくともセルフホスティング、マネージド、そして組み込みモードを提供しており、彼らのアルゴリズムや基礎技術のソースコードがエンドユーザーに透明性があることは言うまでもない。
- **公式ページ**<br>[pinecone.io](https://www.pinecone.io/)

### Weaviate

- **長所**<br>素晴らしい[ドキュメント](https://weaviate.io/developers/weaviate)（技術的な詳細や進行中の実験を含め、最も優れたものの1つ）。Weaviateは可能な限り最高の開発者体験を構築することに真剣に取り組んでいるようで、Dockerを使って非常に簡単にセットアップして実行できる。クエリに関しては、キーワード検索とベクトル検索の両方の機能を提供しながら、ミリ秒未満の高速な検索結果を生成する。
- **短所**<br>WeaviateはGolangで構築されているため、スケーラビリティはKubernetesを通じて達成される。このアプローチは（Milvusと同様に）データが非常に大きくなると、かなりのインフラリソースを必要とすることで知られている。Weaviateのフルマネージドサービスの長期的なコストへの影響は不明であり、QdrantやLanceDBのような他のRustベースの代替との性能を比較することは理にかなっているかもしれない（ただし、どのアプローチが最もコスト効果の高い方法でより良くスケールするかは時間が示すことになるだろう）。
- **私の見解**<br>Weaviateには素晴らしいユーザーコミュニティがあり、開発チームは非常に高いスケーラビリティ（数千億ベクトル）を積極的にアピールしている。ベクトル検索を目的とする大量のデータを持つ大企業がターゲット市場のようだ。ベクトル検索に加えてキーワード検索、そして一般的に強力なハイブリッド検索も提供することで、Elasticsearchのようなドキュメントデータベースと直接的に競合する様々なユースケースに対応できる。Weaviateが積極的に取り組んでいるもう1つの興味深い分野は、ベクトルデータベースを介した[データサイエンスと機械学習](https://www.youtube.com/watch?v=IRWHa57T-zk)であり、これにより従来の伝統的な検索得アプリケーションの枠を超えている。
- **公式ページ**<br>[weaviate.io](https://weaviate.io/)

### Qdrant

- **長所**<br>Weaviateよりも新しいが、Qdrantもドキュメントが素晴らしく、これにより開発者はDocker経由で簡単に立ち上げることができる。Qdrantは完全にRustで構築されており、現在のバックエンド開発者にとって最も人気のある言語であるRust、Python、GoのクライアントからAPIを利用できる。ベースとなるRustのパワーにより、リソースの使用率はGoで作られた代替よりも（少なくとも私の経験では）低いようだ。スケーラビリティは現在、パーティショニングとRaft合意プロトコルによって達成されており、これはデータベースの領域での標準的な実践である。
- **短所**<br>Qdrantは競合他社よりも比較的新しいツールであるため、WeaviateやMilvusのような競合に対してクエリのユーザーインターフェースのような部分で遅れをとっていたが、新しいリリースごとに[そのギャップは急速に縮小](https://qdrant.tech/articles/qdrant-1.3.x/#qdrant-web-user-interface)している。
- **私の見解**<br>Qdrantは、インフラコストを最小限に抑え、モダンなプログラミング言語であるRustのパワーを活用したい多くの企業にとって、最初の選択肢としてのベクトル検索バックエンドとしての地位を確立する可能性があると思う。本稿執筆時点においては、ハイブリッド検索はまだ利用できないが、ロードマップによれば積極的に取り組んでいる。また、Qdrantはインメモリとオンディスクの両方で、HNSWの実装をどのように最適化しているかについて、継続的に最新情報を公開しており、長期的な検索精度とスケーラビリティの目標に大いに役立つだろう。[GitHub Starsのヒストリー](https://star-history.com/#weaviate/weaviate&qdrant/qdrant&Date)を見る限り、Qdrantのユーザーコミュニティが急速に成長（興味深いことにWeaviateよりも速い）しているのは明らかである。おそらく世界中がRust🦀に熱狂しているのだろうか？いずれにせよ、Qdrantの上でビルドするのは"とても"楽しい、少なくとも私の視点では。
- **公式ページ**<br>[qdrant.tech](https://qdrant.tech/)

### Milvus/Zilliz

- **長所**<br>ベクトルDBのエコシステムに長く存在し、多くのアルゴリズムを持つ非常に成熟したデータベース。ベクトルインデックスの[オプションが豊富](https://milvus.io/docs/index.md)で、高いスケーラビリティを目指してGoで書かれている。2023年現在、最も効率的なオンディスクのベクトルインデックスと言われる[DiskANNの実装](https://milvus.io/blog/2021-09-24-diskann.md)を提供している唯一の主要ベンダーである。
- **短所**<br>私の目にはから見ると、Milvusはスケーラビリティの問題に対して「できることは何でもやる」というソリューションのように思える。プロキシ、ロードバランサー、メッセージブローカー、Kafka、Kubernetesを組み合わせることで高度なスケーラビリティを実現しているが、その結果、システム全体が非常に複雑になり、リソースを多く消費しがちになっている。クライアントAPI（Pythonなど）についても、WeaviateやQdrantのような開発者体験をより重視する新しいデータベースに比べると、読みにくく直感的ではない。
- **私の見解**<br>Milvusがストリーミングデータのベクトルインデックス化において大規模なスケーラビリティを念頭に構築されたことは明らかである。多くの場合、データのサイズがあまり大きくないような場合、Milvusは過剰に思える。大規模だがより静的で更新頻度が少ない場合には、QdrantやWeaviateのような選択肢の方が、より安価かつより迅速に本番稼動させることができるかもしれない。
- **公式ページ*<br>[milvus.io](https://milvus.io/)、[zilliz.com](https://zilliz.com/)

### Chroma

- **長所**<br>便利なPython/JavaScriptインターフェースを提供しており、開発者はベクトルストアを迅速に立ち上げることができる。データベースとアプリケーションのレイヤーが密接に統合された組み込みモードをデフォルトで提供した組み込みモードを市場で初めて提供した。これにより開発者は、迅速なビルド、プロトタイプの作成、そしてプロジェクトを世界に公開することができる。
- **短所**<br>他の特化ベンダーとは異なり、Chromaは主に既存のOLAPデータベース（Clickhouse）と既存のオープンソースベクトル検索実装（[hnswlib](https://github.com/nmslib/hnswlib)）をまとめたPython/TypeScriptのラッパーである。現時点（2023年6月時点）では、独自のストレージレイヤーは実装していない。
- **私の見解**<br>ベクトルDB市場は急速に進化しており、Chromaは「待って観察する」という哲学を採用しているように見える[^7]。そして、サーバーレス/組み込み、セルフホスティング（クライアント-サーバー）、クラウドネイティブな分散SaaSソリューションなど、複数のホスティングオプションを提供することを目指している数少ないベンダーの1つであり、組み込みモードとクライアント-サーバーモードの両方を提供する可能性がある。ロードマップ[^4]によれば、Chromaのサーバー実装は現在進行中である。Chromaが取り入れているもう1つの興味深い革新分野は、「クエリの関連性」を定量化する手段であり、これは返された結果が入力ユーザークエリにどれだけ近いかを示すものである。埋め込み空間の可視化もロードマップに記載されており、検索以外の多くのアプリケーションでデータベースが使用される可能性がある革新分野である。しかし、長期的な視点から見ると、ベクトル検索の領域で組み込み型データベースアーキテクチャが収益化に成功した例はまだ見られないため、その進化（後述のLanceDBと並んで）は注目すべきである！
- **公式ページ**<br>[trychroma.com](https://www.trychroma.com/)

### LanceDB

- **長所**<br>マルチモーダルデータ（画像、オーディオ、テキスト）に対してネイティブに分散インデックス作成と検索を行うように設計されており、MLのための新しく革新的なカラム型データフォーマットである[Lanceデータフォーマット](https://lancedb.github.io/lance/)を基盤としている。Chromaと同様に、LanceDBは組み込み型のサーバーレスアーキテクチャを採用しており、Rustで一から構築されているため、Qdrantと並んで、Rust🦀の速度🔥、メモリ安全性、比較的低いリソース利用を活用する数少ない主要ベクトルデータベースベンダーである。
- **短所**<br>LanceDBは非常に新しいデータベースなので、多くの機能が活発に開発中であり、小規模なエンジニアリングチームのため、今後1年程度での機能の優先順位付けが課題となるだろう。
- **私の見解**<br>世の中にあるすべてのベクトルデータベースの中で、LanceDBは他と最も差別化されていると思う。これは主に、データストレージレイヤー（parquetよりも高速なカラム型フォーマットであるLanceを使用して、非常に効率的なルックアップのために設計されている）とインフラストラクチャレイヤー（サーバーレスアーキテクチャを採用）が革新的なためである。その結果、開発者は、データレイクに直接接続されたセマンティック検索アプリケーションを分散方式で、より簡単に、より自由度高く構築できる。
- **公式ページ**[lancedb.com](https://lancedb.com/)

### Vespa

- **長所**<br>実績のあるキーワード検索とHNSW上のカスタムベクトル検索を組み合わせて、最も「エンタープライズ対応」なハイブリッド検索機能を提供。Weaviateのような他のベンダーもキーワード検索とベクトル検索を提供しているが、Vespaはこのサービスをいち早く市場に投入したことで、十分な時間をかけて、高速で正確、そしてスケーラブルに進化させてきた。
- **短所**<br>GoやRustのようなパフォーマンス志向の言語で書かれたよりモダンな代替に比べると、開発者体験はスムーズとは言い難い。なぜならばアプリケーション層がJavaで書かれているためである。また、最近まで、DockerやKubernetesのようなツールを利用して、開発環境を簡単にセットアップしたり撤去したりすることができなかった。
- **私の見解**<br>Vespaは非常に優れたサービスを提供しているが、アプリケーションのほとんどはJavaで構築されており、バックエンドとインデックスレイヤーはC++で構築されている。そのため、長期的なメンテナンスが難しく、他のデータベースよりも開発者に優しくない傾向がある。最近の新しいデータベースのほとんどは、GoやRustといった単一の言語で書かれており、WeaviateやQdrant、LanceDBといったデータベースでは、アルゴリズムやアーキテクチャの革新がより速いペースで進んでいるように見える。
- **公式ページ**<br>[vespa.ai](https://vespa.ai/)

### Vald

- **長所**<br>高度に分散されたアーキテクチャによってマルチモーダルなデータストレージを扱うように設計されており、インデックスのバックアップのような便利な機能も備えている。非常に高速なANN検索アルゴリズムNGT(Neighborhood Graph & Tree)を使用しており、高度に分散されたベクトルインデックスと組み合わせて使用した場合、ANNアルゴリズムとしては最速の部類に入る。
- **短所**<br>他のベンダーに比べ、全体的な認知度と利用度がかなり低いようで、またドキュメントにもどのようなベクトルインデックスが使われているのかが明確に書かれていない（「分散インデックス」はかなり曖昧な表現）。また、Yahoo! Japanという1企業が完全に出資しているようで、他の主要ユーザーに関する情報はほとんどない。
- **私の見解**<br>Valdは他のベンダーよりもずっとニッチな存在で、主にYahoo! Japanの検索要件に対応しているように思える。少なくともGitHub Starsを基準に考えれば、全体的にかなり小さなユーザーコミュニティに思える。その理由のひとつは、Valdが日本に拠点を置いていること、そしてEUやベイエリアにある他のベンダーのように大々的にマーケティングされていないことだろう。
- **公式ページ**<br>[vald.vdaas.org](https://vald.vdaas.org/)

### Elasticsearch、Redis、pgvector

- 長所：すでにElasticsearch、Redis、PostgreSQLのような既存のデータストアを使用している場合、新しい技術に頼ることなく、ベクトルインデックス作成と検索を利用できる。
- 短所：既存のデータベースは汎用的に設計されているため、データを最も最適な方法で保存したりインデックスを作成したりするわけではなく、その結果、百万単位以上のベクトル検索を含むデータではパフォーマンスが低下する。Redis VSS（Vector Search Store）は、純粋にインメモリで動作するため高速だが、データがメモリより大きくなった瞬間、代替ソリューションを考える必要が出てくる。
- **私の見解**セマンティック検索を必要とする分野では、ベクトル専用に特化したデータベースが徐々に既存のデータベースを凌駕していくだろうと思う。HNSWやANNアルゴリズムのようなインデクシング手法は文献で十分に文書化されており、ほとんどのデータベースベンダーは独自の実装を展開することができるが、特化型ベクトルデータベースは目の前のタスクに最適化されているという利点があり（GoやRustのようなモダンなプログラミング言語で書かれていることは言うまでもない）、スケーラビリティとパフォーマンスの理由から、長期的にはこの分野で勝利する可能性が高い。
- **公式ページ**<br>
  - Elasticsearch: [elastic.co](https://www.elastic.co/what-is/vector-search)
  - Redisベクトル検索ストア: [Redis Labs](https://redis.com/solutions/use-cases/vector-database/)
  - PostgreSQL *pgvector*拡張: [supabase.com](https://supabase.com/docs/guides/database/extensions/pgvector)

## 結論: 兆スケールの問題

VCのエコシステムは言うに及ばず、ある特定の種類のデータベースがこれほど世間の注目を集めるとは歴史上誰も想像できなかったことだろう。ベクトルデータベースのベンダー（Milvus[^8]やWeaviate[^9]など）が解決しようとしている重要なユースケースの1つは、1兆スケールのベクトル検索を可能な限り低レイテンシーで実現する方法である。これは非常に困難な課題であり、最近ではストリームやバッチ処理で大量のデータが送られてくるため、ストレージとクエリのパフォーマンスに最適化されたベクトル特化型データベースが、近い将来この壁を破る可能性が高いと言える。

データベースの世界で歴史的に最も成功したビジネスモデルは、最初にコードをオープンソース化し（それにより情熱的なコミュニティがテクノロジーを中心に構築される）、その後マネージドサービスやクラウドサービスを通じてツールを商品化するという、試行錯誤を重ねたアプローチであったという見解でこの記事を終えることとする。組み込み型データベースはこの分野では比較的新しく、どのようにして製品を収益化し、長期的な収益を生み出すことができるかについてはまだ未知数である。そのため、完全なクローズドソースの場合は大きな市場シェアを獲得するのは難しいと考えられる。長期的には、開発者のエコシステムと開発者の経験を重視するデータベースが成功する可能性が高く、そのツールを信じる活発なオープンソースコミュニティを構築することは想像以上に重要なことになると私は直感している！

この要約が役に立つことを願っている！次回は、ベクトルデータベースにおける基本的な検索とインデックス作成のアルゴリズムを要約し、技術的な詳細についてもう少し掘り下げてみたいと思う。🤓 🚀

**このシリーズの他の記事**

- Vector databases (Part 2): Understanding their internals [[元記事]](https://thedataquarry.com/posts/vector-db-2/) [[日本語訳]](https://zenn.dev/kun432/articles/20230921-vector-databases-jp-part-2)
- Vector databases (Part 3): Not all indexes are created equal [[元記事]](https://thedataquarry.com/posts/vector-db-3/) [[日本語訳]](https://zenn.dev/kun432/articles/20230923-vector-databases-jp-part-3)
- Vector databases (Part 4): Analyzing the trade-offs [[元記事]](https://thedataquarry.com/posts/vector-db-4/) [[日本語訳]](https://zenn.dev/kun432/articles/20230928-vector-databases-jp-part-4)

## 更新

日々コミュニティから新しいことを学び続けているので、更新があればこのセクションに記録していく😄。

- 2023-07-01: Vespaのプログラミング言語と位置情報を修正。
- 2023-07-02: [^4]と[^10]に基づきChromaに関する情報を追加。

-----

[^1]: 資金調達額のソース: [objectbox.io](https://objectbox.io/vector-database/)
[^2]: "Rewriting a high performance vector database in Rust", [Pinecone blog](https://www.pinecone.io/learn/rust-rewrite/)
[^3]: "Rewriting Lance in Rust", [LanceDB blog](https://blog.lancedb.com/please-pardon-our-appearance-during-renovations-da8c8f49b383)
[^4]: Chromaの [ロードマップ](https://docs.trychroma.com/roadmap)
[^5]: LanceDB Cloud (サーバレス), "[coming soon](https://lancedb.com/)"
[^6]: "DiskANN: Fast Accurate Billion-point Nearest Neighbor Search on a Single Node", [NeurIPS 2019](https://suhasjs.github.io/files/diskann_neurips19.pdf)
[^7]: "Chroma: Open source embedding database", [Software Engineering Daily Podcast](https://softwareengineeringdaily.com/2023/04/20/open-source-embedding-database/)
[^8]: Milvus Was Built for Massive-Scale (Think Trillion) Vector Similarity Search, [Milvus blog](https://milvus.io/blog/Milvus-Was-Built-for-Massive-Scale-Think-Trillion-Vector-Similarity-Search.md)
[^9]: "ANN algorithms: Vamana vs. HNSW", [Weaviate blog](https://weaviate.io/blog/ann-algorithms-vamana-vs-hnsw#large-scale)
[^10]: "Fireside Chat with Jeff Huber, co-founder of Chroma", [Data Driven NYC](https://www.youtube.com/watch?v=NI0faK90TP4)

