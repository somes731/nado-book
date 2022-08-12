\newpage
## libsecp256k1 (Программные библиотеки) {short="libsecp256k1" link="sec:libsecp"}

\EpisodeQR{9}

libsecp256k1^[<https://github.com/bitcoin-core/secp256k1>] — это библиотека, о которой некоторые люди, возможно, слышали мимоходом, но многие на самом деле толком не вникают в нее и не понимают ее важности. В этой главе мы обсудим, что это такое и почему это важно для Биткоина. Но прежде чем заняться этим, было бы полезно сделать обзор библиотек в целом.

Библиотека — это многократно используемая часть программного обеспечения. Согласно Techopedia, ^ [<https://www.techopedia.com/definition/3828/software-library>] «Программная библиотека — это набор данных и программного кода, который используется для разработки программ и приложений». Примером этого в мире криптографии является OpenSSL: это часть программного обеспечения, которая позволяет вам выполнять различные криптографические операции — от создания случайных чисел до подписи чего угодно при помощи каких угодно кривых. Библиотека сама по себе не является программой, поскольку она ничего не делает независимо. Однако другие программы могут использовать такую библиотеку, как OpenSSL, или ее подмножество, для выполнения желаемых действий без необходимости самостоятельно писать весь код.

В случае OpenSSL пользователи загружают Bitcoin Core, самое популярное программное обеспечение, используемое для подключения к сети Биткоина. Его двоичный файл содержит элементы, специфичные для Bitcoin Core, а также множество соответствующих библиотек. Одной из таких библиотек является OpenSSL. Или, скорее, _была_, как мы объясним позже.

С самого начала OpenSSL использовалась в Биткоине для всего, что связано с криптографией, например, для подписи транзакций и создания безопасных случайных закрытых ключей. Сатоши пришлось выбрать одну из криптографических кривых, предлагаемых библиотекой. По разным причинам, о которых мы можем только догадываться, ^ [см. раздел «Выбор правильной эллиптической кривой» в этой статье Виталика Бутерина (написанной до создания им Эфириума): <https://bitcoinmagazine.com/technical/satoshis-genius-unexpected-ways-in-which-bitcoin-dodged-some-cryptographic-bullet-1382996984>] он выбрал кривую secp256k1. В результате ему не пришлось самому писать необходимый криптографический функционал — чего никогда не хочется делать самому, потому что это опасно.^[<https://security.stackexchange.com/a/18198/209204>] Соответственно, Сатоши не выбрал более совершенную схему подписи Шнорра — эта тема будет рассмотрена в главе @sec:schnorr — потому что OpenSSL не поддерживал ее, и для нее не существовало другой библиотеки.

Необходимая библиотека включается в загружаемый файл каждой новой версии Bitcoin Core. Не все программы включают в себя все свои библиотеки. Альтернативой является использование библиотек, которые уже присутствуют на вашем компьютере, что уменьшает объем загрузки, поскольку библиотеку не нужно загружать повторно. Однако это может создать проблемы.

Наиболее очевидная проблема не включения всех библиотек в загрузку заключается в том, что у пользователя может не быть одной или нескольких требуемых библиотек. Затем им нужно будет дать указание загрузить их, а это плохой пользовательский опыт. С другой стороны, на самом деле это обычный опыт в жизни разработчиков программного обеспечения, которые тратят большую часть своего времени на поиск библиотек и других компонентов для профессиональных инструментов, которые они пытаются установить. Опыт часто является рекурсивным, когда каждая библиотека, в свою очередь, зависит от еще одной библиотеки.

Другая проблема заключается в том, что библиотеки могут меняться, и в самлм деле меняются. Библиотека в вашей системе может быть слишком старой или слишком новой. Возможно, изменились важные вещи, из-за которых библиотека больше не будет совместима с программой, которая на нее опирается.^[Изменения в библиотеках с течением времени затрудняют компиляцию и запуск старых версий Bitcoin Core, даже если исходный код доступен. Это будет проблемой для будущих археологов кода. <https://blog.lopp.net/running-bitcoin-core-v0-7-and-earlier/>] Последнее, что вам нужно, имея дело с криптографическими вещами — это получать сюрпризы от того, что находится на вашем компьютере.

Including libraries in the download means you always have the right version. This is why your computer probably contains many copies and versions of the same libraries.

But even when libraries are included in a download, things can go wrong when a software developer decides to update a library to a newer version.

Someone out there is maintaining the library. They don’t have time to test each of their changes in every single software package out there that uses their library. So if you’re not paying attention to what the library maintainer is doing — either by looking at changes in the release notes or by checking out the code itself — they might break something.

Then, when you download the library along with the rest of Bitcoin Core, your computer now uses that changed part of the library. But what if the Bitcoin Core developers didn’t notice this particular change that happened to the library? Then, all of a sudden, the stuff they wanted Bitcoin Core to do isn’t actually happening.

Most breaking changes in libraries are accidents, but not all. Chapter @sec:guix goes deeper into the process of checking dependencies and attacks from rogue dependency maintainers.

A breaking change in how a library behaves is particularly problematic when it causes a change in the interpretation of the rules of the blockchain: Bitcoin Core would consider a particular block valid in one version of the library, and in another version, it’d consider that same block invalid. This leads to a chain split.^[<https://coinmarketcap.com/alexandria/glossary/chain-split>]

This is exactly what happened with a past version of Bitcoin Core: There was a bug in OpenSSL, which meant the developers of Bitcoin Core had to upgrade OpenSSL because the old version was simply no longer safe. But unbeknownst to the Core developers, there was another change in OpenSSL when they upgraded.

This particular change dealt with what happened with signatures and whether or not they’re considered valid. The original version of OpenSSL was pretty relaxed, so it would accept signatures as valid even if they didn’t meet the exact specifications. They couldn’t be signed by somebody else, so it wasn’t about stealing funds, but it was more about the fact that the notation could be a bit sloppy.

Now, the new version was extremely picky. If you used Bitcoin software to create a transaction, that wasn’t a problem, because any Bitcoin transaction was signed strictly, according to the protocol. And if you decided to validate these transactions using old software, it would see the sloppy version that was also made with the old software, and it wouldn’t have an issue with the transaction. However, the new software would say it’s invalid, because it doesn’t accept these sloppy signatures. So all of a sudden, there’s an accidental soft fork, which is what happens when previously valid transactions become invalid.

Fortunately, some developers became aware of this problem in time. Had they not noticed this, there could’ve been a big chain split once the updated OpenSSL library was bundled in a Bitcoin Core release binary. Instead, several measures were taken to defuse this timebomb.

First, users could still download the Bitcoin Core binary as usual, since it came bundled with this older version of OpenSSL. Even though this version had a security bug in it, remember that each piece of software on your computer can choose to bring its own version of OpenSSL. This particular bug^[<https://nvd.nist.gov/vuln/detail/CVE-2014-8275>] impacted browser software, but not Bitcoin Core, so your browser would have to ship the new version immediately, while Bitcoin Core could wait a little longer.

Second, those who prefer to compile their own software from source were offered two possible workarounds.^[<https://lists.linuxfoundation.org/pipermail/bitcoin-dev/2015-January/007097.html>] They could either hold off on updating OpenSSL, or use a sufficiently recent version of the source code, which now contained a patch that worked around the issue.

Third, the BIP 66 soft fork was proposed,^[<https://en.bitcoin.it/wiki/BIP_0066>] and it was indeed successfully activated (read more on soft fork activation in chapter @sec:taproot_activation). This fork required that future signatures all abide by the stricter standard, so that both old and new versions of OpenSSL would accept them.

This incident wasn’t necessarily surprising, as OpenSSL is famous for its vulnerabilities. The main reason for this is because these libraries have been used by everyone for decades, but they’re only maintained by a tiny number of volunteers on a shoestring budget.

cURL^[<https://github.com/curl/curl>] is another example of this. It’s a library that downloads files, and it’s used everywhere, but again, there isn’t a well-funded team behind it.

In the case of OpenSSL, one reason it had these bugs is that it’s easy to make mistakes with cryptographic code. What’s more, OpenSSL is written in C, so if you forget a semicolon, whoops, now you’re skipping a line, and perhaps that line was actually checking the password. A famous example of this is the Heartbleed^[<https://gizmodo.com/how-heartbleed-works-the-code-behind-the-internets-se-1561341209>] bug from 2014, in which a small mistake made it so anyone with the know-how could log into any computer on the internet without a password.

When a bug in Bitcoin isn’t discovered and fixed in time, a malicious person can trigger a sudden network split. In the example above, someone could’ve broadcast a transaction with a signature that’s valid according to old versions of OpenSSL, yet invalid according to new versions of OpenSSL. This would cause old nodes to accept the block and new nodes to reject it. Chain splits can be triggered by all sorts of programming mistakes, not just changes in libraries. There have been a few close calls.^[<https://blog.bitmex.com/bitcoins-consensus-forks/>]

While all this was happening, Pieter Wuille^[<https://github.com/sipa>] was working on a library that was specifically designed to create and verify Bitcoin signatures. His original motivation had nothing to do with security; he just wanted it to work faster than OpenSSL.

He explains this in a podcast he did with Chaincode.^[<https://podcastaddict.com/episode/94276066>] Basically, he wanted to make a library that would be about four times faster. He could’ve tried to modify the OpenSSL code itself, but it’s such a nightmare to change that code. Additionally, the OpenSSL code is very generic: It has to support all different kinds of cryptography. So if you want to change anything, you have to be very abstract in all the things you do.

Instead, he decided to essentially write it from scratch, specifically for the secp256k1 curve. It was added to Bitcoin Core relatively early — first just to verify signatures, and then later on to create signatures as well.

This happened to coincide with the aforementioned security vulnerability, and the general reaction was that because there was a near miss which could’ve been a serious problem, moving away from OpenSSL for critical matters would be a good idea.

With signing and signature validation taken care of, Bitcoin still relied on OpenSSL for other things — though much less than it had in the past. But developers had already made the decision to get rid of the remaining OpenSSL uses over time by copying or rewriting the various parts of the library that Bitcoin Core needs. This process was completed^[<https://github.com/bitcoin/bitcoin/pull/17265>] in 2019. The first version of Bitcoin Core to ship without OpenSSL was 0.20.0, which was released in June 2020.

So Wuille’s libsecp256k1 — initially designed to be a performance improvement — pivoted to be a new library for Bitcoin that would remove the risks that came with OpenSSL. However, this came with two risks of its own:

 - Writing your own cryptographic library (this is dangerous outside of cryptocurrency, too).
 - Swapping out one critical library for another, because even the slightest difference in behavior could cause a chain split.

However, it was deemed a risk worth taking, because the other option was waiting for OpenSSL to explode. Additionally, a lot of good cryptographers reviewed libsecp256k1 and compared it against OpenSSL before its adoption. It’s also used by Ethereum and other cryptocurrencies — basically, any cryptocurrency that uses the secp256k1 elliptic curve.^[<https://en.bitcoin.it/wiki/Secp256k1>]
