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

### Для чего проводилась атака?

Через пару недель после этой атаки Матиас Грундманн и Макс Баумстарк написали статью^[<https://arxiv.org/abs/2108.00815>] с описанием атаки и рассуждениями о ее причинах.

Они предполагают, что этот злоумышленник не столько пробовал разрушить сеть, сколько пытался составить карту сети, чтобы понять, насколько хорошо узлы связаны друг с другом. И причина, по которой это вполне реально сделать, заключается в том, что, получив 10 IP-адресов, вы перенаправляете каждый из них ровно двум своим одноранговым узлам. ^ [Дальше процесс не распространится. Хотя вы получили эти спам-адреса аккуратными пакетами по 10, ваш узел будет пересылать их пакетами куда большего размера. Узлы не пересылают адреса в пакете, если он больше 10, поэтому атака прекращается уже после двух прыжков.]

Если злоумышленник также подключается к вам, используя обычный (не рассылающий спам) узел, он получит некоторую часть спам-адресов, который вы пересылаете. Из этой доли он может рассчитать, сколько у вас подключений. В более общем смысле, при подобной атаке наблюдения атакующий как бы отслеживает эхо своей собственной атаки. Глядя на это эхо, он может частично понять, как выглядит сеть, включая ее форму, насколько у нее хорошая связность, насколько она надежна и т. д. Эта информация потенциально могла быть востребована для будущих атак или же могла просто собираться в исследовательских целях.

### Защитные механизмы

У узлов существуют некоторые защитные механизмы, затрудняющие использование этой информации. Например, если вы сообщаете узлу множество IP-адресов, он не сможет сразу подключиться ко всем из них — не только потому, что это слишком упростило бы приглашение узла к подключению к ловушке, но и потому что в большинстве случаев узел уже имеет достаточно соединений. Он также не ретранслирует их все, а для тех адресов, которые ретранслируются, существует случайная временная задержка. Таким образом, очень сложно сказать конкретно, какой узел соединяется с каким узлом, соединенным с каким узлом.

Эти защитные механизмы добавляются в Bitcoin Core постепенно, часто в качестве защиты от атак затмения, как мы рассказывали в главе @sec:eclipse. По сути, когда люди совершают атаки такого типа — странным образом исследуют сеть — опытные разработчики и исследователи безопасности смотрят на них и видят, что они могли бы добавить для противодействия таким атакам.

В этом случае была добавлена контрмера.^[<https://github.com/bitcoin/bitcoin/pull/22387>] Обычно, когда люди ведут себя корректно, они подключаются к вам и отправляют вам один IP-адрес — и именно свой. Время от времени они отправляют вам другие IP-адреса, но не очень часто — средний узел будет делиться IP-адресом с узлом примерно раз в 20 секунд.

Поскольку злоумышленник будет отправлять адреса с гораздо более высокой скоростью, контрмерой будет введение ограничителя скорости. Такой узел как бы говорит: «Хорошо, когда новый узел подключается ко мне, я разрешаю ему отправлять мне один адрес немедленно, а затем я разрешаю до одного адреса каждые 20 секунд». Он отслеживает, сколько секунд прошло, и если узел отправляет слишком много адресов, он будет игнорировать новые, которые превышают лимит частоты. Таким образом, «злоумышленников» не наказывают, а скорее игнорируют.^[Чрезмерное наказание за плохое поведение может привести к разделению сети, что злоумышленник может даже использовать, обманывая обычные узлы, чтобы они «разозлили» своих пиров и отключились.]

Конечно, бывают случаи, когда узлы действительно хотят получить от своих пиров много адресов. Например, если кто-то подключается к вам, и вы говорите: «Пожалуйста, сообщите мне адреса, до тысячи штук», то, конечно, ответ не нужно ограничивать по скорости. В таком сценарии вы будете уверены, что вы действительно сможете получить от них эти адреса, но если вы их не запрашиваете, то просто ограничиваете скорость.

### Ответственное раскрытие информации

Now, what’s really interesting is that this fix was published in the weeks _before_ the attack happened. It wasn’t merged into Bitcoin Core yet, but rather just an open pull request, or proposed change. It remained open until shortly after the attack, but then it was merged and released in version 22.0.

So it almost sounds like somebody saw the solution and saw an opportunity to carry out this specific attack. Or perhaps somebody was already planning this attack and then figured they should do it soon, before it was no longer possible.

For more on this attack and related issues, there’s also a great Chaincode Labs podcast episode.^[From 23:15 “Rate limiting on address gossip”: <https://podcast.chaincode.com/2021/10/26/pieter-wuille-amiti-uttarwar-p2p.html>]

There have been other examples of a situation where publishing a fix may itself have caused the attack. Back in the day, there was an alternative node implementation Bitcoin Unlimited. It had a bug that was fixed, but before the fix was deployed, the bug was exploited by somebody. That attack brought down all the Bitcoin Unlimited nodes at that time.^[<https://bitcoinmagazine.com/technical/security-researcher-found-bug-knocked-out-bitcoin-unlimited>]

Additionally, around 2013, something similar didn’t happen, but could’ve happened, on Bitcoin, which we covered in chapter @sec:libsecp. This is because the OpenSSL library was made stricter in its software by imposing constraints on signatures. Had it been discovered only a few months later, after a Bitcoin Core release was already out there with the bug in it, then it would’ve been a zero-day situation. Perhaps in that case, the fix would’ve been disguised as nice cleanup software, rather than a patch for a security vulnerability. If someone had found out, they could’ve posted a slightly different kind of signature and caused a fork because some nodes would accept it and other nodes wouldn’t accept it.

A final example is that of an inflation bug in 2018.^[<https://bitcoincore.org/en/2018/09/20/notice/>] It was presented as a fix to a bug that will crash your node. And that was true — it could crash your node — but it could also create inflation. The latter fact was a bit more important, and it was of course not announced, because somebody could’ve exploited it in that window of opportunity.

One way to handle scenarios like this is for developers to pretend that something isn’t a big deal until people have actually downloaded and used the fix — and then to reveal only later that it was actually a much bigger problem they were fixing. This makes it less likely that an attacker detects the vulnerability while it’s still exploitable.

But that’s really not an ideal approach, because in open source development, you want to be very transparent about things you’re changing. Because if you’re not being transparent about fixing a critical bug, then maybe you’re also not transparent about adding inflation. It’s a delicate balance.
