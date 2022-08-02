\newpage
## SegWit {#sec:segwit}

\EpisodeQR{32}

Segregated Witness (разделённое свидетельство), также известный как SegWit - это софт-форк, активированный в сети Биткоина летом 2017 года. Это был последний софт-форк перед активацией Taproot осенью 2021 года, и, возможно, это все еще крупнейшее обновление протокола Биткоина на сегодняшний день.

Вкратце, SegWit позволил разделить данные о транзакциях и о подписях в блоках Биткоина. В этой главе объясняется, как это работает, и подробно рассказывается о том, что это даёт.

### Зачем разделять свидетельства?

До появления SegWit существовала проблема с пластичностью транзакций. Каждая транзакция имеет уникальный идентификатор. Когда кто-то отправляет вам монеты, а вы отправляете их кому-то еще, ваша транзакция (B) использует идентификатор (ID) транзакции (A) для ссылки на нее. Теперь, если обе транзакции не подтверждены, может возникнуть проблема, когда злоумышленник получит доступ к транзакции А и поменяет ее. Эта манипуляция изменяет идентификатор A. В результате транзакция B теперь использует устаревший идентификатор для ссылки на транзакцию A, что означает, что она ссылается на пустоту. Транзакция, которая пытается потратить из пустоты, недействительна и никогда не попадет в блок. В лучшем случае это создает серьезное неудобство.

