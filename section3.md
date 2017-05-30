# Section 3

#### context間 Hub gateway pattern
 - partnership : 
 - shared kernel :一緒のmemory領域を使う
 - Upstream downstream : 処理の流れ、
 - mud dumpling :antiだけど一緒にしちゃう,
 - Open Host service : REST
 - Published Language :Model Convertion. messageの中身の共通認識
 - AntiCurruption Layer :明確な方法はない。考え方

 なぜ、project ovatio (core domain)nは、ACLでくくられてる？coreだから？
 
#### ACL between idOvation and collabOvation
 - MemberService has TeamMember, Production object
 - maintainMembers() : IF
 - SynchonizeMembers() : IdOvation
 - updateMembers()
  
#### ポイント
 
1. 例えばユーザー認証していないと使えない機能は、明示的に使えないとした方が良いので、使えない状態をenum等で表現するのが良いとあった。

2. projectOvationからcollabOvationへのaccessには、CollaborationAdapterを挟んでいる.

#### 何か分からない

1. RPC?
2. Translator patternが初耳→httpから受け取ったものをobject変換してっぽい (toMember)
