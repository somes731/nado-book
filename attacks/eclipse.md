\newpage
## Атаки затмения {#sec:eclipse}

\EpisodeQR{17}

Атака затмения — это тип атаки, которая изолирует биткоин-узел, занимая все его слоты подключения, чтобы заблокировать узел от получения каких-либо транзакций и блоков, кроме тех, которые были отправлены ему злоумышленником. Это не позволяет узлу видеть, что происходит в сети Биткоина, и потенциально даже может обманом заставить узел принять альтернативную ветвь блокчейна Биткоина. Хотя узлы никогда не примут недействительную транзакцию или блок, атака затмения все же может, как мы увидим, причинить вред.

В этой главе обсуждается, как такой тип атаки можно использовать для обмана пользователей и майнеров. В ней также рассказывается о решениях для противодействия такому типу атак, некоторые из которых были опиисаны в статье 2015 года «Атаки затмения на одноранговую сеть Биткоина», написанной Итаном Хейлманом, Элисон Кендлер, Авивом Зохаром и Шэрон Голдберг из Бостонского университета и Еврейского университета/MSR Israel.^[<https://cs-people.bu.edu/heilman/eclipse/>] Многие из решений, предложенных в этой статье, постепенно внедрялись в программное обеспечение Bitcoin Core в течение нескольких последних лет. В этой главе также обсуждаются некоторые решения, которых не было в статье.

### Почему атаки затмения болезненны

При нормальных обстоятельствах ваш узел подключается к внешнему миру через так называемые «исходящие узлы» числом до восьми. Внешний мир также может подключиться к вам, и для этого ваш узел допускает максимум 117 входящих пиров. В случае атаки затмения ваш узел видит только атакующего и подключается только к нему. Таким образом, вы можете думать, что разговариваете со всем миром, но на самом деле вы разговариваете только с одним человеком. Другими словами, этот человек затмевает ваш взгляд на мир.

Причина, по которой вы подключаетесь ко всем этим узлам, заключается в том, что вам нужно запрашивать у них новые транзакции и блоки. Данные передаются в обоих направлениях, независимо от того, является ли одноранговый узел входящим или исходящим. Ваши пиры могут спонтанно отправлять вам блоки и транзакции, а ваш узел, в свою очередь, будет пересылать их другим. Пока вы подключены хотя бы к одному честному узлу, вы не пропустите последние блоки и транзакции.

Атака затмения происходит, если все одноранговые узлы, к которым вы подключены, контролируются одним объектом, который вас атакует. Теперь они решают, какие блоки и транзакции будет видеть ваш узел, и вы вполне можете упустить самые последние и лучшие из них.

Но почему это важно? Они не могут отправить вам недействительный блок с фальшивой подписью, которая сделает их миллиардерами. Ваш узел по-прежнему проверяет все условия и никогда не примет подобного. Но что они могут сделать, так это провести против вас атаку двойной траты.

Допустим, вы ожидаете денег от кого-то, в данном случае от злоумышленника или кого-то, с кем он вступил в сговор^[На практике жертвы часто знают, кто применил к ним двойную трату. Так, например, не рекомендуется дважды тратить деньги с биржи, если вы только что загрузили туда копию своего паспорта. Автор называет это доказательством отсидки (proof-of-prison). Вероятно, поэтому атаки с двойной тратой случаются редко, даже на альткойнах с гораздо более низким бюджетом на безопасность майнинга, чем у Биткойна (ниже мы перейдем к роли доказательства работы).], и вы видите, что их транзакция появляется в вашем мемпуле — где ждут подтверждения валидные транзакции — но она еще не в блоке.

Вы можете считать этот платеж завершенным и доставить за него товар или услугу. Но оказывается, что за вашей спиной они отправили альтернативную транзакцию в остальную часть сети, которая вам _не_ платит. Эта альтернативная транзакция двойной траты ^ [То, что тратится дважды — это одна или несколько монет, которые формируют вход для транзакции. Биткоин-транзакция принимает монеты на входе и создает новые монеты на выходе. Вход может быть потрачен только один раз. Когда узел видит две транзакции, которые тратят одни и те же входы, он должен выбрать одну и проигнорировать другую. То же самое касается майнеров, которые решают, какой кандидат на транзакцию окажется в блоке.] включается в блок, но ваш злоумышленник скрывает этот блок от вас. Таким образом, с вашей точки зрения, эта транзакция все еще находится в мемпуле, и вы по-прежнему не в курсе того факта, что на самом деле вы никогда не получите эти монеты.

Это самый простой пример атаки затмения. И это еще одна причина, по которой принимать транзакции с нулевым подтверждением — плохая идея. Смысл существования доказательства работы состоит в предотвращении двойных трат, поэтому надо, чтобы транзакция была включена в блок, и тогда вы получите защиту при помощи доказательства работы в этом блоке и во всех блоках поверх него.

В дни, предшествовавшие появлению Биткоина, проблема двойной траты была повсюду, и ее считали решаемой только путем привлечения доверенной третьей стороны. Например, при получении любой суммы в e-cash Чаумана программное обеспечение на вашем компьютере немедленно сообщит вашему банку, что вы получили токены. Затем компьютер банка проверит, что эти токены никогда раньше не видели. Без этого шага одни и те же электронные деньги могли бы быть уплачены многим разным получателям.^[Вот краткое описание электронных денег Чаумана: <https://bitcoin.stackexchange.com/a/10666/4948>. Чтобы получить более подробное объяснение, а также предлагаемый вариант, легший в основу Биткоина, послушайте 52-й эпизод подкаста_Биткойн, разъяснения: <https://nadobtc.libsyn.com/federated-ecash-episode-52>]

Но даже если вы дождетесь подтверждения, вы еще не выбрались из леса. Атака затмения, как мы увидим, все еще возможна, но она будет уже намного дороже. Это связано с тем, что злоумышленник должен создать валидный блок, а валидность блока частично обусловлена предоставлением доказательства работы.

Допустим, вы продали что-то дорогое. Как и в предыдущем примере, атакующий сначала отправляет вам свою транзакцию, а затем отправляет конфликтующую транзакцию остальному миру, при этом следя за тем, чтобы вы ее никогда не увидели. Эта конфликтующая транзакция подтверждается, но атакующий не пересылает вам этот новый блок. Вместо этого он майнит свой собственный блок с исходной транзакцией в нем. Ваш узел сообщает вам, что транзакция подтверждена, и вы предоставляете товар или услугу.

Между тем, внешний мир обычных майнеров продолжает производить блоки, а злоумышленник продолжает прятать эти обычные блоки от вас. Атакующий не будет создавать новые блоки, потому что у него уже есть то, что ему от вас нужно. В какой-то момент он прекращает атаку, либо вы сами выясняете, что произошло, и вмешиваетесь. В любом случае, ваш узел снова подключается к легитимным узлам. Затем он узнает о более длинной цепочке, которая продолжала расти во время атаки. Несмотря на то, что в этой более длинной цепочке вам никогда не платили, ваш узел все равно переключается на нее. Полученные вами монеты, в свою очередь, исчезают.

Атака с двойной тратой также возможна и без атаки затмения, но вероятность ее успеха гораздо меньше. Во-первых, вы можете заметить конфликтующую транзакцию в собственном мемпуле и проявить дополнительную осторожность перед поставкой каких-либо товаров или услуг. И, во-вторых, если предположить, что атакующий контролирует 10 процентов хэш-мощности; его атака в 90% случаев потерпит неудачу, потому что все созданные им блоки, содержащие конфликтующую транзакцию, устаревают. ^ [Когда два блока строятся поверх одного и того же блока, оба одинаково валидны. По мере того, как майнеры продолжают генерировать больше блоков, они выбирают один из этих блоков для построения поверх него (обычно тот, который они увидели первым). В конце концов, связь разрывается, и одна цепочка становится длиннее. Блок, который больше не является частью самой длинной цепочки, называется устаревшим. Сайт <https://forkmonitor.info/> отслеживает эти события и показывает оповещение, как только два блока появляются на одной высоте. Он также отслеживает, сколько блоков построено с каждой стороны, пока одна сторона не проиграет гонку, и ее блоки не устареют.]

Если вы дождетесь двух подтверждений, шансы злоумышленника упадут до 1%. Это хорошее напоминание о том, почему для Биткоина важно быть довольно-таки дорогим, чтобы создавать блоки в альтернативных ветках, которые дважды тратят монеты, было не слишком легко. Еще в 2015 году, когда была написана статья, о которой мы упоминали выше, эти атаки были намного дешевле: 5000–10000 долларов за блок.^[<https://bitcoinvisuals.com/chain-block-reward>] Таким образом, атакующий имеет стимул атаковать вас, если стоимость создания фальшивого блока меньше, чем сумма денег, за которую вас обманывают. Обратите внимание, что на практике некто не может просто заказать индивидуальный блок для своей атаки, если только он не _является_ майнером. Поскольку любая атака с двойной тратой нанесет ущерб репутации Биткоина, что уменьшит доходы майнеров, они не особенно заинтересованы в содействии таким атакам. ^[Однако, когда одно и то же оборудование можно использовать для майнинга нескольких монет, то нанесение некоторого ущерба репутации одной монеты оставляет возможность для добычи множества других монет. Существуют платформы, на которых люди могут арендовать хэш-мощность, и иногда они используются для проведения атаки с двойной тратой. Такая атака может быть очень дешевой, особенно когда у монеты очень мало хэш-мощности. См., например. <https://www.coindesk.com/markets/2020/08/07/ethereum-classic-attacker-successfully-double-spends-168m-in-second-attack-report/>]

Производство блока стоит денег, в основном на оборудование и электричество. Честный майнер компенсирует это, продавая или одалживая монеты, созданные в транзакции coinbase. Это первая транзакция в каждом блоке, и для нее действуют особые правила. Первое правило заключается в том, что, в отличие от обычных транзакций, транзакция coinbase не имеет входов. Она создает деньги из ниоткуда, но сумма ограничена суммой субсидии блока (в настоящее время 6,25 BTC и уменьшается вдвое каждые четыре года) и всеми комиссиями, уплачиваемыми транзакциями, включенными в блок. Выходная сторона транзакции отправляет эту сумму туда, куда хочет майнер, но обычно это адрес, управляемый майнинг-пулом, который затем перераспределяет полученное.

Обычно майнер создает блок, который строится поверх последнего известного ему добытого блока. Между тем, транзакция coinbase имеет второе применяющееся всеми узлами специальное правило, ^ [Кроме того, как мы объясняли в главе @sec:segwit, SegWit налагает третье специальное правило на транзакцию coinbase: она должна содержать вывод `OP_RETURN` с хэшем свидетельств для данного блока.] которое говорит о том, что она не может быть потрачена до тех пор, пока не будет получено 101 подтверждение, то есть пока поверх него не будет построен 101 блок. Это называется зрелостью coinbase.^[<https://bitcoin.stackexchange.com/a/1992/4948>]

Атакующий не заботится о вознаграждении в coinbase, потому что он может (гипотетически) больше заработать на мошенничестве с вами. Вместо того, чтобы строить поверх самого последнего блока, он создает блок поверх последнего блока, о котором вы знаете. И он не транслирует этот блок в мир, поэтому никакие другие майнеры не будут майнить поверх него. Это означает, что его транзакция coinbase никогда не достигнет зрелости, поэтому затраты на производство блока не могут быть возмещены напрямую. Для майнера нет экономического смысла делать это, если только он не получит прямую выгоду от атаки… или если его не обманут, как мы объясним ниже.

Оказывается, атакующий также может попытаться отколоть от сети майнеры, и таким образом использовать их против вас без сотрудничества с ними. Чтобы это сделать, он использует атаку затмения не только против вас, но и против одного или нескольких майнеров. Затмеваемые майнеры, предположительно, меньшинство, увидят ту же транзакцию, что и вы, фактическая целевая жертва. После того, как они произведут блок, атакующий гарантирует, что ваш узел — единственный, кто увидит этот блок. Эти майнеры тоже оказываются жертвами, потому что когда блок в конечном итоге устареет, их транзакция coinbase исчезает точно так же, как и ваша транзакция. Нанося весь этот экономический ущерб, атакующий может украсть у вас всего сотню долларов. Поэтому нам действительно важно, чтобы атаки затмения были очень сложными.

Майнеры и операторы пулов, конечно, не наивны. Они могут запускать несколько узлов в разных странах и принимать меры предосторожности, чтобы злоумышленник не знал, какой узел затмить. Кроме того, майнинг по-прежнему несколько централизован, поэтому существуют специализированные сети, которые их соединяют, что еще больше затрудняет атаки затмения.^[Одной из таких сетей для подключения майнеров является Fast Internet Bitcoin Relay Engine (FIBRE): <https://bitcoin .stackexchange.com/questions/56485/can-someone-please-explain-fibre-to-me-like-im-5-and-why-is-it-useful>] Но это не должно быть единственным, на что мы полагаемся для предотвращения подобных атак. К счастью, внедряется все больше и больше улучшений, призванных усложнить эти атаки.

### Как работает атака затмения

До сих пор мы лишь предполагали, что атака затмения может быть проведена, в порядке объяснения, как она используется, чтобы обманом заставить вас расстаться с вашими с трудом заработанными монетами. Но как она фактически осуществляется?

Напомним сказанное выше о том, что для затмевания вашего узла злоумышленнику необходимо захватить все восемь исходящих соединений и любое количество входящих соединений, которые есть у вашего узла. Это игра в кошки-мышки, и даже до того, как была написана вышеупомянутая статья, приложение Bitcoin Core было усилено для предотвращения атак затмения. Но давайте посмотрим, как, согласно статье, предполагается преодоление существующей защиты.

Есть пара ингредиентов. Во-первых, как упоминалось в главе @sec:dns, когда узел запускается, он пытается найти другие пиры, и после того, как он некоторое время поработает, у него появится список адресов, которые он узнал от других пиров и сохранил в файле. Затем всякий раз, когда узел теряет одно из восьми своих исходящих соединений или когда перезапускается, он просматривает этот файл со всеми адресами, о которых он когда-либо слышал, и начинает случайным образом подключаться к ним.

Идея атакующего состоит в том, чтобы загрязнить этот файл, предоставив вашему узлу кучу адресов, которые либо не существуют, либо которые он (атакующий) контролирует. Таким образом, какой бы адрес ни выбрал ваш узел, каждый раз, когда он устанавливает соединение, он либо терпит неудачу, потому что там ничего нет, либо подключается к атакующему — и в конечном итоге все соединения будут вести к нему.

Атакующему также необходимо контролировать все входящие подключения к вашему узлу. Не вдаваясь в этой главе в подробности, укажем, что один из подходов состоит в том, чтобы просто делать много-много попыток подключения, пока все ваши 117 входящих слотов не будут заполнены. Со временем, возможно, в течение нескольких недель, когда честные пиры время от времени отключаются от вас, злоумышленник быстро заполняет открытые входящие слоты, не давая подключиться новым честным пирам. <https://github.com/bitcoin/bitcoin/pull/6374>]

Еще в 2012 году разработчики поняли, что атакующий может дать вашему узлу огромное количество контролируемых им IP-адресов. Допустим, у вашего узла есть 1000 реальных IP-адресов других узлов. Затем атакующий скармливает вам 9000 адресов, которые он контролирует. Когда ваш узел начинает выбирать IP-адреса, вероятность того, что он подключится к атакующему, составляет 90 процентов.

Но пока эти адреса тесно связаны, например, все они находятся в одном дата-центре, тут еще можно что-то сделать. Была введена система сегментов, которая помещает все IP-адреса с одинаковыми двумя начальными цифрами, например. `172.67.*.*` в один сегмент. Затем узел будет пиры для каждого исходящего соединения из разных сегментов.^[<https://github.com/bitcoin/bitcoin/pull/787>]

В приведенном выше примере все IP-адреса атакующего оказываются в одном сегменте, а таких сегментов 256, поэтому шансы подключиться даже к одному узлу злоумышленника резко падают. Имейте в виду, что для защиты от атак затмения вам достаточно только _одного_ честного пира.

Каждый сегмент также ограничен по размеру, поэтому большинство из 9000 адресов в приведенном выше примере будут выброшены из своего сегмента почти сразу же, как только туда попадут.

Наконец, в рамках того же предложения по редакции кода узлы также начали запоминать, к каким узлам они ранее подключались. Всякий раз, когда им нужно было новое соединение, они бросали монету и либо подключались к одному из них, либо выбирали новое из одного из 256 сегментов.

### The Botnet

You might think this would do the trick, but here’s where the paper comes into play. The authors ran a simulation to see how difficult it was to actually overflow all these buckets, and it found that, within a matter of days, it can be successful.

How did they do this? By using a botnet^[<https://en.wikipedia.org/wiki/Botnet>] — not a real one of course, as that would probably be unethical for university researchers, not to mention potentially illegal. But they simulated one. A botnet is a group of random computers in the world that have been hacked and can be remote controlled. Because they’re not all in the same data center as our example above, their IP addresses have many different starting digits, so they end up in different buckets.

The paper estimated that a botnet with less than 5,000 computers can successfully pull off an eclipse attack. That might sound like a big botnet, but you can rent that from various nefarious “companies” for less than $100.^[Business Model of a Botnet: <https://arxiv.org/pdf/1804.10848.pdf>]

In addition to attacking your node from many different directions, thereby defeating the bucket system, the hypothetical attacker in the paper also exploited other weaknesses.

First, they would flood your node with IP addresses that are known to be fake. This would flush all buckets with fake nodes. Remember that when your node needs a new peer, it’ll toss a coin to either connect to familiar node or try a new one. Well, there wouldn’t be any new ones to try.^[We’ll revisit the problem of fake nodes in chapter @sec:fake_nodes.]

For the other side of the coin flip — connecting to a familiar node — the attackers exploited another weakness. It turns out your node considers any node it ever connected to “familiar.” That includes botnet nodes that connected _to_ it, even if only briefly.^[Fixed in 2016: <https://github.com/bitcoin/bitcoin/pull/8594>] There’s a separate 64-bucket system for these familiar nodes, and over time, they get filled up by botnet IPs.

### Don’t Crash

At this point, your node still has long-lived connections to the real world from before the attack began, so the attacker still needs to get rid of those. The trick is to either wait for your node to restart, or to try and crash it.

Whenever your node restarts,^[Nodes that run on a server are typically automatically restarted after a crash or system reboot, using something like systemd: <https://en.wikipedia.org/wiki/Systemd>] it starts out with zero connections. Firstly, this creates an opportunity to very quickly fill up all 117 inbound slots. And secondly, it’s going to look at that file of peers it knows, and it’s going to try and connect to them. If an attacker succeeded at dominating these buckets, your node is exclusively going to connect to attacker IP addresses. That’s all that’s needed for the eclipse attack to be in play.

So although crashing a Bitcoin node isn’t a very useful attack on its own, it can help when performing an eclipse attack. This is one reason why it’s important for developers to ensure they don’t write code that can make a node crash.

### How to Solve It

It’s important to understand that attacks like these are a numbers game. An attacker needs to give your node a lot of spam addresses to fill up all the buckets and make sure it only connects to you.

So one obvious mitigation^[mitigate — “to cause to become less harsh or hostile”: <https://www.merriam-webster.com/dictionary/mitigate>\
A mitigation isn’t a complete solution. Although a bit redundant, the term “partial mitigation” is often used as well.] of an attack like this is to have more buckets. Unfortunately, this doesn’t help much with isolation, because doubling the number of buckets only doubles the attack cost,^[O(n): The cost of an attack grows linearly with the number of buckets (n). A much stronger defense is something that increases the cost of an attack quadratically - O(n^2^) - or even exponentially. <https://www.freecodecamp.org/news/big-o-notation-why-it-matters-and-why-it-doesnt-1674cfa8a23c/>] and we already saw how cheap it is. Still, the number of buckets was quadrupled almost immediately after the paper was published.^[<https://github.com/bitcoin/bitcoin/pull/5941/commits/1d21ba2f5ecbf03086d0b65c4c4c80a39a94c2ee>]

Another countermeasure lies in the aforementioned coin toss. This toss was actually biased toward trying new nodes and toward those that your node most recently learned about. This was changed to just a coin toss (in that same early pull request). Why not go further and only connect to peers your node has known the longest? There are always tradeoffs — in this case, your node might spend too much time going through a list of no-longer-reachable IP addresses.

But there was another proposed mitigation that also provided a bias toward familiar nodes, only in a safer way. It pertained to how buckets are handled when they’re about to overflow. When you hear of a new address and you want to put it in a bucket and remove something else, you first check the address that’s already in the bucket. That entails connecting to it to see if it still exists. If it does exist, you don’t replace it. This is called a feeler connection. This was more complicated and it took until mid-2016 to be implemented.^[By one of the authors in fact: <https://github.com/bitcoin/bitcoin/pull/8282>]

Still, other mitigations took much longer. When Bitcoin Core 0.21.0 was released in January 2021, it included a new method to prevent eclipse attacks that was suggested in this same 2015 paper: the use of anchor connections.^[<https://github.com/bitcoin/bitcoin/pull/17428>] What happens is that when you restart, you try to remember some of the last connections you had. Your node remembers the two connections that it only exchanges blocks with, and it tries to reconnect to those.

Why two? It’s not a good idea to always try to reconnect to the same nodes again when you restart, as, for all you know, the reason you crashed in the first place is because one of those nodes was evil. The same logic applies to the scenario where you’re _already_ being eclipsed.^[<https://github.com/bitcoin/bitcoin/issues/17326#issuecomment-550521262>]

### What Else Can Be Done?

In addition to the many suggestions from the paper, there are other things that can be done, and some have been implemented.

You may be wondering: Why wouldn’t you just have as many connections as possible from the get-go? But the problem is that it requires a lot of data exchange — especially for the transactions in a mempool — and that’s extremely data intensive, so you can’t just add more connections without also increasing bandwidth use.

Erlay (see appendix @sec:more_eps) is a proposal for reducing the bandwidth needed for these mempool synchronizations. It reduces the main cost (bandwidth) _per connection_. A lower cost per connection allows nodes to have more connections. Having more connections makes any eclipse attack scheme more difficult.

Another way to have more connections without increasing bandwidth too much is to constrain some connections to blocks only, and to not sync the mempool with those peers. This was implemented in 2019.^[<https://github.com/bitcoin/bitcoin/pull/15759>]

Finally, there’s the Blockstream Satellite,^[<https://blockstream.com/satellite/>] or any other satellite or even radio broadcast.^[<https://www.wired.com/story/cypherpunks-bitcoin-ham-radio/>] These allow anyone in the world to receive the latest blocks. This is mainly useful for people with very low bandwidth internet connections in remote areas. But it can also offer protection against an eclipse attack. This is because when your node receives the satellite signal, even if all inbound and outbound connections are taken over by an attacker, you’ll still learn about new blocks.

Note, however, that you shouldn’t blindly trust the satellite either, for _it_ might try to eclipse you. But remember that you only need a single honest peer, and you achieve this by having as diverse a set of connections as possible.

The Bitcoin Core development wiki also contains an overview of eclipse attacks and various counter measures.^[<https://github.com/bitcoin-core/bitcoin-devwiki/wiki/Addrman-and-eclipse-attacks>]

### Erebus Attack

\EpisodeQR{18}

If you want to learn more about eclipse attacks, you might be interested in the Erebus attack^[<https://erebus-attack.comp.nus.edu.sg>]: an eclipse attack where an attacker essentially spoofs an entire part of the internet.

How this works is the internet is made up of Autonomous Systems (AS), which are basically clusters of IP addresses owned by the same entity, like an ISP.^[<https://www.cloudflare.com/learning/network-layer/what-is-an-autonomous-system/>]

As it turns out, however, some Autonomous Systems can effectively act as bottlenecks when trying to reach other Autonomous Systems. This allows an attacker controlling such a bottleneck to launch a successful eclipse attack — even against nodes that connect with multiple Autonomous Systems.

As explained above, Bitcoin Core nodes already counter eclipse attacks by ensuring they’re connected to a variety of IP addresses, based on the first two digits of the IP address. This can be further improved by separating buckets by Autonomous Systems instead.

But this doesn’t thwart the Erebus attack. For that, recent versions of Bitcoin Core include an optional feature — ASMAP.^[<https://blog.bitmex.com/call-to-action-testing-and-improving-asmap/>]

The episode explains how mapping the internet has allowed Bitcoin Core contributors to create a tool which ensures that Bitcoin nodes not only connect to various Autonomous Systems, but also that they avoid being trapped behind said bottlenecks.