Хорошо известный пример пластичности транзакций - то, что произошло с Mt.Gox, биткоин-биржей из Японии. [Чтобы получить более глубокое представление о кончине Mt.Gox, послушайте <https://www.whatbitcoindid.com/mtgox-interviews>] Согласно некоторым источникам, Mt.Gox вела свой внутренний учет на основе идентификаторов транзакций. Клиент снимал средства, использовал пластичность, чтобы немного изменить транзакцию снятия и получить деньги, потому что транзакция все еще оставалась действительной, но затем заявлял: «Я вывел деньги, но так и не получил их». В ответ Mt.Gox использовала идентификатор транзакции и смотрела, есть ли она в блокчейне. Видя, что в блокчейне нет совпадающего идентификатора транзакции, биржа считала, что клиент прав, и повторно отправляла монеты. ^[<https://en.wikipedia.org/wiki/Mt._Gox#Withdrawals_halted;_trading_suspended;_bitcoin_missing_(2014)>]

Если более точно, то манипулированию поддавалась та часть транзакции, где содержалась подпись: каждая транзакция подписывается криптографической подписью. До SegWit существовало множество способов настроить эту подпись так, чтобы она выглядела по-другому, но оставалась действительной, с неизменной суммой и получателем. Менялся только идентификатор транзакции.

Один из способов заключался в том, чтобы поставить минус перед подписью — помните, что подпись это просто большое число — и любой мог это сделать. ^ [Это было исправлено с помощью BIP 66: <https://en.bitcoin.it/wiki/BIP_0066> ] Когда вы транслируете транзакцию, она переходит с вашего узла на другой и двигается далее. Вышеупомянутый злоумышленник увидит, что транзакция появится на его узле, перехватит ее, поменяет бит, отвечающий за знак числа, и отправит дальше. В этот момент по сети распространяются две версии одной и той же транзакции, и только одна из них попадет в следующий блок.

Можно, однако, спросить себя: «А в чем проблема-то?» Проблема, однако, может случится не на только что описанном этапе, когда вы получаете монеты и отправляете их дальше. Если в блок попадает измененная версия транзакции A, вы просто создаете новую транзакцию (B), которая ссылается на новый идентификатор транзакции A.

Но представьте, что вы отправили транзакцию (А) в сверхзащищенное хранилище в Арктике, расположенное в тысячах метров под землей. А затем вы отправились в Арктику и создали транзакцию возврата (B) обратно в свой горячий кошелек, подписали ее, но еще не транслировали. Затем, как только вы транслируете первую транзакцию (A) для отправки денег в хранилище, и кто-то ее подпортит, внезапно вторая транзакция (B) становится недействительной, поскольку она относится к неизмененной (A). Теперь вам нужно вернуться в Арктику, чтобы создать новую транзакцию (B), которая ссылается на измененную версию (A) — и этот сценарий в лучшем случае сильно осложнит вам жизнь.^^ [Этот пример может показаться надуманным, но при проектировании хранилищ необходимо учитывать фактор пластичности транзакций. <https://bitcoinops.org/en/topics/vaults/>]

Другой, более приближенный к практике пример того, как пластичность становится проблемой, связан с сетью Lightning^[Lightning не описывается в этой книге, но можно глянуть приложение @sec:more_eps.], где вы создаете неподтвержденные транзакции на друг над другом. Таким образом, если одна из базовых транзакций изменена, последующие транзакции становятся недействительными.

В рамках протокола Lightning два человека отправляют деньги на общий адрес, и единственный способ получить деньги с этого адреса — использовать специальную транзакцию, которую обе стороны подписали _до того_, как они отправили деньги на общий адрес. Вы не хотите, чтобы кто-то подправлял транзакцию, которая идет на этот адрес, потому что тогда вы больше не сможете потратить эти деньги — или, пожалуй, сможете, но вы оба должны подписать ее заново. Это потенциально дает одной стороне возможность шантажировать другую под угрозой не дать вернуть свою справедливую долю монет.

### Solving Transaction Malleability

So it’s easy to see how much of an issue this was.

A transaction consists of all the transaction data and the signature. It’s identified by the transaction ID, which, before SegWit, was the hash of those two things. For example, the 10 BTC transaction from Satoshi to Hal Finney is f4184fc5…^[f4184fc596403b9d638783cf57adfe4c75c605f6356fbc91338530e9831e9e16 <https://bitcointalk.org/index.php?topic=155054.0>]

However, because the signature can be tweaked, the hash (transaction ID) can also be tweaked, and you end up with basically the same transaction but with a different transaction ID. That’s the problem that needed to be solved: somehow either making sure the signature can’t be modified, or that such a modification won’t change the transaction hash. The first approach appears to be very difficult if not impossible,^[<https://en.bitcoin.it/wiki/BIP_0062>] so SegWit involves the second approach.

The solution was to append the signature to the end of a transaction. This new transaction part isn’t included when calculating the identifier hash. It’s also not given to old nodes. As far as old nodes are concerned, the signature is empty and anyone can spend the transaction.^[
To be more precise: The `scriptSig` is empty, where before it would’ve put a public key and signature on the stack. In turn, the `scriptPubKey` is a `0` followed by a public key hash. To old nodes, this combination results in a non-zero item on the stack, i.e. `True`, which is a valid spend. On the other hand, SegWit-enabled nodes will interpret the `scriptPubKey` as a SegWit v0 program and use the new `witness` field when evaluating. See <https://en.bitcoin.it/wiki/BIP_0141>
] Because the new signature part isn’t included in the transaction hash, its identifier doesn’t change when the signature changes. Both new nodes, which have the signature, and old nodes which don’t, can calculate the transaction ID and it’s identical for both.

In short, SegWit solved the transaction malleability issue, where transaction IDs could be altered without invalidating the transactions themselves. In turn, solving the transaction malleability issue enabled second-layer protocols like the Lightning network.

### SegWit as a Soft Fork

How could SegWit be deployed as a soft fork (backward-compatible upgrade)? We’ll dive more deeply into how soft forks work in chapter @sec:taproot_activation, but the basic idea is that upgraded nodes are aware of the new rules, while un-upgraded nodes don’t perceive a violation of the rules.

With SegWit this is achieved by appending data to the end of a block, kind of like a subblock, and not sending that data to legacy nodes. A hash of this data is added to the coinbase transaction^[The company Coinbase was named after this first transaction in a block, which creates coins out of nowhere and pays the miner their reward.] in an `OP_RETURN` statement.

An `OP_RETURN` typically signifies that transaction verification is done, but it can be followed by text, which is then ignored. So old nodes just see an `OP_RETURN` statement, they don’t care what follows, and they won’t ask for the additional data. New nodes make an exception to this rule when it comes to the coinbase transaction, they do check the hash. A SegWit node verifies that the witness data is correct by comparing it to this hash. It also expects this extra data to be present, and will request it from other SegWit aware nodes if necessary.

### Block Size Limit

Before SegWit, blocks had a one-megabyte limit, and that limit included the transaction data, plus all the signatures, plus a little bit of block header data. Today, because SegWit transactions put their signature data in a separate place that old nodes won’t see, blocks can be larger. Theoretically, they can be up to four megabytes, but in practice with typical transactions, it’s closer to two and a half.

Because the signature (witness) data is in a place that old nodes don’t see, we can bypass the one-megabyte block size limit without a hard fork. Old nodes will keep seeing a block with no more than one megabyte in it, but new nodes are aware of the witness data, which takes the total size well over one megabyte.

However the increase isn’t unlimited. SegWit nodes use a new way of calculating how data is counted, which gives a 75 percent discount to this segregated signature data. The percentage is somewhat arbitrary — enough to make SegWit transactions cheaper than their pre-SegWit counterparts, but not so much to incentivize abuse.

### Future SegWit Versions, e.g. Taproot

The topic of Taproot is covered in depth in chapter @sec:taproot_basics. But what’s important to know here is SegWit’s script versioning allows for easier upgrades to new transaction types, and the recent Taproot upgrade is the first example of this feature.

The versioning works as follows, and is also touched on in chapter @sec:address, which covers addresses. The output of each transaction contains the amount and something called the `scriptPubKey`. The latter is a piece of Bitcoin script that constrains how to spend this coin, as we briefly mentioned in chapter @sec:address and will explain in more detail in chapter @sec:miniscript. With SegWit, the `scriptPubKey` always starts with a number, which is interpreted as the SegWit version. The rules for interpreting SegWit version 0 are set in stone, as are those for interpreting SegWit version 1, aka Taproot. But anything following a 2 or higher is up for grabs: Those rules may be written later.

Before a new soft fork activates, anything following an unknown version number is ignored, thus it’s anyone-can-spend. As we’ll explain in chapter @sec:taproot_activation one of the things that could go wrong with soft fork activation is that a majority of miners aren’t actually enforcing the new rules. But as long as most miners do enforce the new rules, they’ll ensure that these anyone-can-spend outputs, from the perspective of old nodes, won’t actually get spent.

Miners that run updated node software consider blocks that spend these coins invalid. And as long as they’re in the majority, they’ll also create the longest chain. So now the new nodes are happy because all the new rules are being followed, and the old nodes are happy because no rules are being broken from their perspective and they just follow the longest chain. So the network stays in consensus.

### Hardware Wallets

In addition to all the aforementioned benefits — fixing malleability, increasing block size, versioning, etc. — SegWit introduces a commitment to the inputs, which primarily benefits hardware wallets.

A hardware wallet is an external device that holds your private keys and can sign Bitcoin transactions. Because the device is purpose built and otherwise very simple, it’s less likely than your regular computer to have malware on it. It usually shows you a summary, based on its understanding of a transaction, and then asks you to approve the transaction before it actually signs.

Before signing a transaction, the device shows you the destination address and amount. That way you can verify that an attacker didn’t swap out the address for one they control.

The device also checks that the input amounts add up to the output plus the fee. This protects you against a scenario where an attacker makes you pay an absurd amount of fees (perhaps colluding with a miner).

However, transactions don’t actually specify their input amounts. The only way for the device to learn those is if you give it the input transactions. It can then inspect their output amounts. But having to send all the input transactions to the hardware wallet can be problematic, especially when they’re big, because these devices tend to be slow and have very limited memory resources.

To be clear, any wallet should perform these checks — not just hardware wallets. You always have inputs, which are coins you own. And then you have the outputs, which are coins you’re sending, including a change output to yourself usually. The difference between them is the fee the miner keeps, and the fee isn’t mentioned in the transaction, so the wallet calculates it for you.

This works for a regular wallet, because it knows how much all of the inputs are worth. But a hardware wallet is disconnected from the internet, so it doesn’t necessarily know how much all the inputs are worth. Without that information, it can’t be sure how much money it’s about to send.

Therefore, a hardware wallet has the risk that it’s sending 10 million coins as a fee without realizing it. And if somebody colludes with the miner or just wants to take your coins hostage in some weird way, that’s not good. So what SegWit does is it commits to those inputs.^[Unfortunately, the approach used by SegWit still left some potential attacks open, but these have been addressed by Taproot.]

What SegWit adds to this is that, before creating a signature, the output amount is added to the data that’s signed. The device now receives these amounts, along with the transaction it needs to sign. It uses that to inform the user and to create the signature. If your computer lied to the device about the amount, then the resulting signature is invalid. So the device no longer needs to look at the actual previous transaction.

Note that nothing is stopping your computer from crafting a fake transaction with fake inputs and any output amount it wants — the hardware wallet will happily sign it. But when your computer then broadcasts it to the network to get it included in the blockchain, it’s just not going to be valid. So it’s pointless for an attacker to try this.

### Recap

The main benefit of SegWit is that it fixes malleability, which enables things like the Lightning network, resulting in a pretty big increase in potential transaction throughput.

The second benefit is an increase in block size, even though this is dwarfed by the capacity increase Lightning could achieve. The third is versioning, which makes it easier to deploy future upgrades. And the fourth is improved hardware wallet support.

### Block Size War

If the above sounds great and uncontroversial, it’s because, in my opinion, it is. There was, however, a lot of drama in the years surrounding this soft fork. One account of this is provided in Jonathan Bier’s _The Blocksize War: The battle over who controls Bitcoin’s protocol rules_.^[<https://www.amazon.com/Blocksize-War-controls-Bitcoins-protocol/dp/B08YQMC2WM>]
