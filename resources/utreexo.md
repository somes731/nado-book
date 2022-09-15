\newpage
## Utreexo {#sec:utreexo}

\EpisodeQR{15}

Всякий раз, когда выполняется новая биткоин-транзакция, биткоин-узлы используют набор UTXO, чтобы определить, что потраченные монеты действительно существуют (см. главу @sec:assume). Этот набор UTXO в настоящее время имеет размер в несколько гигабайт, продолжает расти с течением времени, и нет верхнего предела того, насколько большим он может стать.

Поскольку биткоин-узлы работают лучше всего, если набор UTXO хранится в оперативной памяти, а оперативная память для компьютеров остается относительно дефицитным ресурсом, для производительности узла было бы полезно, если бы набор UTXO можно было хранить в более компактном формате. Это обещает сделать Utreexo.^[Произносится как U Tree X O. См. также: <https://bitcoinmagazine.com/articles/bitcoins-growing-utxo-problem-and-how-utreexo-can-help-solve-it>]

В рамках Utreexo все существующие UTXO включаются их в дерево Меркла, которое представляет собой структуру данных, состоящую только из хэшей. В этой главе объясняется, как при совершении новой транзакции компактной структуры Utreexo может быть достаточно для доказательства того, что в ней присутствует конкретный UTXO. В главе также рассматриваются потенциальные преимущества, которые могут появиться при использовании этого решения, а также некоторые потенциальные компромиссы.


### Проблема

При синхронизации нового биткоин-узла одной из проблем является объем оперативной памяти (ОЗУ), который у вас есть. В обычной работе вам не нужно иметь на своем компьютере много оперативной памяти, но если вы хотите синхронизироваться быстро, она вам потребуется.

Причина, по которой это необходимо - набор UTXO, который представляет собой список всех существующих монет, включая те, которыми вы владеете. Каждый раз, когда приходит новый блок, для каждой транзакции в нем вы проверяете, тратит ли он реально существующие монеты. Вы также помните, какие новые монеты были созданы транзакцией, чтобы знать об этом, когда они будут потрачены в следующем блоке. Эта информация хранится в базе данных, и если она находится на вашем жестком диске, ее проверка и обновление — медленный процесс. Однако, если база данных находится в вашей оперативной памяти, все работает быстро.

Время, затрачиваемое на это, может быть различным, но, скажем, у вас есть относительно новый MacBook Pro с 32 ГБ ОЗУ, и вы позволили узлу использовать половину этого объема; для синхронизации всей цепи потребуется около семи часов. ^ [В конце 2020 года бывший разработчик Bitcoin Core Йонас Шнелли синхронизировал цепочку на Apple M1 за пять часов. Было бы быстрее, если бы он выделил более 5 ГБ оперативной памяти (используя `-dbcache`). Тем временем сеть выросла, отсюда и более консервативная оценка. <https://twitter.com/_jonasschnelli_/status/1333303029370675201>] Однако, допустим, у вас есть Raspberry Pi только с 2 ГБ ОЗУ и небольшим жестким диском. В этом случае процесс может занять несколько недель.^[Есть два узких места. Во-первых, ваша нода сохраняет как можно больше в оперативной памяти во время синхронизации, но как только она заполняет эту память, она должна записать базу данных UTXO на диск, очистить свою память и снова начать кэширование. Вторая проблема заключается в том, что если блокчейн не помещается на жесткий диск, то вашей ноде приходится удалять старые блоки, чтобы освободить место для новых. Побочным эффектом этого оказывается то, что находящаяся в оперативной памяти база данных монет должна быть записана на диск, что еще больше замедляет процесс синхронизации.]

Ключевым моментом здесь является то, что если вы можете хранить больше UTXO в оперативной памяти, вы будете синхронизироваться быстрее. Было бы неплохо, если бы размер набора UTXO можно было уменьшить, но это не всегда возможно. Конечно, если вы тратите больше монет, чем создаете, то количество UTXO и использование оперативной памяти уменьшаются. Однако в наборе UTXO много мусора, потому что в прошлом люди создавали фальшивые транзакции на адреса с мультиподписью, чтобы, например, поместить фотографии Обамы в блокчейн. И все это хранится в вашей оперативной памяти, потому что ваш узел понятия не имеет, что в них нет смысла.

Однако, если мы ожидаем, что все в мире в конечном итоге будут использовать Биткоин и будут иметь по крайней мере один или два UTXO на каждого, это потребует терабайтов оперативной памяти, и закон Мура^[<https://en.wikipedia.org/wiki/Moore %27s_law>] в ближайшее время не позволит за этим угнаться. Но даже если далеко не весь мир использует Биткоин, все может прийти к тому, что все меньше и меньше людей будет иметь достаточно оперативной памяти для быстрой синхронизации, и это проблема. Если меньше людей управляют собственным узлом, система становится менее децентрализованной.

### Utreexo

Один из способов решения этой проблемы - предложение Таджа Дрийи, Utreexo.^[<https://dci.mit.edu/utreexo>, <https://www.youtube.com/watch?v=6Y6n88DmkjU>] Дрийя — научный сотрудник Digital Currency Initiative Массачусетского технологического института.

Сегодня, имея дело с Биткоином, вы можете сократить архивное хранилище блоков, чтобы сэкономить место на диске. После того, как ваш узел загрузит блок, обработает его и обновит набор UTXO, он больше не будет использовать этот блок. Ваш узел точно знает, какие монеты существуют в наборе UTXO, и это вся информация, необходимая для проверки будущих транзакций.

Когда у узла включен режим pruning (обрезки), он удерживает блок еще несколько дней, а затем удаляет его. Таким образом, вам потребуется менее гигабайта места, хотя сам блокчейн занимает сотни гигабайт. Недостаток состоит в том, что вы не храните блоки, поэтому вы не можете поделиться ими с другими узлами. Это допустимо, если у достаточного количества других узлов есть полный архив.

В рамках Utreexo вы обрезаете и набор UTXO. Вместо того, чтобы выбрасывать транзакции (вместе с блоками, в которых они находятся), вы выбрасываете список существующих монет. Единственное, что вы сохраняете — это корень Меркла, то есть хэш, представляющий все существующие монеты. Каждый лист дерева Меркла представляет собой один UTXO, и корень Меркла связывает их все. Для каждой монеты, которая вам небезразлична, поскольку, например, находится в вашем собственном кошельке, ваш узел хранит доказательство Меркла. Когда вы тратите монету, вам необходимо приложить это доказательство к своей транзакции, чтобы другие узлы могли убедиться, что монета существует.

Иными словами, обычно, когда кто-то отправляет вам транзакцию, транзакция говорит: «Я трачу эти монеты на входе, и вы, человек, управляющий узлом, несете ответственность за проверку существования этих входов в вашей собственной базе данных. ” А теперь вы переворачиваете ситуацию и говорите другому узлу: «Я понятия не имею, какие монеты существуют, потому что у меня недостаточно оперативной памяти, чтобы отслеживать все это. Вы сами должны доказать мне, что эта монета действительно существовала». Таким образом, бремя доказывания перекладывается на отправителя, и осталось только понять, как это сделать.

\newpage
### Краткое введение в доказательство Меркла

![Дерево Меркла. Чтобы доказать существование Монеты 3, вам необходимо предоставить доказательство Меркла, состоящее из трех отмеченных элементов.^[Приводится с изменениями из: <https://commons.wikimedia.org/wiki/File:Hash_Tree.svg>]](resources/tree.svg)

На рисунке выше показано, как вы можете доказать существование Монеты 3, используя доказательство Меркла, при наличии верификатора, который знает только корень Меркла (вверху). Во-первых, вы раскрываете саму монету, которая представляет собой просто выход транзакции с суммой и `scriptPubKey`. Верификатор хеширует полученное и получает Хэш 1-0 (прямо над Монетой 3 на рисунке). Затем вы предоставляете Хэш 1-1. Даже если вы, возможно, не владеете Монетой 4 и можете даже не знать ее количество и `scriptPubKey`, вы все же знаете ее хеш, потому что ваш кошелек отслеживал эту информацию. При этом верификатор может вычислить Хэш 1. Затем вы предоставляете Хэш 0, и теперь верификатор может увидеть, что ваше доказательство приводит к тому же корневому хэшу Меркла, который ему уже известен. Так вы продемонстрировали право собственности на Монету 3 без необходимости для верификатора знать весь набор UTXO.

Мы вернемся к деревьям Меркла в главе @sec:miniscript и главе @sec:taproot_basics.

### Наблюдение леса за деревьями

Currently, if someone sends a transaction to you, you check inside your node and the database with your UTXO set to see if the transaction is spending valid UTXOs. With Utreexo, the sender will have to provide you with the proof that their transaction is spending existing UTXOs. Although you no longer have to hold on to several gigabytes of UTXO data, you do still need to keep track of a few things, namely a Merkle tree — or several trees — of hashes.

В настоящее время, если кто-то отправляет вам транзакцию, вы проверяете свой узел и базу данных с установленным UTXO, чтобы убедиться, что транзакция тратит действительные UTXO. С Utreexo отправитель должен будет предоставить вам доказательство того, что его транзакция тратит существующие UTXO. Хотя вам больше не нужно хранить несколько гигабайт данных UTXO, вам все равно нужно отслеживать несколько вещей, а именно дерево Меркла — или несколько деревьев — хэшей.

All the UTXOs in existence would be put into this tree and everybody can construct this tree if they replay the whole blockchain. Basically what the tree would look like is you have the first and second UTXO next to each other. Then, you take the hash of those two — basically combined — which is one new hash. You can do that again for another two UTXOs that exist and combine their hashes. So, for example, you have four UTXOs. Two of them are shared, and then those two are shared again, and you end up with one hash.

Utreexo uses perfect trees, which means the number of leaves in each must be a power of two. Because the UTXO set contains an arbitrary number of coins, you end up with a forest of trees. For example, if there are six coins, your forest would have a tree with 2^2^=4 leaves and a tree with two leaves. All your node needs to store is the Merkle root hash of each tree. There are currently a little under 100 million coins in existence.^[<https://txstats.com/dashboard/db/utxo-set-repartition-by-output-type>] Because this is less than 2^27^, it’d take 27 trees to represent them all. Each SHA-256 hash^[<https://qvault.io/cryptography/how-sha-2-works-step-by-step-sha-256/>] is 32 bytes, so your node needs to store 27 * 32 = 864 bytes. If every human owns multiple coins, it’d only grow to one kilobyte.

How do we update the tree for every block as leaves are removed and added, i.e. coins are spent and created? You can actually take the UTXO that’s being spent out of the tree and put the new one into the tree. To do that, you need to recalculate the tree, and that’s done by knowing its neighbors. We already illustrated how to prove that something exists in the tree, and it turns out that’s exactly the same information you need to put something else at the bottom of the tree and then provide the root hash.

When you’re syncing the blockchain, you could keep track of the entire tree, but then you’d need a lot of RAM, just like in the original scenario. This is why you only store the root of each tree. Then, when somebody has a new transaction that you want to verify, they need to give you the Merkle proofs for all the inputs they’re spending to prove that they exist. They’ll also tell you which outputs are there — these will be swapped in at the same places where those inputs were — and they’ll tell you about any new trees being made.

### Bridge Nodes

There are two ways you can learn about a transaction. Someone can send it to you via the network, in which case you add it to your mempool. Or, it can be part of a block you receive.

How would you validate the transaction in the first scenario? With Utreexo, you have the top of the trees in your RAM. The sender is responsible for sending the transaction, as well as the proof that the transaction is valid, which also includes information for you so you know where to find it in the forest. If the sender doesn’t support Utreexo and doesn’t provide the proof, your node could simply ignore the transaction.

But what about the second scenario? If a miner mines a block and a transaction is in there, the block doesn’t contain the proof. To get this proof, you need the help of a bridge node. This is a node that has the actual UTXO set, the old-fashioned way, so it has lots of RAM or it’s just slow. And it produces all these proofs and it sends them around to whoever wants them.

When a bridge node receives a transaction that doesn’t have a Merkle proof, it takes the proof it has, attaches it to the transaction, and sends it to other Utreexo nodes. The same goes for entire blocks. You don’t have to be directly connected to such a bridge node, as other Utreexo-aware nodes can relay blocks with the proofs attached. From the perspective of the Utreexo node, the bridge node is just a Utreexo node, and from the perspective of the old-fashioned nodes, it’s just an old-fashioned node.

There’s nothing magical about bridge nodes. Any node that has the original UTXO set can construct the proof for any transaction. But doing so would defeat the purpose of Utreexo, because keeping track of both the regular UTXO set and these proofs takes a lot of memory.

So these bridge nodes do the translation between the current world of nodes that track the UTXO set in memory and these new Utreexo-enabled nodes that don’t have to. As long as one bridge node exists, it can bootstrap the network. However, this relies on these bridge nodes being backed by people with good intentions. But these nodes could change, or disappear, or run out of battery.

Looking at the longterm picture, if people like Utreexo given the advantages — or even if they don’t like it — if the UTXO set gets too big and takes too long to sync on any normal computer, then you could basically make a soft fork that requires all proofs to be in the block.^[You’d include the hash of the proofs somewhere in the coinbase transaction. As we explained in chapter @sec:segwit for SegWit, the proofs themselves would go in a special place inside the block that old nodes don’t see.] By including proofs in the blocks, they’re guaranteed to be available to all nodes.

The tradeoff there is that bigger blocks require more bandwidth and storage, but less RAM is used. At the moment, bandwidth is probably a bigger constraint than RAM, so a soft fork isn’t likely to happen, but this could change in the decades ahead.

### Cool Things

With this solution, because you wouldn’t need a lot of RAM, you could start doing things in specialized hardware. For example, smartphones tend to have very little RAM, so they could get a big performance boost from Utreexo. Or, you could even have a specialized chip — like a GPU — with a tiny onboard memory that validates Bitcoin blocks.^[Then you have the protocol literally set in stone, or at least set in silicon. If somebody wants to do a hard fork, you’d have to break all the node hardware, and not just all the mining hardware. So, that’s a nice extra barrier to not do hard forks. Unfortunately, this also makes soft forks less attractive, as nodes can’t verify the new rules with the accelerated hardware — so your computer would have to slow down to check all the new rules whenever it encounters transactions that fall under the new rules.]

But even without specialized hardware, there’s a potential speedup if the CPU can do most of the block validation work. A 1KB Merkle forest can easily be kept in a typical CPU cache.^[<https://en.wikipedia.org/wiki/CPU_cache>] This avoids having to ferry UTXO set information between the CPU and RAM. Just like using RAM to avoid physical disk reads speeds things up, so does using the CPU cache to avoid using RAM.

In chapter @sec:assume, we described how the source code contains a hash that represents the UTXO set a given snapshot height. The node still needs to obtain that UTXO set, which is several gigabytes in size, and it’d probably download it from its peers. With Utreexo, the UTXO set is so small that it can be put in the source code, thus removing the need to download the UTXO set for the snapshot.

### A Couple Tradeoffs

Although Utreexo has the potential to be cool, there are some tradeoffs. The most apparent is that if you start using it, and then later, somebody finds a better accumulator,^[The general term for what Utreexo uses for its coin accounting is called an accumulator. It’s something you can use to add stuff to, and in this case, to also remove stuff from. But there are all sorts of mathematical tricks you can deploy to do this. The Merkle tree is conceptually very simple, as we hopefully illustrated, but there have been other proposals, like an RSA accumulator. There’s all sorts of cool cryptographic math you can do to just add things to a set and remove them from a set, essentially. It’s too early to set any particular accumulator in stone with a soft fork.] you’d have to switch. Such a switch is easy in a scenario with bridge nodes — multiple solutions could exist in parallel, each with their own bridge nodes. But once the proofs are added into blocks with a soft fork, there’s no easy way back.

Another downside is that bandwidth seems to be the bottleneck for Bitcoin right now, and this could make it worse. For that reason, Utreexo is more of an option that people can opt into if, in their case, bandwidth isn’t a problem. However, if the UTXO set grows to a significant degree where it does become a burden and slows down validation, then this might be more appealing.
