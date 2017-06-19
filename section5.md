# Section 5 Entity

## 概要

- tableに合わせたobjectを作ると大量のgetter, setterに悩まされる。

- CRUDとも違う。単純なapplicationであれば、CRUDでDBとのやりとりを
直接行えば良い。Rails, GRailsが向いている。

- ただし、巨大なData tableや大量のfieldが招く複雑性に対処したいのなら、DDDを採用することを検討したほうがいい。

- Entityは、life cycleの連続性と同一性をもつ。また、同一性から識別も可能である。

### 一意の識別性について

entityの設計はまずはここかららしい。

方法
 - 一般的に1意と認められているIDを使う。納税者ID, my number
   - 変更しにくい
 - アルゴリズムから生成 (UUIDとか)
   - 可読性が悪い
 - DBに任せる。 (id)
   - ただし、一意性をDB accessして確認する場合、performanceに影響あり、application側のrepositoryにcacheさせとくような工夫がいる。
   ←どっちみちこれはいると思う。
 - システム間で取り決めたIDを使う。
 
乱数とかはbyte[]にしてDBに入れるとmemoryの節約に良さそう。

注意として、DB側の関心ごとをApplicationに持ち込まないようにする。
DBのIDを混同させないほうがいい。DBの採番を使ってもいいが、逆にDBの採番をapplication側で担うのはDB依存になる可能性があるため、よくないと言っているように見える。

識別子はobjectであったほうがいいそう。stringはよくない。
 
NoSQLのRiakやMongoDBは、識別子を作ってくれるそう。（DBに属すると思うのだが、objectで保存するからそこは違う扱いなのかな）
 
実装としては、`ProductRepository`が、

```
public productId nextIdentity
```

というmethodを持っており、採番させてた。

unique serializeは、async処理、CQRS, event sourcingが絡むと難しくなりそう、その際は、Domain Eventを参照させるなど工夫がいる。

### タイミング

いつ採番するか? object生成の時 or DB格納時

→　前者が言いそう(fig.5-4)、respositoryから見れば、識別子のないentityがrepositoryに含まれていることになるのだから。
repository無いの操作が新しいobjectがたくさんあるときに、ごちゃごちゃになるらしい。

対策は、識別子の早期生成か、equals methodを使って、識別子の無いobjectを識別させるようにするからしい。

### Entity探し

SaaSOvation
 - User
   - `tenant`が関連づけられている
   - system userは常に`認証済み`
   - 名前(`name`)、連絡先(`address`)をもつ
   - 自分か、`manager`が個人情報のedit可能
   - `password`のeditは可能, `password`の暗号化はdomain serviceで実装する。

↑これでは曖昧、実装方法は複数ある。

`Tenant`はUserのaggregationでは無い。答えはどちらもaggregation.
`Tenant`はuserを収集したりしないのでaggregationでは無いらしい。

`entity`を見つける→一意性をどうやって持たせるか検討する。
```
class Tenant extends Entity{
  private final String tenant_id;
  private final String name;
  
  Tenant(String name){
    if (name.isNull){
      throws new IllgalStateException();
    }
    this.tenant_id = UniqueProductId;
  }
  
  public boolean isActive(){return true}
  public void active(){}
  public void deactive(){}
  public void registerUser(){}
  
  private String UniqueProductId(){
    //...
    return productId;
  }
}
```

TenantとUserの振る舞いを考えた過程で新たにdomainを抽出

```
- AuthentificationService
- Invitation (ただしどんなinvitationかや管理方法が不明のため保留)
- Person (E) ※Userの責務軽減のため、切り出した
  - name (VO)
  - ContactInformation (VO)
- User
  - changePassword
  - changeUsername
  - changeContactInformation
```

sourceでは、UserはPersonを継承するのではなく、fieldに定義している。

### Role and Responsibility

roleとは、認証、アクセスコンテキストにおいてはセキュリティの関心ごと全般。

一般的にinterfaceがclassのroleを定義する。

```
- interface
User
Person
HumanUser
Principal
```

Object統合失調症
→委託先のclassがobject識別子を知らずに自分が誰かわからない状態のこと

interface classの名前の先頭に`I`を文章っぽくなる。

Interfaceを実装するか、methodを持つか
例えば`changePassword`はIF ? `public method?`

```
IaddUserToTenant
```
みたいな、この発想は`...able`に似ているが、具体的で読みやすいと思う。冗長性が薄くなるのと其の分`impliment`が増えるのはきになる。


```
IaddUserToTenant
```
みたいな、この発想は`...able`に似ているが、具体的で読みやすいと思う。冗長性が薄くなるのと其の分`impliment`が増えるのはきになる。

### contextの発見

- サポートコンテキスト
- 請求コンテキスト
- 顧客関係管理コンテキスト

### Validatiion

Entityは`isvalid`的なのを持っておらず、`setter`でvalidation checkしていた。またdomainはmethod毎に
argumentsをvalidation checkしている。

基本的にmethod内の実装は

1. argumentsのvalidation check
2. 処理, 何してるかパッとわかる表現と量
3. DomainEnvetPublisherで引数に関連objectをせsetしてpublishする。

また、AssettionConcernというclassを`Entity class`は継承しており、method名の通りにvalidationしたあと、throwでExceptionを投げるために使われていた。

UserのConstructorはprotectedになったその代わりに`Tenant`が`factory`となり、`registerUser`でUserを生成してた。

#### 自己カプセル化

component designは事前条件、事後条件、不変条件を指定する。

<strong>fieldが、OVにするか否かは判断どうするか？</strong>



#### その他

sourceより気になったこと
- PersonもUser fieldを持っているし、UserもPerson fieldを持ってる、ただしPersonはUserをconstractorに入れてない。
