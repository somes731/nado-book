\newpage
## DNS Bootstrap и Tor V3 {#sec:dns}

\EpisodeQR{13}

В Bitcoin Core 0.21 в 2020 году добавлена поддержка адресов Tor V3.^[<https://github.com/bitcoin/bitcoin/pull/19954>] В этой главе объясняется, что это значит и почему это важно. Также будет обсуждаться, как новые биткоин-узлы находят существующие биткоин-узлы при первичной загрузке в сеть.

### Как работает Tor?

Когда вы видите адрес Tor, ^[например, на сайт <https://bitcoincore.org> также можно попасть и с помощью браузера Tor по адресу <http://6hasakffvppilxgehrswmffqurlcjjjhd76jgvaqmsg6ul25s7t3rzyd.onion/>], он выглядит довольно странно. Дло в том, что это не удобочитаемое имя, такое как домен, а скорее открытый ключ, который соответствует скрытому сервису где-то в Интернете. Поскольку вы не знаете его IP-адреса, то вы общаетесь с этим скрытым сервисом не напрямую, а косвенно, через сеть Tor.

Tor (сокращение от The Onion Router) — это луковичная сеть, в которой сообщения передаются по сети через несколько переходов (или серверов), при этом каждый переход снимает с сообщения один зашифрованный слой, как с луковицы. Последний переход отправляет сообщение в конечный пункт назначения, который снимает последний слой шифрования, раскрывающий фактическое сообщение. Это позволяет легко поддерживать анонимность и безопасность.

Для подключения вам нужно использовать браузер Tor.^[<https://www.torproject.org/download/>] Этот браузер создает луковые пакеты. Сообщения в них — это обычные вещи, которыми обмениваются браузеры: скажем, запрос HTML-документа или изображения и, в обратном направлении, получение указанного документа или изображения. Браузер Tor сначала создает само исходное сообщение. ^ [По факту все немного сложнее: чтобы защитить конфиденциальность получателя, отправитель упаковывает луковый пакет только до промежуточного узла рандеву, который затем пересылает сообщение дальше.] Далее он упаковывает поверх него другое сообщение с инструкциями о том, где находится конечный пункт назначения, и которое сможет прочитать только последний узел перед скрытым сервисом. Затем он заворачивает поверх другое сообщение с инструкциями для предпоследнего узла о том, как достичь последнего узла, и так далее.

Под капотом этот процесс использует IP-адреса, но вы не знаете IP-адрес целевого узла Tor, с которым вы общаетесь. Вместо этого вы общаетесь с другими узлами Tor, и каждый из этих узлов связывается с теми узлами, с которыми он напрямую связан. Таким образом, все знают только IP-адреса своих прямых пиров, но не знают, откуда пришло сообщение и куда оно попадает. Кроме того, они не могут прочитать сообщение, поскольку оно зашифровано.

Чтобы поддерживать этот механизм, все узлы Tor имеют свой собственный вид IP-адреса — онион-адреса — и именно с ними вы общаетесь напрямую. Между тем, узлы Bitcoin Core могут работать за такими скрытыми сервисами, и это означает, что каждый может запустить свой узел Bitcoin в секретном месте, в результате чего его IP-адреса останутся скрытыми.

### Запуск узла Биткоина из-под Тора

По тем или иным причинам вы можете не захотеть, чтобы остальной мир знал, что на вашем IP-адресе запущен биткоин-узел. В частности, вы можете не хотеть, чтобы ваши биткоин-адреса ассоциировались с вашим IP-адресом, поскольку первый говорит о том, сколько у вас денег, а второй часто может быть напрямую связан с вашим именем и адресом — не только для правительства, но и для кого-то с доступом, например, ко взломанной базе данных интернет-магазина с IP-адресами и домашними адресами своих клиентов. Это может привести к неприятным результатам.^[<https://github.com/jlopp/physical-bitcoin-attacks>]

Биткоин-узлы уже пытаются вести себя так, чтобы их поведение было неотличимо от других узлов. В идеале узел не раскрывает другим узлам, какие монеты он контролирует. Узел загружает всю цепочку блоков и отслеживает все транзакции в мемпуле^[Мемпул — это очередь транзакций, которые еще не были подтверждены в блоке. См. приложение @sec:more_eps в книге _Разъясненный Биткоин_. Когда в мемпуле содержится много транзакций, комиссии, как правило, растут. См., например. <https://mempool.space/>] вместо получения информации только о своих монетах.

К сожалению, система не идеальна. Тщательный анализ сети злоумышленником иногда может выявить, откуда произошли транзакции, особенно когда вы отправляете и получаете транзакции со своего IP-адреса. Этот тип анализа — бизнес на миллиарды долларов, что не способствует этичному поведению компаний. ^[<https://www.coindesk.com/business/2021/09/21/leaked-slides-show-how-chainalysis-flags-crypto-suspects-for-cops/>, <https://www.coindesk.com/markets/2019/03/05/coinbase-pushes-out-ex-hacking-team-employees-following-uproar/>]

Таким образом, использование биткойнов из-под Тора^[<https://github.com/bitcoin/bitcoin/blob/master/doc/tor.md>] может улучшить вашу конфиденциальность, разорвав связь между вашим IP-адресом и любой информацией о вас, которые ваш узел может случайно раскрыть.

В результате обновления протокола Tor появился новый тип онион-адреса: Tor V3. Он и рекомендуется для практическго использования. Эти новые адреса Tor длиннее, что делает их более безопасными.^[<https://blog.torproject.org/v3-onion-services-usage>] Однако для поддержки этой увеличенной длины потребовалось обновление Bitcoin Core.

### Bitcoin Nodes and Gossip

Why did the longer Tor V3 address require a Bitcoin Core upgrade? This has to do with how Bitcoin nodes spread the word about where they are. Nodes communicate with each other in a gossip network: They send each other lists of known nodes, and they ask each other, “Hey, which Bitcoin nodes do you know?” In return, they get a list of IP addresses, which are usually IPv4 or IPv6 addresses.

IPv6 addresses were formalized in 1998 with the intention of replacing IPv4, because the number of IPv4 addresses was limited. There are nearly 4.3 billion potential unique IPv4 addresses,^[<https://en.wikipedia.org/wiki/IPv4>] whereas there are enough IPv6 addresses for every molecule in the universe.

Bitcoin nodes keep lists of other Bitcoin nodes and their IP addresses, both IPv4 and IPv6. Now the way they communicate a Tor address is to piggyback on IPv6. If an “address” starts with fd87::d87e::eb43, then Bitcoin Core knows that what follows should be interpreted as a Tor address. RFC-4193 ensures that such addresses won’t clash with any computer in the real world.^[<https://datatracker.ietf.org/doc/html/rfc4193>]

The problem with Tor V3 addresses is they’re 32 bytes, which is twice as long as an IPv6 address. That doesn’t fit in the RFC-4193 piggyback mechanism, so nodes had no way to communicate those addresses.

Fortunately in 2019, Wladimir van der Laan wrote a new standard — BIP155 — for how to communicate addresses.^[<https://github.com/bitcoin/bips/blob/master/bip-0155.mediawiki#Specification>] It introduces the new ADDRv2 message, which nodes can use to gossip those new Tor addresses (among other things). A major improvement is that each message specifies the type of address along with the address itself. This removes the need to piggyback. There are various address types, including the new Tor one, and each address type can have a different length. So, in the future, if a new address format comes along, it’s not going to be a problem.^[For example, I2P (Invisible Internet Project, an alternative to Tor) support was added in 2021: <https://github.com/bitcoin/bitcoin/blob/7740ebcb023089d03cd2373da16305a4e501cfad/doc/i2p.md>]

The nice thing about this new peer-to-peer message is that old nodes just ignore it. And if your node knows it’s talking to an old node, it won’t use ADDRv2. So newer nodes will know this new message and can communicate all these new address types, and old the nodes carry on like nothing happened. Unless you want to use Tor V3, you’re not required to upgrade.

However, since the Tor project _is_ centralized, it can and did force users to — with a long grace period — upgrade from Tor V2 to V3. So if you relied on Tor V2 for the privacy of your Bitcoin node, you’ll have no choice but to upgrade your node.

### How DNS Works

But how do you connect to that first node or bootstrap to the network?^[<https://stackoverflow.com/questions/41673073/how-does-the-bitcoin-client-determine-the-first-ip-address-to-connect>]

Assume you just downloaded Bitcoin Core or some other client, and you started up. Now what? Is it just going to guess random IP addresses? No. It needs to know at least one other node to connect to, but preferably more than that. The way it tries to connect is using something called DNS seeds. The internet DNS system is used for websites, e.g. you type an address like www.google.com, and what your browser does is it asks a DNS server what IP addresses are from that Google domain.

The DNS system is ultimately centralized. So basically, if you run a website, your hosting provider will have a DNS server that points to your website, and your country will have a DNS server that points to your hosting provider, and your internet provider will have a DNS server that points to all these different countries, etc.

If you’re maintaining a website, you usually have to go into a control panel and type in the IP address of your server, as well as your domain name, and that’s stored on the DNS server. One of the fields you have to fill out is the timeout. This is how long others on the internet may assume this IP address still belongs to your website.

So, when you’re visiting a website, you’re going to ask your ISP, “Hey, do you know the IP address for this website?” If it doesn’t, it’s going to ask the next DNS server up the street, “Do you know it?” And then as soon as it finds a record, it’s going to say, “OK, is this record still valid or is this expired?” If it’s still valid, it’ll use it, and if it’s expired, it’ll go up closer and closer to the actual hosting provider. So it’s basically cached.

Because of this caching, DNS records are stored very redundantly. That’s good for both privacy and availability.

Bitcoin kind of abuses this system, because Bitcoin nodes aren’t websites. There are a couple of Core developers who run DNS seeds, which are essentially DNS servers. And we’re just pretending that, for example, seed.bitcoin.sprovoost.nl is a “website,” and when you ask that “website” what its IP address is, you get a whole list of IP addresses. However, those IP addresses are Bitcoin nodes, and every time you ask, it’s going to give you different IP addresses.

A DNS seed is just a simple crawler.^[<https://github.com/sipa/bitcoin-seeder>] It calls a random Bitcoin node, asks it for all the nodes it knows, keeps a list, goes through the list, and pings them all. Then, once it’s done pinging them all, it’s just going to ping them all again.

This means that the standard infrastructure of the internet — including all the ISPs in the world — is caching a huge list of Bitcoin nodes that you can connect to, because it thinks they’re just websites. It also allows Bitcoin to piggyback on any protections against censorship built into DNS.^[Matt Corallo tried to take things even further by publishing block headers via DNS: <https://github.com/bitcoin/bitcoin/pull/16834>]

### So We Trust These Developers?

What if one of the DNS seed operators were to lie and provide a list of fake or somehow malicious nodes? Perhaps as part of an elaborate eclipse attack (see chapter @sec:eclipse). Nothing would stop them, but it would be very visible. Anyone can request IP addresses from the DNS seed and then check if they actually lead to Bitcoin nodes or not, and if these nodes are behaving in suspicious ways. This visibility discourages cheating.

Another potential problem would be if none of the DNS seeds are reachable because, for example, they’re offline. For that scenario inside the Bitcoin Core source code (and thus also the binary you download) is a list of IP addresses, as well as some hidden services.

Every six months or so, all the DNS seed maintainers are asked to provide a list of the most reliable nodes — just all the nodes sorted by how frequently they’re online, i.e. which DNS seeds keep track of. The Bitcoin Core developers combine that information from all the DNS seed operators and that goes into the source code.^[<https://github.com/bitcoin/bitcoin/blob/v22.0/contrib/seeds/nodes_main.txt>]

Both DNS seeds and the baked-in fallback addresses are, ideally, only used once in the lifetime of your node: when it starts up for the very first time. After that, your node keeps track of the nodes it learns about by storing all these gossiped nodes in a file. When it restarts, it opens the file and tries some random nodes from it. Only if it runs out of new IP address to try, or if it takes too long, does it ask the seeds again.

Whenever a node connects to you for the first time, one of the first things it asks is: “Who else do you know?” Your node can even send IP addresses to its peers unsolicited. In particular, it announces its own IP address to them. As your IP addresses is gossiped further around the network, you start getting inbound connections.

And with that, your node is up and running!
