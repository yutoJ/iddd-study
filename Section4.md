# Section 4

#### Architecture
 - Hexagonal architecture (Port & Adapter)
   - applicationと外部を関係を明確にし、hubをadapterという表現で使っている. adapterはMQ, http, REST, SOAP, Messaging等々、java classとは限らない. 6角形の各変がadapterの種類になるそう。use caseから作図するっぽい(RDRAかも)。
 - CQRS
 - Pipe & Filter (Distributed computing)

keyword
 - aggregation
 - domain event (登録するらしい)
 - domain event publisher ()
 - DomainRegistry
 - service factory
 
#### New context
 - TrackOvation

#### check

1. testはportに依存しないでもつくれるはず (coverageも満たせる？)
2. RESTful httpは、resourceを1つのURIに載せている。

#### other
 - DIP: 依存関係逆転の法則 
   - respositoryはdomainのinterfaceになる、さらにinfra層のdomainにaccessもする。よって逆転する場合も有るってことか？

 - ProductServiceからprpduct instanceを作ってるsourceがあった、serviceはfactoryでもある？ただし、domain modelを直接公開するというのは違う。IFは例えば抽象的に

JAX-RSでは、resourceがcontrollerっぽい役割もしてる？
resourceは網羅的に色々入ってるか？

```
@Path(tenants/{tenantId}/products)
```

```
Product product = ProductService.product(tenantId, productId)
```

#### example

```
/user:/task:/

でgetTaskでtask objectを変更するAPIが有る際、実はよくない。
task objectの変更が難しくなる。
```

#### CQRS

```
requestが来て、dataを返却するまでに多くのinstanceやDBにaccessしたりして、必要なdataを全て取得するのが難しい場合が有る。
集約型になっておりmethod上ではsimpleなinterfaceであっても、実装してみるとperfomanceに悩むことがある。
```

 - 対応策1: clientに複数回accessしてもらう
 - 対応策2: responseがすぐに作れるようにあらかじめresponseを作っておく.


CQRSは、、、？

 - Command: actionを実行する
 - Query: 呼び出し元にdataを戻す

を分離させる

例: 質問をされた時に、質問の答えを変えない。

#### rule

statusを持つinstanceのstatusを変えるときはcommandとなり、void型。queryはその逆で変更してはいけない。

command model とは,,,? class

BackLogItem classが、commitTo(springId)の中で、
DomainEventPublisher (Observer Patternで実装)を生成し、publishしていた。このPublisherはquery modelの更新を保証する。よってsubscriberが必要

→この当たりはlanmda architectureのbatch viewとreal time viewにちょっと近いかもしれない。
law dataをどんどん積み上ガっている状態から、batchが定期的にquery用にdataを処理し、queryは処理後のdataへaccessしてclientへ返す。

この辺りの同期、非同期はSLAによる。また、Event Sourcingを
考えるのも良い。


例ではcommitToは、`@Transactional`を持つcommitBacklogToSprint内で使われている。

command, query modelのDBは、同じDB, schemaでいいよう.

#### pipe and filter

pipe: パイプラインの処理単位
filter: 入力データの取得, 入力データの変更・洗練・変換, データの出力

→これって用はbatch? っぽいが、並列処理させることが主目的？

#### Event sourcing

updated_atを持っていたとしてもdataがどう変化したかは記録できない CMSのようなversion管理の概念を取り入れると追跡できる。

アーキテクチャーは更新eventを記録し、実際に参照するときは、そのタイミングでさまったのを返す。ただし、ずっと過去のeventのものをさまるのは時間がかかるので、ある程度snapshotをもうけながらさまる必要がある。

とはいえ、上記の実装はCQRSの方式を活用する。

##### 業務的なmerit
 - bugや障害があっても、その部分を修正して再度
集約を作ればいいという発想ができるようになる。
※ここは、結構運用や実装を練らないと難しそう
 - 追跡ができるので、分析や傾向が掴みやすく、BI的にもいいそう。

#### Questions

command, query modelって何単位？object ?table ? 

例えば、更新requestが発行されて, 同期で更新して更新結果をviewに表示させたい時、1 transaction内での処理はaction modelを起動して同期で結果を待ち、query modelにて最新情報を更新してviewに返すような動き？

tableを分けるとquery側への更新はだれがどのタイミングで行うか？全部CQRSにした方がいいというのが前提？
