\newpage
## Время синхронизации и AssumeUTXO {#sec:assume}

\EpisodeQR{14}

Одно из наиболее узких мест — если не самое узкое — при масштабирования Биткоина — начальная загрузка блокчейна. Это время, которое требуется биткоин-узлу для синхронизации с сетью Биткоина, поскольку ему необходимо обработать все исторические транзакции и блоки, чтобы создать набор последних неизрасходованных выходных данных транзакций (UTXO), то есть текущее состояние владения биткоинами.

В этой главе будут рассмотрены некоторые способы ускорения синхронизации с течением времени. Сначала она был улучшена за счет подхода Headers First, когда первым делом синхронизируются заголовки, и это гарантирует, что новые биткоин-узлы не будут тратить время на проверку (потенциально) более коротких цепочек. Одно из нескольких недавних улучшений, касающихся времени синхронизации, называется Assume Valid (презумпция валидности) — это установка по умолчанию, которая позволяет узлам пропускать проверку подписи старых транзакций, вместо этого полагаясь на то, что процесс разработки Bitcoin Core — в сочетании с ресурсоемким характером майнинга — обеспечивает надежную версию истории транзакций.

Также будет рассказываться, как предположения безопасности, лежащие в основе презумпции валидности, могут быть расширены, чтобы обеспечить потенциальное будущее обновление, AssumeUTXO (презумпция ), чтобы предложить новым пользователям Bitcoin Core быстрое решение для ускорения работы в сети Биткоина путем синхронизации самых последних блоков в первую очередь и проверки исторических блоков в фоновом режиме позже.

Помимо сопровождающего эту главу эпизода подкаста _Разъяснение Биткоина_, вы также можете послушать эпизод _The Chaincode Podcast_ с автором AssumeUTXO Джеймсом О'Бейрном, в котором рассматриваются те же темы, что и в этой главе.^[<https://podcast.chaincode.com /2020/02/12/james-obeirne-4.html>]

### Скачивание блокчейна

Когда вы запускаете свой биткоин-узел, он находит другие узлы и подключается к ним, как мы объясняли в главе @sec:dns. Затем он переходит к загрузке блокчейна.

Самым наивным способом загрузки блокчейна было бы просто попросить своих коллег прислать вам все, что у них есть. Это не очень хорошая идея, потому что, если один из узлов, к которым вы подключились, окажется злонамеренным, он может обманом заставить вас загрузить терабайты поддельных блоков, пока у вас не закончится место на диске и ваш узел не выйдет из строя.

Чтобы предотвратить такое злоупотребление, первоначальная версия Биткоина сначала запрашивала у узлов заголовок блока. Этот заголовок включает в себя доказательство работы, которое, как следует из названия, доказывает, что для создания блока была проделана определенная работа. Проверка доказательства в этом заголовке перед извлечением самого блока усложняет для злоумышленника создание достаточного количества поддельных блоков, чтобы переполнить ваш жесткий диск. Как только блок выбран и проверен, ваш узел запрашивает следующий заголовок и т. д., последовательно обрабатывая заголовки и блоки.^[По факту процесс был немного сложнее. Когда узел получал блок, который не находился напрямую на вершине цепочки, он, как выразился Сатоши в комментарии к исходному коду, «перенаправлял его в область хранения». Оттуда его можно было позже присоединить к концу цепи. Эти блоки назывались _сиротскими блоками_, этот термин часто путают с _устаревшими блоками_.]

Хотя это защищает от самой простой формы спама блоками, мы еще не выбрались из леса. Такой подход все еще очень близорук, поскольку ваш узел проверяет только блоки прямо перед собой, не видя общей картины.

Проблема в том, что вы не знаете, не идете ли вы в тупик, и кто-то может скормить вашему узлу длинную ветвь блоков, которые не являются частью наиболее длинной цепи доказательства работы. Узлы, которые только что подключились к сети, особенно перед этим уязвимы. Это связано с тем, что сложность доказательства работы исторически увеличивалась. Создание тупиковых ветвей, начинающихся с последних блоков, обходится дорого, потому что многие майнеры конкурируют за производство блоков, что увеличивает стоимость создания блока. Но если злоумышленник начинает с очень старого блока, из тех времен, когда майнеров было меньше, и они были менее мощными, то стоимость производства таких блоков очень низкая.

