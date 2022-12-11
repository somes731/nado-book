\newpage
## Фейковые узлы {#sec:fake_nodes}

\EpisodeQR{49}

В этой главе рассказывается об атаке, имевшей место летом 2021 года. В ней обсуждается, что произошло, выдвигаются предположения, почему это могло произойти, и рассказывается об исправлении, которое предотвратит повторение этой атаки.

<!-- Blank line to move the next section header below the QR code -->
\

### Случайные подключения

В середине 2021 года владельцы узлов начали замечать, что к ним подключаются случайные люди. ^[Для прочтения ветки, в которой упоминается об этой атаке, см. <https://bitcointalk.org/index.php?topic=5348856.0>] Это само по себе совершенно нормально. Как мы объясняли в главе @sec:dns, это часть механизма первоначальной загрузки узлов в сеть. Новык узлы случайным образом подключаются к существующим узлам и запрашивают адреса других узлов для подключения еще и к ним. Они также объявляют свой собственный IP-адрес, о котором начинают ходить сплетни, поэтому достаточно скоро узел будет получать входящие соединения.

Однако в данном случае было необычно то, что эти случайные люди подключались к ним, а затем отправляли 500 сообщений, ^[<https://developer.bitcoin.org/reference/p2p_networking.html#addr>] и каждое из этих 500 сообщений содержало 10 IP-адресов, которые должны были представлять другие узлы в сети. После этого они просто отключались. Это, конечно, не казалось опасным, но это было не обычным поведением.

Хотя сообщения были абсолютно достоверными, их содержание было бессмысленным, потому что IP-адреса, отправляемые этими узлами, были просто случайно сгенерированными числами. Вы могли бы понять это, если бы нанесли их на карту; шаблон распределения будет соответствовать шаблону случайно сгенерированных чисел. Другой способ, которым вы могли бы это установить, состоит в том, что список содержал IP-адреса, которые просто не могут существовать по разным причинам, например. потому что они зарезервированы для частных сетей, таких как 192.168.0.1.

Проблема с этими случайно сгенерированными IP-адресами заключается в том, что, если вы перегружены ими, практически невозможно подключиться к реальному узлу. Существует менее ста тысяч узлов, к которым может подключиться ваш узел, но существует четыре миллиарда адресов IPv4. Цель протокола сплетен об адресах как раз и состоит в том, чтобы предотвратить подобное случайное угадывание. Но силы этой атаки было недостаточно, чтобы привести к сбоям отдельных узлов.

По мере изучали этого явления было обнаружено, что это происходит в довольно больших масштабах, и такое поведение классифицировали как атаку. На самом деле такая атака не представляет большой проблемы для отдельного узла, особенно если у него уже есть много IP-адресов от честных нод. Он может подключаться к нескольким несуществующим узлам, но в основном это пустая трата времени и ресурсов, поскольку он подключается и сохраняет IP-адреса, которые не являются реальными IP-адресами биткоин-узлов. Так что на индивидуальном уровне это похоже по эффекту на то, как ребенок бросает в вас маленький камешек.

Кроме того, мы знаем, что все это не имело большого значения, просто потому что почти никто даже не заметил происходящего. Но все же явление заслуживает расследования.

### Почему кто-то атакует?

A couple weeks after this attack, Matthias Grundmann and Max Baumstark wrote a paper^[<https://arxiv.org/abs/2108.00815>] describing the attack and speculating about the reasoning behind it.

What they’re guessing is that this attacker wasn’t so much trying to destroy the network, as they were trying to map the network to get a sense of how well nodes are connected to each other. And the reason they can do that is because when you receive 10 IP addresses, you’ll forward each of them to exactly two of your peers.^[It won’t propagate further. Although you received these spam addresses in neat packages of 10, your node will forward them in bigger packages. Nodes don’t forward any addresses in a package if it was bigger than 10, so the attack fizzles out after just two hops.]

If the attacker also connects to you using a regular (not spamming) node, it’ll receive some fraction of the spam address that you forward. From that fraction, it can calculate how many peers you have. More generally, in a surveillance attack like this, it’s like they’re monitoring the echo of their own attack. By looking at this echo, they can determine a little bit of what the network looks like, including the shape of it, how well connected it is, how robust it is, etc. This information could potentially be used for future attacks, or it could just be for research purposes.

### Defense Mechanisms

There are some existing defense mechanisms in nodes that make it more difficult to use this information. For example, if you’re telling a node a bunch of IP addresses, it’s not going to immediately connect to all of them — not only because that would make it too easy to invite a node into connecting to a trap, but also because, most of the time, a node already has sufficient connections. It also doesn’t relay all of them, and for those addresses that are relayed, there’s a random time delay. So, it makes it very difficult to say specifically which node connects to which node connects to which node.

These defense mechanisms are added to Bitcoin Core incrementally, often as a defense against eclipse attacks, as we explained in chapter @sec:eclipse. Essentially, when people do these types of attacks — probing the network in weird ways — experienced developers and security researchers will look at them and see how they can add something against them.

In this instance, they added a counter measure.^[<https://github.com/bitcoin/bitcoin/pull/22387>] Normally, when people are acting nice, they’ll connect to you and send you one IP address — namely their own. Occasionally, they’ll send you some other IP addresses, but not very frequently — the average node will share an IP address with a peer maybe once every 20 seconds.

Because an attacker will send addresses at a much higher rate, the counter measure is to introduce a rate limiter. This basically says, “OK, when a new node connects to me, I’ll allow it to send me one address immediately, and then I’ll allow up to one address every 20 seconds.” It tracks how many seconds have gone by, and if the node is sending too many addresses, it’ll ignore the new ones that go over the rate limit. So the “attackers” don’t get punished, but rather ignored.^[Overzealously punishing bad behavior can lead to network partitioning, which is something that an attacker could even exploit by tricking regular nodes into “angering” their peers and getting themselves disconnected.]

Of course, there are cases where nodes actually want to receive lots of addresses from their peers. For example, if somebody connects to you and you say, “Please tell me addresses, and give me up to 1,000,” then of course the response won’t be rate limited. In such a scenario, you’ll make sure that they can actually give you those addresses, but if it’s unsolicited, then you rate limit it.

### Responsible Disclosure

Now, what’s really interesting is that this fix was published in the weeks _before_ the attack happened. It wasn’t merged into Bitcoin Core yet, but rather just an open pull request, or proposed change. It remained open until shortly after the attack, but then it was merged and released in version 22.0.

So it almost sounds like somebody saw the solution and saw an opportunity to carry out this specific attack. Or perhaps somebody was already planning this attack and then figured they should do it soon, before it was no longer possible.

For more on this attack and related issues, there’s also a great Chaincode Labs podcast episode.^[From 23:15 “Rate limiting on address gossip”: <https://podcast.chaincode.com/2021/10/26/pieter-wuille-amiti-uttarwar-p2p.html>]

There have been other examples of a situation where publishing a fix may itself have caused the attack. Back in the day, there was an alternative node implementation Bitcoin Unlimited. It had a bug that was fixed, but before the fix was deployed, the bug was exploited by somebody. That attack brought down all the Bitcoin Unlimited nodes at that time.^[<https://bitcoinmagazine.com/technical/security-researcher-found-bug-knocked-out-bitcoin-unlimited>]

Additionally, around 2013, something similar didn’t happen, but could’ve happened, on Bitcoin, which we covered in chapter @sec:libsecp. This is because the OpenSSL library was made stricter in its software by imposing constraints on signatures. Had it been discovered only a few months later, after a Bitcoin Core release was already out there with the bug in it, then it would’ve been a zero-day situation. Perhaps in that case, the fix would’ve been disguised as nice cleanup software, rather than a patch for a security vulnerability. If someone had found out, they could’ve posted a slightly different kind of signature and caused a fork because some nodes would accept it and other nodes wouldn’t accept it.

A final example is that of an inflation bug in 2018.^[<https://bitcoincore.org/en/2018/09/20/notice/>] It was presented as a fix to a bug that will crash your node. And that was true — it could crash your node — but it could also create inflation. The latter fact was a bit more important, and it was of course not announced, because somebody could’ve exploited it in that window of opportunity.

One way to handle scenarios like this is for developers to pretend that something isn’t a big deal until people have actually downloaded and used the fix — and then to reveal only later that it was actually a much bigger problem they were fixing. This makes it less likely that an attacker detects the vulnerability while it’s still exploitable.

But that’s really not an ideal approach, because in open source development, you want to be very transparent about things you’re changing. Because if you’re not being transparent about fixing a critical bug, then maybe you’re also not transparent about adding inflation. It’s a delicate balance.
