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

command, query modelのDBは、同じDB, schemaでいいよう、tableは別にするか？

clientからcommandを叩く、command内には実施させたい振る舞いの名前と引数がある。それが実施される。

発行されたcommandはcommnadhandlerが受け取るらしい。

clientからcommandを叩くという動作は、application serviceのmethod実行となる場合もあるし、
複数のcommnad handlerを一つのapplication serviceとして扱うのを分類方式というよう。
単一methodをもつ単一classという方式もあり専門方式というよう。

sampleのsourceは`tenant, backlogitem, sprint`を1 command method内で集約してるっぽかった、内訳は
`sprint`に`tenant, backlogitem`をsetして、`backlogitem`のcommitTo metodを読んでいる。
また、commitToは、logicのあと、最後にDomainEventPublisherよりpublishを実行



#### Questions

command, query modelって何単位？object ?table ? 

例えば、更新requestが発行されて, 同期で更新して更新結果をviewに表示させたい時、1 transaction内での処理はaction modelを起動して同期で結果を待ち、query modelを access