Таким образом, злоумышленник может создать цепочку блоков очень низкой сложности, которые ответвляются от какого-то старого блока. Если ваш узел новичок в этом городе, то когда он видит две или даже тысячи возможных ветвей, он не знает, какая из них настоящая. Если он сначала выберет ветку от злоумышленника, он может потратить много времени и ресурсов компьютера на проверку блоков. Несмотря на то, что сложность создания этих блоков невелика, узлу от этого не легче проверять транзакции; эти тупиковые ветки могут быть заполнены под завязку мегабайтными блоками со специально созданными транзакциями, которые проверяются очень медленно.

Помимо атак увязания узлов в тупиковых ветках с блоками низкой сложности, существует также опасность атак затмения, которые мы рассмотрим в главе @sec:eclipse.

### Контрольные точки

Одним из решений этой проблемы было использование контрольных точек: разработчики помещали в исходный код хэш и высоту нескольких известных валидных блоков, и любой новый блок, который не происходит от одной из этих контрольных точек, может смело игнорироваться. Это не полностью устранило проблему тупиковых веток, но ограничило их максимальную длину.

Недостатком контрольных точек является то, что они потенциально дают разработчикам слишком много возможностей. Злонамеренная клика разработчиков или благонамеренный диктатор, делающий то, что лучше для сообщества — в зависимости от того, какую точку зрения вы предпочитаете — могут объявить определенный блок действительным. Даже если существует альтернативная ветка с большим количеством работы, узлы не будут рассматривать эту ветку.

Возможно, разработчик потерял свои биткоины в результате взлома; после этого разработчики могут ввести злонамеренную контрольную точку прямо перед перемещением взломанных монет и переместить свои монеты в безопасное место в исправленной истории. Такая атака не может происходить тайно, и если бы это действительно произошло, пользователи могли бы просто отказаться устанавливать новое программное обеспечение узла с контрольной точкой. Но лучше все-таки предотвращать подобное.

