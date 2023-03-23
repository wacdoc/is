# Búðu til þinn eigin SMTP póstsendingarþjón

## formála

SMTP getur beint keypt þjónustu frá skýjaframleiðendum, svo sem:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali ský tölvupóstskeyti](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Þú getur líka smíðað þinn eigin póstþjón - ótakmarkað sending, lágur heildarkostnaður.

Hér að neðan sýnum við skref fyrir skref hvernig á að byggja upp okkar eigin póstþjón.

## Val á netþjónum

SMTP-þjónninn sem hýsir sjálfan krefst opinberrar IP-tölu með gáttum 25, 456 og 587 opnar.

Algengt opinber ský hafa sjálfgefið lokað á þessar höfn og það gæti verið hægt að opna þær með því að gefa út verkbeiðni, en það er mjög erfitt þegar allt kemur til alls.

Ég mæli með að kaupa af gestgjafa sem hefur þessar gáttir opnar og styður uppsetningu öfugs léns.

Hér mæli ég með [Contabo](https://contabo.com) .

Contabo er hýsingaraðili með aðsetur í München, Þýskalandi, stofnað árið 2003 með mjög samkeppnishæf verð.

Ef þú velur evru sem gjaldmiðil fyrir kaup verður verðið ódýrara (þjónn með 8GB minni og 4 örgjörva kostar um 529 júan á ári og upphafsuppsetningargjaldið er ókeypis í eitt ár).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

Þegar pantað er skaltu athuga `prefer AMD` , og þjónninn með AMD CPU mun hafa betri afköst.

Hér á eftir mun ég taka Contabo's VPS sem dæmi til að sýna hvernig á að byggja upp þinn eigin póstþjón.

## Ubuntu kerfisstillingar

Stýrikerfið hér er Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

Ef þjónninn á ssh sýnir `Welcome to TinyCore 13!` (eins og sýnt er á myndinni hér að neðan), þýðir það að kerfið hefur ekki verið sett upp ennþá. Vinsamlegast aftengdu ssh og bíddu í nokkrar mínútur til að skrá þig inn aftur.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

Þegar `Welcome to Ubuntu 22.04.1 LTS` birtist er frumstillingunni lokið og þú getur haldið áfram með eftirfarandi skrefum.

### [Valfrjálst] Frumstilla þróunarumhverfið

Þetta skref er valfrjálst.

Til hægðarauka setti ég uppsetningu og kerfisstillingar á Ubuntu hugbúnaði í [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Keyrðu eftirfarandi skipun til að setja upp með einum smelli.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Kínverskir notendur, vinsamlegast notaðu eftirfarandi skipun í staðinn og tungumálið, tímabeltið osfrv. verður sjálfkrafa stillt.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo gerir IPV6 kleift

Virkjaðu IPV6 þannig að SMTP geti líka sent tölvupóst með IPV6 vistföngum.

breyta `/etc/sysctl.conf`

Breyttu eða bættu við eftirfarandi línum

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Fylgdu eftir með [tengiliðakennslunni: Bætir IPv6 tengingu við netþjóninn þinn](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Breyttu `/etc/netplan/01-netcfg.yaml` , bættu við nokkrum línum eins og sýnt er á myndinni hér að neðan (Contabo VPS sjálfgefna stillingarskrá hefur nú þegar þessar línur, taktu bara af þeim).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Síðan `netplan apply` til að láta breytta stillingu taka gildi.

Eftir að uppsetningin hefur tekist geturðu notað `curl 6.ipw.cn` til að skoða ipv6 vistfang ytra netsins þíns.

## Klóna stillingargeymsluna ops

```
git clone https://github.com/wactax/ops.soft.git
```

## Búðu til ókeypis SSL vottorð fyrir lénið þitt

Til að senda póst þarf SSL vottorð fyrir dulkóðun og undirritun.

Við notum [acme.sh](https://github.com/acmesh-official/acme.sh) til að búa til vottorð.

acme.sh er opinn uppspretta sjálfvirkt undirritunarverkfæri fyrir vottorð,

Sláðu inn stillingarvöruhús ops.soft, keyrðu `./ssl.sh` og `conf` mappa verður búin til í **efri möppunni** .

Finndu DNS veituna þína frá [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , breyttu `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Keyrðu síðan `./ssl.sh 123.com` til að búa til `123.com` og `*.123.com` vottorð fyrir lénið þitt.

Fyrsta keyrslan mun sjálfkrafa setja upp [acme.sh](https://github.com/acmesh-official/acme.sh) og bæta við áætluðu verkefni fyrir sjálfvirka endurnýjun. Þú getur séð `crontab -l` , það er svona lína sem hér segir.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

Slóðin fyrir myndaða vottorðið er eitthvað eins og `/mnt/www/.acme.sh/123.com_ecc。`

Endurnýjun skírteina mun kalla `conf/reload/123.com.sh` skriftu, breyttu þessu skriftu, þú getur bætt við skipunum eins og `nginx -s reload` til að endurnýja vottorðs skyndiminni tengdra forrita.

## Byggja SMTP miðlara með chasquid

[chasquid](https://github.com/albertito/chasquid) er opinn SMTP netþjónn skrifaður á Go tungumáli.

Í stað hinna fornu póstþjónaforrita eins og Postfix og Sendmail er chasquid einfaldara og auðveldara í notkun og það er líka auðveldara fyrir aukaþróun.

Keyrðu `./chasquid/init.sh 123.com` verður sjálfkrafa sett upp með einum smelli (skipta um 123.com fyrir sendilénið þitt).

## Stilla tölvupóstundirskrift DKIM

DKIM er notað til að senda tölvupóstundirskriftir til að koma í veg fyrir að farið sé með bréf sem ruslpóst.

Eftir að skipunin hefur keyrt með góðum árangri verðurðu beðinn um að setja DKIM skrána (eins og sýnt er hér að neðan).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Bættu bara TXT færslu við DNS (eins og sýnt er hér að neðan).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Skoða þjónustustöðu og annála

 `systemctl status chasquid` Skoða þjónustustöðu.

Staða eðlilegrar notkunar er eins og sýnt er á myndinni hér að neðan

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` eða `journalctl -xeu chasquid` getur skoðað villuskrána.

## Snúið uppsetningu lénsheita

Hið gagnstæða lén er til að leyfa að IP-tölu sé leyst í samsvarandi lén.

Að stilla öfugt lén getur komið í veg fyrir að tölvupóstur sé auðkenndur sem ruslpóstur.

Þegar pósturinn er móttekinn mun móttökuþjónninn framkvæma andstæða lénsgreiningu á IP-tölu sendiþjónsins til að staðfesta hvort sendandi miðlarinn hafi gilt andstæða lén.

Ef sendiþjónninn er ekki með öfugt lén eða ef öfugt lénið passar ekki við IP-tölu sendiþjónsins, gæti móttökuþjónninn viðurkennt tölvupóstinn sem ruslpóst eða hafnað honum.

Farðu á [https://my.contabo.com/rdns](https://my.contabo.com/rdns) og stilltu eins og sýnt er hér að neðan

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Eftir að hafa stillt andstæða lénið, mundu að stilla áframupplausn lénsins ipv4 og ipv6 á netþjóninn.

## Breyttu hýsingarheitinu chasquid.conf

Breyttu `conf/chasquid/chasquid.conf` í gildi hins gagnstæða léns.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Keyrðu síðan `systemctl restart chasquid` til að endurræsa þjónustuna.

## Backup conf í git geymslu

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Til dæmis afrita ég conf möppuna í mitt eigið github ferli sem hér segir

Búðu til einkavöruhús fyrst

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Sláðu inn conf skrána og sendu til vöruhússins

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Bæta við sendanda

hlaupa

```
chasquid-util user-add i@wac.tax
```

Getur bætt við sendanda

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Staðfestu að lykilorðið sé rétt stillt

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Eftir að notandanum hefur verið bætt við verður `chasquid/domains/wac.tax/users` uppfært, mundu að senda það inn á vöruhúsið.

## DNS bætir við SPF skrá

SPF ( Sender Policy Framework ) er tölvupóststaðfestingartækni sem notuð er til að koma í veg fyrir tölvupóstsvindl.

Það sannreynir auðkenni sendanda pósts með því að athuga að IP-tala sendandans passi við DNS-skrár lénsins sem það segist vera, og kemur í veg fyrir að svikarar sendi falsa tölvupósta.

Að bæta við SPF færslum getur komið í veg fyrir að tölvupóstur sé auðkenndur sem ruslpóstur eins mikið og mögulegt er.

Ef lénsþjónninn þinn styður ekki SPF gerð skaltu bara bæta við TXT gerð skrá.

Til dæmis er SPF fyrir `wac.tax` sem hér segir

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF fyrir `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Athugaðu að ég hef `include:_spf.google.com` hér, þetta er vegna þess að ég mun stilla `i@wac.tax` sem sendandi heimilisfang í Google pósthólfinu síðar.

## DNS stillingar DMARC

DMARC er skammstöfun á (Domain-based Message Authentication, Reporting & Conformance).

Það er notað til að fanga SPF hopp (kannski af völdum stillingarvillna, eða einhver annar er að þykjast vera þú til að senda ruslpóst).

Bæta við TXT færslu `_dmarc` ,

Innihaldið er sem hér segir

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

Merking hverrar breytu er sem hér segir

### p (Stefna)

Sýnir hvernig eigi að meðhöndla tölvupóst sem mistakast SPF (Sender Policy Framework) eða DKIM (DomainKeys Identified Mail) staðfestingu. Hægt er að stilla p færibreytuna á eitt af þremur gildum:

* engin: Engar aðgerðir eru gerðar, aðeins staðfestingarniðurstaðan er send til sendanda í gegnum tölvupóstskýrslukerfið.
* Sóttkví: Settu póstinn sem hefur ekki staðist staðfestingu í ruslpóstmöppuna, en mun ekki hafna póstinum beint.
* hafna: Hafna beint tölvupósti sem mistakast staðfestingu.

### fo (bilunarvalkostir)

Tilgreinir magn upplýsinga sem skýrslugerðin skilar. Það er hægt að stilla á eitt af eftirfarandi gildum:

* 0: Tilkynna staðfestingarniðurstöður fyrir öll skilaboð
* 1: Tilkynntu aðeins skilaboð sem mistakast staðfestingu
* d: Tilkynntu aðeins mistök í staðfestingu léns
* s: tilkynntu aðeins bilanir í SPF sannprófun
* l: Tilkynntu aðeins DKIM sannprófunarbilanir

### rua & ruf

* rua (Reporting URI for Aggregate reports): Netfang til að taka á móti uppsöfnuðum skýrslum
* ruf (Reporting URI for Forensic reports): netfang til að fá nákvæmar skýrslur

## Bættu við MX færslum til að framsenda tölvupóst í Google Mail

Vegna þess að ég fann ekki ókeypis fyrirtækjapósthólf sem styður alheimsföng (Catch-All, get tekið á móti hvaða tölvupósti sem er sendur á þetta lén, án takmarkana á forskeytum), notaði ég Chasquid til að áframsenda allan tölvupóst í Gmail pósthólfið mitt.

**Ef þú ert með þitt eigið gjaldskylda viðskiptapósthólf skaltu ekki breyta MX og sleppa þessu skrefi.**

Breyta `conf/chasquid/domains/wac.tax/aliases` , stilla áframsendingarpósthólf

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` gefur til kynna alla tölvupósta, `i` er netfangsforskeyti sendinotandans sem búið er til hér að ofan. Til að framsenda póst þarf hver notandi að bæta við línu.

Bættu síðan við MX færslunni (ég bendi beint á heimilisfang hins gagnstæða léns hér, eins og sýnt er í fyrstu línu á myndinni hér að neðan).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Eftir að uppsetningunni er lokið geturðu notað önnur netföng til að senda tölvupóst á `i@wac.tax` og `any123@wac.tax` til að sjá hvort þú getir tekið á móti tölvupósti í Gmail.

Ef ekki, athugaðu chasquid log ( `grep chasquid /var/log/syslog` ).

## Sendu tölvupóst á i@wac.tax með Google Mail

Eftir að Google Mail fékk póstinn vonaðist ég að sjálfsögðu til að svara með `i@wac.tax` í stað i.wac.tax@gmail.com.

Farðu á [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) og smelltu á "Bæta við öðru netfangi".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Sláðu síðan inn staðfestingarkóðann sem berast í tölvupóstinum sem var sendur til.

Að lokum er hægt að stilla það sem sjálfgefið heimilisfang sendanda (ásamt möguleikanum á að svara með sama heimilisfangi).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Þannig höfum við lokið stofnun SMTP póstþjónsins og notum um leið Google Mail til að senda og taka á móti tölvupósti.

## Sendu prófunarpóst til að athuga hvort uppsetningin heppnist

Sláðu inn `ops/chasquid`

Keyrðu `direnv allow` að setja upp ósjálfstæði (direnv hefur verið sett upp í fyrra eins lykla frumstillingarferli og krók hefur verið bætt við skelina)

þá hlaupa

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

Merking breytanna er sem hér segir

* notandi: SMTP notandanafn
* pass: SMTP lykilorð
* til: viðtakanda

Þú getur sent prufupóst.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

Mælt er með því að nota Gmail til að taka á móti prófunarpósti til að athuga hvort stillingarnar gangi vel.

### TLS staðlað dulkóðun

Eins og sést á myndinni hér að neðan er þessi lítill lás, sem þýðir að SSL vottorðið hefur verið virkt.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Smelltu síðan á „Sýna upprunalegan tölvupóst“

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Eins og sýnt er á myndinni hér að neðan sýnir upprunalega póstsíða Gmail DKIM, sem þýðir að DKIM stillingin hefur tekist.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Athugaðu móttekið í haus upprunalega tölvupóstsins og þú getur séð að heimilisfang sendanda er IPV6, sem þýðir að IPV6 hefur einnig verið stillt með góðum árangri.
