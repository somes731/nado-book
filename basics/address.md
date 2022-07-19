\newpage

## Биткоин-адреса {#sec:address}

\EpisodeQR{28}

Биткоин-адреса это не часть блокчейна Биткоина; скорее, это соглашения, используемые программным обеспечением Биткоина (кошельками) для сообщения, куда должны быть отправлены монеты: открытый ключ (P2PK), хеш открытого ключа (P2PKH), хэш скрипта (P2SH), хэш открытого ключа свидетельства. (P2WPKH) или хеш скрипта-свидетельства (P2WSH). Адреса также содержат некоторые метаданные о самом типе адреса.

Биткоин-адреса сообщают об этих возможностях оплаты, используя свои собственные системы кодирования, и в данной главе будет рассмотрено, что означают эти разные системы. Также будут рассмотрены некоторые преимущества использования биткоин-адресов в целом и адресов bech32 в частности. Также в ней будет описано, как первая версия адресов bech32 содержала (относительно безобидную) ошибку, и как она была исправлена. Глава заканчивается разговором о квантовых технологиях.

### Немного истории

Когда вы отправляете биткоины кому-то, вы создаете транзакцию, которая имеет несколько входов и как минимум один выход. Выход определяет, кто может его потратить, путем наложения на него ограничения (также используется более причудливый термин обременение). ^[<https://www.oreilly.com/library/view/mastering-bitcoin/9781491902639/ch05.html#tx_script>]

Самое банальное обременение - монету может потратить любой желающий. Это не очень хорошая идея, потому что ее очень быстро украдут. Поэтому в первые дни большинство монет в блокчеине были обременены одним из двух способов: оплата по открытому ключу (P2PK) и оплата по хэшу открытого ключа (P2PKH). Первое может быть прочитано как «только владелец закрытого ключа, соответствующего открытому ключу X, может потратить эту монету», а второе как «только владелец закрытого ключа, соответствующего (секретному) открытому ключу, хэш от которого равен X, может потратить эту монету».

В то время можно было отправлять биткойны на IP-адреса получателей, ^[<https://en.bitcoin.it/wiki/IP_transaction>], но в 2012 году от этой функции отказались. Пока это было возможно, вы могли подключиться к чьему-либо IP-адресу и попросить открытый ключ, и человек давал бы вам свой открытый ключ. ^ [Примечание для любопытствующего археолога кода: на узле отправителя интерфейс представлял собой диалоговое окно, в котором запрашивалась сумма и IP-адрес. Функция `StartTransfer()` создала пустую "чековую транзакцию", в которую "выставляющий чек" на узле-получателе вставлял сценарий P2PK (как `scriptPubKey`). Далее функция `OnReply2()` вставляла сумму, подписывала транзакцию, возвращала ее получателю и транслировала в открытый доступ. <https://github.com/bitcoin/bitcoin/blob/v0.1.5/main.cpp#L2004-L2042>] Затем ваш кошелек создавал для отправляемой монеты обременение скриптом P2PK.

Сегодня этот рабочий процесс может показаться странным^ [и небезопасным, как признал Сатоши: <https://bitcointalk.org/index.php?topic=158.msg1322#msg1322>], но он соответствовал распространенному в то время шаблону работы одноранговых приложения, таких как Napster или Kazaa, где вы могли напрямую подключаться к другим людям и загружать от них что-то. В наше время вы, вероятно, уже не знаете IP-адреса своих друзей, к тому же они могут постоянно меняться, если речь идет о мобильных устройствах. Хотя вы можете указать своему узлу Биткойн специально подключаться к дружественному узлу, обычно он просто подключается к случайным узлам (см. главу @sec:dns).

Более распространенный способ выполнения транзакций аналогичен тому, как работают банковские переводы. Кто-то предоставляет вам адрес, и вы отправляете на него монеты так же, как вы отправляете деньги на номер банковского счета. Как будет поясняться ниже, первоначально всегда использовался P2PKH.

Транзакция, вместо того, чтобы быть отправленной непосредственно получателю, проходит через все узлы в сети, чтобы в конечном итоге быть замеченной узлом майнера, который включает ее в блок. Ваш контрагент может увидеть транзакцию, когда его узел получит ее от одного из своих пиров, или как только он получит блок, в котором она находится.

Третий способ совершения транзакций — это майнинг биткоинов, который включает в себя отправку вознаграждения за блок самому себе. Вначале в Биткоин была встроена часть программного обеспечения для майнинга, поэтому, если вы загружали софт для Биткоина, он просто сразу начинал поиск блоков. Затем он отправлял найденные монеты на ваш собственный кошелек, поэтому не было необходимости сообщать адрес. Эти монеты были обременены посредством P2PK. ^ [Почему первоначальная версия Сатоши поддерживала как P2PK, так и P2PKH? Мы не знаем. Способ оплаты P2PK использовался только для оплаты по IP-адресу, а для майнера — при выплате вознаграждения за блок. Ни то, ни другое не нуждалось в человеческом взаимодействии. В сценариях, которые включали взаимодействие с человеком, использовался P2PKH. Поэтому, когда речь идет об адресе, подразумевается P2PKH, а не P2PK. Автоматизированным системам не требуется понятие адреса, поскольку они могут с таким же успехом обрабатывать скрипты, поэтому такой вещи, как адрес P2PK, не существует.]

### Так что же содержится в адресе?

Адрес — это удобный способ сообщить, какой скрипт должен быть включен в блокчейн. Как было указано выше, цель этого скрипта — ограничить монету так, чтобы только получатель мог ее потратить. ^[Пока аналогия с банковскими счетами верна, но в главе @sec:miniscript мы узнаем, что сценарии могут быть гораздо более мощными, чем просто емкости для хранения денег своих владельцев.] Сам адрес отсутствует в блокчейне. Там нет даже полного скрипта.

Из двух основных типов скриптов, использовавшихся в то время, адреса использовались только для оплаты по хэшу открытого ключа (P2PKH). Когда кошелек видит такой адрес, он создает скрипт для блокчейна Биткоина, который требует, чтобы человек, который его тратит, имел открытый ключ, соответствующий хэшу (в главе @sec:miniscript приводится текущий скрипт). Публикуется только хэш, поэтому открытый ключ остается секретным, пока получатель не потратит монеты.

Адрес начинается с цифры 1, за которой следует хэш открытого ключа. Он закодирован с использованием некой сущности, называемой base58. Вот пример:
`1HLoFgMiDL3hvACAfbkDUjcP9r9veUcqAF`

### Основания систем счисления

Чтобы понять, что такое base58, важно сначала узнать больше об основаниях систем счисления в целом.

Base10 легко объяснить на примере вашей собственной руки. У вас 10 пальцев. Таким образом, если вы хотите, например, выразить число 115 (1, 1, 5), вы можете сделать три жеста руками, показывая 1, 1 и 5. Аналогичным образом вы записываете и числа, которые — начиная с изобретения глиняных табличек и бумаги — удобнее, чем счет на пальцах. Таким образом, base10 — это десятичная система, которая использует 10 различных символов в различных комбинациях для представления любого числа (целого числа).

Однако существовали, да и сейчас существуют, иные системы счисления. Например, вавилоняне^[<https://blogs.scientificamerican.com/roots-of-unity/ancient-babylonian-number-system-had-no-zero/>] использовали основание 60. А для чтения машинного кода обычно используется шестнадцатеричный код^[<https://en.wikipedia.org/wiki/Hexadecimal>], который имеет основание 16 — от 0 до 9, а затем от A до F. Между тем, сами компьютеры, как правило, используют base2 — двоичную систему счисления — потому что транзисторы либо включены, либо выключены. Это означает, что все требуется делать с использованием двух цифр, либо 0, либо 1, и таким образом можно выразить любое число.

Сатоши предложил систему ^[<https://tools.ietf.org/id/draft-msporny-base58-01.html>] base58, в которой используется 58 различных символов: от 0 до 9, а затем большая часть алфавита как в нижнем, так и в верхнем регистре. Но есть некоторые буквы и цифры, которые пропускаются, потому что они неоднозначны, и пользователи могут перепутать их друг с другом — например, цифра 0 и заглавная буква O, а также заглавная I и строчная l.

Вы когда-нибудь видели исходный код вложенного в емэйл файла или чего-то подобного? Там много странных символов. Это base64, а base58 основана на ней. Но base64 включает в себя такие символы, как подчеркивание, плюс, равенство и косая черта. В base58 они опущены, для упрощения внешнего вида и корректности использования в качестве части URL.

### Base58 и оплата по хэшу открытого ключа

So how does this relate to P2PKH? Well, the address is expressed as a 1, followed by the public key hash, which is expressed in base58.

Итак, как это связано с P2PKH? Все просто: адрес состоит из цифры 1, за которой следует хэш открытого ключа, записанный в системе base58.

That’s the information you send to somebody else when you want them to send you bitcoin. You could also just send them 0x00,^[A pair of hexadecimal digits, prefixed by 0x, is often used to denote bytes, which contain 16 × 16 = 256 bits, so this represents one byte with the value 0.] and then the public key. And maybe they’d be able to interpret that, but probably not.

In theory, you could send somebody the Bitcoin script in hexadecimal, which is the format used on the blockchain, because that’s just binary information. The blockchain has this script that says, “If the person has the right public key hash and the public key belonging to this public key hash, then you can spend it.” To learn more about how Bitcoin scripts work, refer to chapter @sec:miniscript.

But even with all these options, the convention is that you use this standardized address format, which explains why all traditional Bitcoin addresses start with a 1, and why they’re all roughly the same length.

In addition to using base58 for sending a Bitcoin address, you can also use it to communicate a private key. In such a scenario, the leading symbol is a 5, which represents 128. That’s then followed by the private key.

In the past, users had paper wallets they could print. And if they were generated securely without a back door, then on one side of the piece of paper would be something starting with a 1, and on the other side of the paper would be something starting with a 5. And then it specified that only the Bitcoin address should be shown, but the private key shouldn’t be shared.

There are also addresses that begin with a 3, which is for coins encumbered by the hash of a script, rather than the hash of a public key. We’ll cover Pay-to-Script-Hash (P2SH) in chapter @sec:miniscript. Usually these are multi-signature addresses, but they could also be SegWit addresses.^[As explained in chapter @sec:segwit, SegWit typically uses bech32 addresses. But it took a long time for all wallets and exchanges to support sending to bech32 addresses. To still take advantage of some of SegWit’s benefits, an address type that looks like regular P2SH to the sender was introduced, but it contains SegWit magic under the hood. This is called a P2SH-P2WPKH address: <https://bitcoincore.org/en/segwit_wallet_dev/>]

Although base58 addresses worked fine, there was room for improvement. And this came in the form of bech32.

### Along Came Bech32

In March of 2017, Pieter Wuille spoke about a new address format,^[<https://www.youtube.com/watch?v=NqiN9VFE4CU>] bech32, and it’s been used since SegWit arrived on the scene. As the name suggests, it’s a base32 system, which means you have almost all the letters, and almost all the numbers, minus some ambiguous characters that you don’t want to have because they look too much like other numbers or letters.

One of the biggest differences between bech32 and base58 is that there isn’t a mixture of uppercase and lowercase letters. Instead, each letter is only in there once — either in all uppercase or all lowercase — which makes reading things out loud much easier. The precise mapping of which letter or number corresponds to which value is, like in base58, fixed but arbitrary: The fact that P means 0 and Q means 1 has no deeper meaning.

     0  1  2  3  4  5  6  7
--- -- -- -- -- -- -- -- --
+0   q  p  z  r  y	9  x  8
+8   g	f  2  t	v	d  w  0
+16  s	3  j  n	5	4  k  h
+24  c	e  6  m	u	a  7  l

Table: Bech32 mapping. E.g. `q` means zero, `3` means 17 (1 + 16)

A bech32^[Bech32 spec (BIP 173): <https://en.bitcoin.it/wiki/BIP_0173>] address consists of two parts separated by 1, e.g.
`bc1q9kdcd08adkhg35r4g6nwu8ae4nkmsgp9vy00gf`.

<!-- The phrasing "e.g." ensures that the address doesn't exceed the right page margin -->

The first part is intentionally human readable, e.g. “bc” (Bitcoin) or “lnbc” (the Lightning network on Bitcoin). The values represented by “b,” “c,” etc. have no meaning. Rather, they’re there so humans can recognize, “OK, if the address starts with bc, then it refers to Bitcoin as the currency.” However, wallets will look for the presence of these values as a confidence check, and it’s included in the checksum.

The 1 is just a separator with no value. And if you look at the 32 numbers, 1 isn’t included — it means “skip this.”

The second part starts with the SegWit version number. Version 0 is represented with Q (bc1q…) — see chapter @sec:segwit. Version 1 is what we call Taproot (see part @sec:taproot), as it’s represented with “P” (bc1p…). For version 0 SegWit, the version number is followed by either 20 bytes or 32 bytes, which means it’s either the public key hash or the script hash, respectively. And they’re different lengths now, because SegWit uses the SHA-256 hash (32 bytes) of the script, rather than the RIPEMD160 hash (20 bytes) of the script.

In base58, the script hash is the same length as the public key hash. But in SegWit, they’re not the same length. So by looking at how long the address is, you immediately know whether you’re paying to a script or you’re paying to a public key hash. As an aside, Taproot removes this length distinction, thereby slightly improving privacy.

So the new part is that there’s a set of 32 characters, but otherwise, things are very similar to base58. It’s again saying, “OK, here’s a P2PK address.” In this case, it’s a Pay-to-Witness-Public-Key-Hash (P2WPKH), where witness refers to SegWit, but it’s the same idea. There’s a short prefix that tells both humans and the computer what the address is about, and this is followed by the hash of the public key or script.

### Thirty-Two Dimensional Darts

However, conciseness isn’t the only benefit here. Another is error correction, or at least detection.

If there’s a typo in an address, then in the worst case scenario, you’re sending coins to the wrong hash of a public key. When the recipient tries to spend the coin, they reveal the public key, but due to the typo, its hash won’t match what the blockchain demands. The coins are forever lost.

Fortunately, base58 addresses contain a checksum at the end. That way, if you make a typo, the checksum at the end of the address won’t work. Your wallet will alert you to this, and it’ll refuse to send the transaction (the blockchain won’t protect you; only your wallet will, hopefully). But if you’re really unlucky, a typo can be such that it produces a correct checksum by sheer coincidence.

Bech32 was designed in such a way to make such a disastrous coincidence extremely unlikely. In addition, it won’t just tell you that there _is_ a typo; it can tell you _where_ the typo is. This is determined by taking all the bytes from the address and then hashing it using some sophisticated mathematical magic.^[Math behind bech32 addresses: <https://medium.com/@MeshCollider/some-of-the-math-behind-bech32-addresses-cf03c7496285>] You can make about four typos and it’ll still know where the typo is and what the real value is. If you do more than that, it won’t.

To illustrate this conceptually, it’s like if you have a wall and you draw a bunch of non-overlapping circles on it. The bullseye of each circle represents a correct value, whereas any other spot within the circle represents a typo. If you’re a good dart player, most of the time you’ll hit the bullseye, i.e. you typed the correct value. If you slightly miss the bullseye but you’re still within that big circle, the value will be slightly incorrect. Error _detection_ is knowing that you missed the bullseye. Error _correction_ is the equivalent of moving the dart to the nearest bullseye.

The idea there is you want the circles to be as big as possible, to facilitate even the sloppiest dart thrower, but you don’t want to waste too much space. Similarly, we don’t want Bitcoin addresses to be hundreds of characters long. That’s the kind of optimization problem mathematicians love.

In the case of bech32, instead of a two-dimensional wall, you have to somehow imagine a 32-dimensional “wall” with 32-dimensional hyperspheres. You’re hitting your keyboard, and somewhere in that 32-dimensional space, you’re slightly off, but you’re still inside this hypersphere, whatever that might look like. In that case, your wallet knows where the mistake is, and it prevents you from sending coins into the ether.^[Pardon the pun, but early Ethereum wallets didn’t use error detection, because their address standard lacked a checksum. Although EIP 55 introduced such a checksum in 2016, not all wallets enforced it. Even in late 2017, people lost coins due to typos: <https://bitcointalk.org/index.php?topic=2161699.0>]

### But… There’s a Problem

In 2019 it was discovered that, if a bech32 address ends with a P, then if you accidentally add one or more Qs to it, it still will match the checksum, and you won’t be told there’s a typo. In turn, your software would think it’s correct, you’d be sending money to the wrong address, and it would be unspendable, as we explained above.

The good news is that bech32 was only used for SegWit, and SegWit addresses were constrained — they had to be either 20 bytes or 32 bytes. So luckily, if you add another Q to a 20- or 32-byte address, then its length would be invalid. Your wallet would detect this and refuse to send coins. A similar size constraint was considered for Taproot, but thanks to the solution below, it wasn’t needed. A flexible length makes future improvements to Taproot easier.

### Enter Bech32m

<!-- Colon intentionally omitted so footnote fits on one line -->
To fix this bug, a new standard called bech32m was proposed.^[Bech32m spec (BIP 350) <https://en.bitcoin.it/wiki/BIP_0350>] It’s actually a very simple change. It adds one extra number to the bech32 checksum math, which then makes sure no extra characters can be added without causing an invalid checksum.

The new standard is only used for Taproot and future addresses. For SegWit, nothing changes, because it’s already protected by the 20- or 32-byte length constraint. At the time of writing, most wallet software supports the new bech32m standard.

### How I Learned to Stop Worrying and Love Quantum

As an aside, Pay-to-Public-Key-Hash (P2PKH) was thought to be safer against quantum attacks, because you didn’t have to say which public key you had. The downside was that the hash consumed more block space — but this wasn’t an issue back then, because blocks were nowhere near full.

Many people are worried that quantum computers will eventually break the security offered by Bitcoin’s cryptography, allowing future quantum hackers to steal coins, potentially crashing the market if they steal millions.

The problem is that, despite widespread P2PKH use, there’s 5 to 10 million BTC out there for which the public key is already known. The irony is that because so much BTC is already vulnerable to quantum theft, there’s no use trying to protect the rest. Even if your coins won’t be stolen, they’ll be worthless from the price crash.

The (un)likeliness of such quantum troubles in the near future, as well as possible countermeasures, is explained in two _What Bitcoin Did_ podcast episodes — with physicist Stepan Snigirev^[<https://www.whatbitcoindid.com/podcast/the-quantum-threat-to-bitcoin-with-quantum-physicist-dr-stepan-snigirev>] and mathematician Andrew Poelstra.^[<https://www.whatbitcoindid.com/podcast/andrew-poelstra-on-schnorr-taproot-graft-root-coming-to-bitcoin>]

Block space is much scarcer now, so not having to put public key hashes on precious block space would save users fees. This is why in the new Taproot soft fork (see part @sec:taproot), Bitcoin addresses are P2PK again.[^rationale][^tweet] Note that the use of Taproot addresses isn’t mandatory, so if you don’t agree with the above reasoning, you can simply choose to not use Taproot.

[^rationale]: Full rationale in BIP 341: <https://github.com/bitcoin/bips/blob/master/bip-0341.mediawiki#cite_note-2>
[^tweet]: <https://twitter.com/pwuille/status/1409560741489778688>