Последняя контрольная точка была добавлена в конце 2014 года. Избавиться от потребности в контрольных точках удалось различными способами, включая введение `nMinimumChainWork` в 2016 году.^[<https://github.com/bitcoin/bitcoin/pull/9053>] Этот параметр указывает, сколько доказанной работы должна продемонстрировать любая цепочка заголовков, прежде чем получит право на рассмотрение. Но чтобы такое работало, узлы должны быть менее близорукими; им нужно сперва разглядеть, _куда_ ведет данная цепочка блоков, прежде чем тратить много компьютерной мощности на ее изучение. И тут на помощь приходит Headers First.

### Headers First

При наличии достаточного количества времени ваш узел - если не выйдет из строя - сравнит все ветки блокчейна и в конечном итоге выберет обладающую наибольшей накопленной сложностью. Но поскольку сначала необходимо проверить каждую ветвь, выявление правильной может занять очень много времени.

Таким образом, вместо того, чтобы загружать и проверять целые блоки, новый подход заключается в загрузке и проверке только заголовков, которые весят намного меньше. В частности, заголовки — это единственное, что вам нужно для определения совокупной сложности доказательства работы в любой заданной ветке.

Как только ваш узел узнает, какая ветка имеет больше всего накопленной работы, он загружает для нее блоки и начинает проверку. Этот шаг нельзя пропустить, потому что все еще существует вероятность того, что в цепи с наибольшим количеством доказанной работы есть недопустимый блок. Если ваш узел сталкивается с таким недопустимым блоком, он отбрасывает ветку и повторяет процесс для той ветки, которая имеет второе по величине количество доказанной работы.

### Assume Valid

Assume Valid is a block hash that’s encoded in the software. More specifically, it’s a hash of a block from just before the last major release. Many Bitcoin Core developers publicly verify this hash, and anyone on GitHub can see the hash, and they can check for themselves whether or not the hash is real.

If you’re a new user and you start Bitcoin Core, it’ll sync all the headers and get all the blocks. And if that particular hash is in the chain, it won’t verify any signatures that came before it. It’ll still verify everything else, e.g. that the proof-of-work is valid and that no coins are created out of thin air. Skipping signature verification mainly saves CPU usage, and this speeds up the whole process.

The Assume Valid mechanism is different from a checkpoint, in that a node doesn’t require that the hash occurs in the blockchain. If your node sees another branch of the blockchain without this hash, and if it has more proof-of-work, it’ll consider that other branch first. The only difference is that it _will_ verify the signatures for that other branch, so it’ll take a bit longer.

What does it mean to not check signatures for blocks before the Assume Valid hash? It means that if somebody stole a coin, i.e. spent a UTXO with an invalid signature, your node wouldn’t notice. But if someone created a coin out of thin air, your node would still see that.

This is where the transparency of source code becomes an important factor. _If_ there ever was a theft of coins that Bitcoin Core developers wanted to cover up, they’d only be able to trick new nodes. First, the developers would have to produce a block that steals^[This hypothetical attack can’t create coins out of nowhere, so the victim of this theft might also make some noise. However, many coins are thought to be permanently lost — e.g. because the owner lost their private keys — and those coins could make a good target. Since Satoshi disappeared, stealing coins that are allegedly his could make sense, but those coins are being very closely monitored by many people. Any block attempting to steal them, even if invalid, would probably get some media attention.] coins by using an invalid signature. Such a block would be considered invalid by all existing nodes. Then, they’d take this invalid block, or a descendant, and use its hash as Assume Valid.

Anyone who already runs a node would be able to see this hash on GitHub and check it against their own node. They’d then either not see the block at all, or their node would point out that it’s invalid (because of the invalid signature). Both would be reason to sound the alarm. Barring some immense social media censorship campaign, anyone about to download a new node might learn what’s going on.

But the hypothetical malicious developers have another problem. No miners are building on top of their invalid block, because the miners already had their node software installed before the invalid hash was produced. Within hours of the developers publishing this hash, and long before they release any software for download, miners have already produced a longer chain that doesn’t include this stealing block. So even if a user didn’t notice the social media drama, their node would simply follow the longest chain. It’d be a bit slower because it couldn’t use the Assume Valid feature, but it’d be fine.

So, what if developers colluded with miners in the theft? If a majority of miners decide to work with the developers and continue building on the invalid stealing block, then they’d be able to trick new users. But they wouldn’t be able to trick existing users, which is generally the vast (economic) majority. Massive drama and probably massive economic losses for these miners would ensue, as no exchange would accept their deposit.

But what if developers, miners, _and_ all existing users conspire to trick new users? Such a conspiracy seems impossible to coordinate secretly. But if you do worry the world is out to get you, rest assured that you can turn the Assume Valid feature off by starting your node with `-assumevalid=0`. Your node would then notice the invalid stealing block, you would yourself see its hash in the source code, and you could run to streets protesting the situation.

What’s important to understand here is that developers can already collude against you and sneak bad things into the code — we’ll talk more about that in chapter @sec:guix. Developers could also put in a backdoor that gives them access to your private keys. This actually happened with an altcoin called Lucky7Coin.^[<https://github.com/alerj78/lucky7coin/issues/1>] These backdoors could be very carefully hidden in the code in a way that only very skilled developers could detect. The Assume Valid hash, on the other hand, is very clearly visible, and it requires very little skill to verify, as explained above. This is why the Bitcoin Core developers believe that this feature is safe against abuse.

Assume Valid has been in Bitcoin Core since v0.14 (2017), and now there’s a new proposal: AssumeUTXO.

### AssumeUTXO

In early 2019, Chaincode Labs alumni James O’Beirne introduced a proposal^[<https://github.com/bitcoin/bitcoin/issues/15605>] for AssumeUTXO that would allow users to get started more quickly. As mentioned earlier in the chapter, the UTXO set is the collection of coins that exists right now. Every time you send someone money, it creates an UTXO, and it destroys the UTXO you sent from. It’s like you have a bank account that’s closed down when you use it, and you get a fresh bank account for the change.

Today, the only way to reconstruct the UTXO set and find out which coins exist right now is to replay all Bitcoin transactions starting from the 2009 genesis block.^[Due to a bug or great benevolence by Satoshi, nodes actually don’t process the genesis block in this manner, so the very first 50 BTC ever created can’t be spent. <https://en.bitcoin.it/wiki/Genesis_block>] You take the first block and see which coins it creates and which coins it destroys. Then you take the second block and do the same. You have to start at the beginning and do it until the end, and you can only do it sequentially — all of which takes a long time.

AssumeUTXO instead uses a recent snapshot of the UTXO set and works from there, skipping hundreds of thousands of historical blocks. It starts out just like nodes today by performing a Headers First sync to determine which chain is the longest one. But once it has the headers, it can load the snapshot. This snapshot is of the UTXO set at a certain height — maybe just before the release. From there, it proceeds as normal, checking each new block to see which coins were destroyed and which were created, until finally, it reaches the most recent block (tip). So then you know your balance exactly and you can start using it.

But in the meantime, in the background, it starts at the genesis block, goes all the way to the snapshot, and verifies that the snapshot is correct. And if the snapshot isn’t correct, it starts screaming (or it unceremoniously crashes with an error message).

With Assume Valid, it still did all of the UTXO set constructing and replayed all of the transactions; it just didn’t check for the signatures. Now, with AssumeUTXO, it skips the transaction replaying altogether, or more accurately: It defers it. Instead, it takes the UTXO set at the snapshot block height, and then processes all subsequent blocks in order to construct the current UTXO set.

### Does the Past Matter?

One question to ask is whether it’s really useful to check historical blocks after loading the snapshot. Of course, if you’re restoring an old wallet from a backup, you’ll need to scan historical blocks for your transaction history, but let’s disregard that scenario for now.

Assuming you’re a new user, the first thing you want to do is receive coins. You want to be sure these coins will later be accepted by others. And you may want to make sure there’s really a 21 million BTC limit. Do you need to check historical blocks to know this?

To start with the second question — checking the 21 million limit — the answer is _no_. You can calculate the total amount of Bitcoin in existence right now by adding up all the values in the UTXO set. And then you can look at the source to understand how many coins can be created in the future. There’s no need to see past blocks for that.

However, to know if others will accept the coins you received, you need to know that the person who sent you the coins didn’t create them out of thin air or steal them. This goes back to the question of if a malicious developer can get away with this. Let’s look into how this compares to manipulating Assume Valid.

Let’s say developers create some coins out of thin air and add them to the UXO set, or that they reassign existing coins to themselves. Anyone verifying the snapshot would find out, so again, code transparency mitigates some of this.

But where, in the Assume Valid example above, the developers would have to create an invalid block right away, before making a new software release, that’s not necessary here. The new or stolen coins would exist in your UTXO set without ever having been in a block. So miners and existing node operators won’t initially detect this, because there’s no invalid block floating around.

But there’s a catch: When you, as the new user, receive a coin that was created out of nowhere, it never gets confirmed in a block. The new transaction won’t be mined, because miners have the correct UTXO set and recognize the transaction as invalid.

What if miners are in on it? Then the transaction would confirm and you’d have been fooled. However, every other node would reject the new block, and the attack would now be visible to everyone involved.

Developers could be very patient though. Instead of immediately trying to spend the from-thin-air coins, they could wait many years. Perhaps by that time, many miners will have reinstalled their node, along with the manipulated snapshot, and synced it. Perhaps many exchanges did so as well. And many regular users. So when they finally spend the from-thin-air coins, perhaps the block is only considered invalid by a small group of old school hardcore bitcoiners.

So as before, this attack requires much of the world to conspire against you, but as far as global conspiracies go, it may be ever so slightly less difficult to get away with.

One way to mitigate this attack is for every block to include a hash of the current UTXO snapshot. This would be a soft fork (see chapter @sec:taproot_activation). That way, every node verifies the snapshot, and it wouldn’t have to be included in the software.

However, as things stand today, producing such a hash would increase the verification time for a block from a few seconds to more than a minute. So a different type of hash has been proposed.^[MuHash: <https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2017-May/014337.html>]

There will probably be a lot more discussion before such a soft fork is even proposed. At the time of writing, AssumeUTXO is still being developed. Nodes can already produce snapshots of their UTXO set, but the code to actually load and use a snapshot is still undergoing review.^[<https://github.com/bitcoin/bitcoin/pull/15606>]
