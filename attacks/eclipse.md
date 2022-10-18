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

If you wait for two confirmations, the odds for your attacker drop to 1 percent. This is a good reminder of why it’s important for Bitcoin to be somewhat expensive, so that it’s not too easy to produce blocks on alternative branches that double-spend coins. Back in 2015, when the paper we mentioned above was written, these attacks were a lot cheaper: $5,000–$10,000 per block.^[<https://bitcoinvisuals.com/chain-block-reward>] So, the attacker is only going to attack you if the cost of making a fake block is lower than the amount of money they’re scamming you for. Note that, in practice, someone can’t just order a tailormade block for their attack, unless they _are_ a miner. Since any double-spend attack would harm Bitcoin’s reputation, which would reduce miner revenue, they aren’t particularly incentivized to facilitate such attacks.^[However, when the same equipment can be used to mine multiple coins, then causing some reputation damage on one coin leaves plenty of other coins to mine. There are platforms where people can rent hash power, and those have sometimes been used to perform a double-spend attack. Especially when a coin enjoys very little hash power, such an attack can be very cheap. See e.g. <https://www.coindesk.com/markets/2020/08/07/ethereum-classic-attacker-successfully-double-spends-168m-in-second-attack-report/>]

Если вы дождетесь двух подтверждений, шансы злоумышленника упадут до 1%. Это хорошее напоминание о том, почему для Биткоина важно быть довольно-таки дорогим, чтобы создавать блоки в альтернативных ветках, которые дважды тратят монеты, не было слишком легко. Еще в 2015 году, когда была написана статья, о которой мы упоминали выше, эти атаки были намного дешевле: 5000–10000 долларов за блок.^[<https://bitcoinvisuals.com/chain-block-reward>] Таким образом, атакующий имеет стимул атаковать вас, если стоимость создания фальшивого блока ниже, чем сумма денег, за которую они вас обманывают. Обратите внимание, что на практике кто-то не может просто заказать индивидуальный блок для своей атаки, если только он не является майнером. Поскольку любая атака с двойной тратой нанесет ущерб репутации Биткойн, что уменьшит доходы майнеров, они не особенно заинтересованы в содействии таким атакам. одна монета оставляет много других монет для майнинга. Существуют платформы, на которых люди могут арендовать хэш-мощность, и иногда они используются для проведения атаки с двойной тратой. Такая атака может быть очень дешевой, особенно когда у монеты очень мало хэш-мощности. См., например. <https://www.coindesk.com/markets/2020/08/07/ethereum-classic-attacker-successfully-double-spends-168m-in-second-attack-report/>]

Producing a block costs money, mainly for equipment and electricity. An honest miner recoups this by selling or borrowing against the coins created in the coinbase transaction. This is the first transaction in every block, and it has special rules. The first rule is that, unlike regular transactions, a coinbase transaction doesn’t have inputs. It creates money out of nowhere, but the amount is capped by the sum of the block subsidy (currently 6.25 BTC and halving every four years) and all the fees paid by transactions included in the block. The output side of the transaction sends this to wherever the miner wants, but usually an address managed by a mining pool, which then redistributes it.

Normally, a miner produces a block that builds on top of the most recently mined block that they’re aware of. And the coinbase transaction has a second special rule,^[As an aside, as we explained in chapter @sec:segwit, SegWit imposes a third special rule on the coinbase transaction: It needs to contain an `OP_RETURN` output with a hash of the witness data for the block.] enforced by all nodes, which says that it can’t be spent until it has 101 confirmations, i.e. until there are 101 blocks built on top of it. This is called coinbase maturity.^[<https://bitcoin.stackexchange.com/a/1992/4948>]

The attacker doesn’t care about the coinbase reward, because they stand to make more from scamming you (hypothetically). Instead of building on top of the most recent block out there, they create a block on top of the last block _you_ know of. And they don’t broadcast this block to the world, so no other miners will mine on top of it. This means their coinbase transaction never reaches maturity, so the costs of producing the block can’t be recouped directly. It makes no economic sense for a miner to do this, unless they directly benefit from the attack… or unless they’re duped, as we explain below.

It turns out an attacker can also use miners against you without their cooperation by trying to split miners. They do this by not just eclipsing you, but also by eclipsing one or more miners. The eclipsed miners, presumably a minority, would see the same transaction as you, the actual target victim. Once they mine it, the attacker ensures that your node is the only one that gets to see this block. This miner is a victim too, because just like your transaction ends up disappearing once the block eventually goes stale, their coinbase transaction disappears too. For all this economic damage, the attacker might only rob you of $100. So we really want eclipse attacks to be very difficult.

Miners and pool operators are of course not naive. They might run multiple nodes in different countries and take precautions so that an attacker won’t know which node to eclipse. In addition, mining is still somewhat centralized, so there are specialized networks that connect them, making eclipse attacks even more difficult.^[One such network to connect miners is the Fast Internet Bitcoin Relay Engine (FIBRE): <https://bitcoin.stackexchange.com/questions/56485/can-someone-please-explain-fibre-to-me-like-im-5-and-why-is-it-useful>] But this shouldn’t be the only thing we’re relying on to prevent these attacks. Luckily, more and more improvements that are designed to make these attacks more difficult are being deployed.

### How an Eclipse Attack Works

So far we’ve assumed that an eclipse attack can be done, in order to explain how it’s used to trick you into parting with your hard-earned coins. But how is it actually done?

Recall from above that, in order to eclipse your node, the attacker needs to take over all eight of your outbound connections and whatever number of inbound connections your node has. This is a cat and mouse game, and even before the above-mentioned paper was written, the Bitcoin Core software was hardened to prevent eclipse attacks. But let’s see how the paper proposed overcoming the existing defenses.

There are a couple of ingredients. First, as mentioned in chapter @sec:dns, when a node starts, it tries to find other peers, and once it’s been running for a while, it has a list of addresses it learned from other peers and it stores them in a file. Then, whenever a node loses one of its eight outbound connections, or when it restarts, it looks at this file with all the addresses it’s ever heard of, and it starts randomly connecting to them.

As an attacker, the idea is to pollute this file by giving your node a bunch of addresses that either don’t exist or that they (the attacker) control. This way, whatever address your node picks, every time it makes a connection, it either fails because there’s nothing there, or it connects to the attacker — and eventually all connections are to the attacker.

The attacker also needs to control all inbound connections to your node. Without going into too much detail in this chapter, one approach is to just make lots and lots of connection attempts until all your 117 inbound slots are full. Over time, perhaps weeks, as honest peers occasionally disconnect from you, the attacker quickly fills the open inbound slots so that no new honest peers get through.^[This has been made more difficult by sometimes dropping an existing inbound connection in favor of a new one: <https://github.com/bitcoin/bitcoin/pull/6374>]

As early as 2012, developers realized it was possible for an attacker to give your node huge numbers of IP addresses, all controlled by them. Let’s say your node has 1,000 real IP addresses of other nodes. Then the attacker feeds you 9,000 addresses that they control. As your node starts to pick IP addresses, the odds are 90 percent that it will connect to the attacker.

But as long as these addresses were closely related, e.g. because they were all in the same data center, there was something that could be done. A bucket system was introduced, and it puts all IP addresses with the same two starting digits, e.g. `172.67.*.*` into the same bucket. The node would then pick from different buckets for each of its outbound connections.^[<https://github.com/bitcoin/bitcoin/pull/787>]

In the example above, all the attacker’s IP addresses end up in one bucket, and there are 256 such buckets, so the odds of connecting to even a single attacker node drop dramatically. Keep in mind that you only need _one_ honest peer to be protected against eclipse attacks.

Each bucket is also limited in size, so most of the 9,000 addresses in the example above would be thrown out of their bucket almost as soon as they entered it.

Finally, in the same pull request, nodes also started remembering which nodes they previously connected to. Whenever they needed a new connection, they would toss a coin, and either connect to one of those, or pick a new one from one of the 256 buckets.

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
