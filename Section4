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

rule

statusを持つinstanceのstatusを変えるときはcommandとなり、void型。queryはその逆で変更してはいけない。

command model とは,,,? class

BackLogItem classが、commitTo(springId)の中で、
DomainEventPublisherを生成し、publishしていた。
例ではcommitToは、`@Transactional`を持つcommitBacklogToSprint内で使われている。
