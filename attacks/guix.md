В этой главе обсуждается программное обеспечение с открытым исходным кодом в контексте того, почему важно, чтобы программное обеспечение Биткоина имело открытый исходный код. Но также здесь раскрывается причина, по которой даже ПО с открытым исходным кодом не обязательно решает все проблемы доверия, связанные с софтом.

Теоретически тот факт, что большинство биткоин-узлов, кошельков и приложений имеют открытый исходный код, должен гарантировать, что разработчики не смогут включать вредоносный код в программы, потому что любой может проверить исходный код на наличие вредоносных программ. На практике, однако, количество людей, обладающих достаточным опытом для этого, ограничено, а зависимость программного обеспечения в целом от внешних библиотек кода еще больше усложняет задачу.

Кроме того, даже если открытый исходный код надежен, это не гарантирует, что двоичные файлы (компьютерный код) действительно соответствуют открытому исходному коду. Первая попытка снизить этот риск для Биткоина включала процесс, называемый Git-сборкой. Несколько разработчиков Bitcoin Core подписывают двоичные файлы на Git, если и только если все они получили одни и те же двоичные файлы из одного и того же исходного кода. Для этого требуются специализированные компиляторы.

Совсем недавно появился Guix, проект, выходящий за рамки процесса Git-сборки. Это помогло свести к минимуму уровень доверия, необходимый для преобразования исходного кода в двоичные файлы, включая доверие к самому компилятору.

### Свободный или открытый код?

Перед тем, как перейти к деталям Git-сборки и Guix, в этом разделе кратко рассмотрим историческую разницу между свободным программным обеспечением и программным обеспечением с открытым исходным кодом и тем, как они были объединены в FOSS (свободное программное обеспечение с открытым исходным кодом).

Идея, лежащая в основе движения за свободный софт, заключается в том, что если программное обеспечение имеет закрытый исходный код, это приводит к отношениям власти разработчиков над пользователями, потому что пользователи не знают, какое программное обеспечение они используют.

В том виде, в котором софт читается вашим компьютером, он написан на языке, который не может понять человек, в двоичном формате, состоящем из единиц и нулей. Но когда люди писали этот софт, они использовали какой-либо язык программирования, вроде C++. Это не одно и то же, даже если большинство смертных тоже не умеют его читать. Поэтому, когда вы используете закрытое программное обеспечение, все, к чему у вас есть доступ — это сам двоичный файл, а не программный код, используемый для его создания. В результате вы понятия не имеете, что делает ваш компьютер.

Так, например, если разработчик помещает вредоносный код в закрытое программное обеспечение, ваш компьютер может шпионить за вами или делать что-то, чего вы не желаете от программы, и вы даже не сможете этого увидеть.

Программисту по имени Ричард Столмен не нравилась идея закрытого софта, поэтому он начал движение за свободный софт, в котором указывалось, что исходный код должен быть доступен, чтобы люди реально могли проверить, что они запускают на своих компьютерах. Это, в свою очередь, устранило силовую динамику. Итак, свободный в этом контексте означает свободу; это не значит "бесплатно", как в "бесплатном пиве".

Несколько иная, но совместимая точка зрения была представлена Эриком С. Рэймондом в его книге 1999 года _Собор и базар: размышления случайного революционера о Linux и открытом исходном коде_.^[<https://en.wikipedia.org/wiki/The_Cathedral_and_the_Bazaar>] В нем он объяснил преимущества свободного программного обеспечения и то, как оно может обеспечить высококачественный код. По его словам, «если обеспечить достаточно глаз, все баги будут мелкими». Другими словами, чем больше фрагмент кода просматривается и проверяется, тем больше шансов, что все его ошибки будут найдены.

Следуя этому прагматичному рассуждению о качестве кода, ребята из Netscape Communications Corporation поддались на уговоры превратить свой внутренний браузер в проект с открытым исходным кодом, Mozilla. Теперь мы называем его открытым исходным кодом, потому что эта группа разработчиков переименовала свободное программное обеспечение в открытое (чтобы избежать путаницы с пивом). В этом и состоит разница между свободным программным обеспечением и открытым исходным кодом.

### Биткоин - проект с открытым исходным кодом.

Теперь вопрос в том, как все это относится к Биткоину. Вот пример. Когда вы запускаете приложение кошелька, оно отображает адрес. Что, если окажется, что адрес принадлежит не вам, а контролируется разработчиком? Тогда каждый раз, когда кто-то платит вам, монеты достаются не вам. Вот почему вам действительно нужна максимальная прозрачность того, что именно работает на вашей машине.

One thing you can do, if you have the skill, is to compile the wallet software yourself, thereby avoiding the need to download an untrusted binary. That doesn’t solve the problem for the vast majority of users though. It also doesn’t entirely solve the problem for you, because even if you can see the code in its original programming language, it’s hard to understand exactly what it’s going to do once it runs on your computer. For one thing, it’s simply too much code for one person to understand.

Если у вас есть навыки, вы можете сделать одну понятную вещь — скомпилировать программу для кошелька самостоятельно, тем самым избегая необходимости загружать ненадежный двоичный файл. Однако для подавляющего большинства пользователей это не решает проблему. Это также не решит для вас проблему полностью, потому что, даже если вы видите код на его исходном языке программирования, трудно точно понять, что он будет делать после запуска на вашем компьютере. Во-первых, тут просто слишком много кода, чтобы его мог понять один человек.

Вот почему вам важно, чтобы любой запускаемый код Биткоина имел открытые исходники, и тогда как можно больше людей могли бы видеть, что это за код. Помните, что если обеспечить достаточно глаз, все баги будут мелкими, и то же самое касается обнаружения вредоносных фрагментов кода.

Код Биткоина имеет открытые исходники и размещен на GitHub в репозитории. Это означает, что любой, у кого есть ноу-хау, может посмотреть исходный код и убедиться, что он делает то, что должен делать. Но на самом деле количество людей, которые действительно могут это сделать и понять, ограничено. ^ [Количество людей, которые могут прочитать этот код, зависит от того, что вы подразумеваете под словом "прочитать". Сколько людей вообще умеет что-то прочесть на компьютере? Сколько человек может приблизительно понять, что делает программа на C++? Вероятно, десятки миллионов (<https://redmonk.com/jgovernor/2017/05/26/just-how-many-darned-developers-are-there-in-the-world-github-is-puzzled/>). Но из них, возможно, только несколько тысяч когда-либо работали в сфере криптовалют или чего-то подобного. Каждый день десятки активных разработчиков отсматривают код. Но никто из них не может контролировать все изменения во всем проекте, потому что это требует специализации: например, один разработчик может знать все о коде одноранговой сети и абсолютно ничего о коде кошелька.] Хотя иногда они даже получают некоторую помощь от разработчиков, которые работают с альткоинами. ^ [Например, очень серьезный баг CVE-2018-17144 был обнаружен разработчиком Bitcoin Cash с ником Awemany (<https://bitcoinops.org/en/topics/cve-2018-17144/>). Многие проекты альткоинов начинались с копипастинга исходного кода Биткоина, с последующим изменением нескольких вещей, чтобы выделиться. Например, Dogecoin изменил график инфляции, уменьшил время между блоками и использовал другой алгоритм proof-of-work. Но при этом 99% его кодовой базы остается идентичной Bitcoin Core: цифровые подписи проверяются таким же образом, транзакции и блоки проверяются таким же образом, одноранговая сеть работает так же и т. д. Поэтому, когда разработчики альткоинов работают над своими проектами, они могут обнаружить ошибки в тех 99% кода, которые они используют совместно с Bitcoin Core. Это повышает безопасность Биткоина.]

If we wanted to increase the number of people who could read and understand Bitcoin source code, it’d need to be cleaner and more readable, because the original code Satoshi wrote was very, very hard to reason about.^[To understand what it means to reason about something, imagine you’re looking at code and you see there’s a function called “make a private key.” Your line of thinking might go as follows: “OK, what does that function do? Oh, it calls in this other function. Where’s that other function? Oh, it’s 20,000 lines up in the same file. Let me scroll 20,000 lines up and have a look at that code. I see, it’s referring to a variable. Oh, but this variable is also accessed in 15 different places in the codebase…”]

Если мы хотим увеличить количество людей, которые могли бы читать и понимать исходный код Биткоина, он должен быть как можно чище и читабельнее, потому что исходный код, написанный Сатоши, было очень, очень трудно понять. ^ [Чтобы понять, что означает подобное понимание, представьте, что вы смотрите на код и видите, что есть функция под названием «создать закрытый ключ». Ход ваших мыслей может быть таким: «Хорошо, что делает эта функция? О, она вызывает вот эту другую функцию. Где еще эта функция? О, она двадцатью тысячами строк выше в этом же файле. Давайте-ка прокрутим на 20 000 строк вверх и посмотрим на ее код. Ага, я вижу, что она обращается к переменной. Ну вот, а к этой переменной идут обращения еще из 15 разных мест в коде…»]

### Checking the Validity

Let’s say you trust the development and release process, so you download the binary from bitcoincore.org. The first problem is that you don’t know if bitcoincore.org is run by the Bitcoin developers. But even if you were confident of that, it could be that the site is hacked, or the site isn’t hacked, but the DNS is hacked. There are many ways in which you could end up downloading malware.

To get around this, open source projects almost always publish a checksum, which is a sequence of numbers and letters. What this means is that if you download something and run a particular script on it, the resulting checksum you get should match what the developers say it should be. The project maintainer usually publishes the checksum on the download page. In theory, that works. However, whoever hacked the site might have also hacked the checksum, so it’s not foolproof.

The next step is to sign the checksum. So, for example, a well-known person — in this case, Wladimir van der Laan, the (Dutch) lead maintainer^[Maintainers aren’t as powerful as some people think they are: <https://blog.lopp.net/who-controls-bitcoin-core-/>\
Also, as of recently, multiple developers sign the release checksum.] of Bitcoin Core — signs the checksum using a PGP key that’s publicly known. It’s been the same for 10 years. So assuming you weren’t fooled the first time, whenever you download an updated version, you know which PGP key the checksums ought to be signed with.

Why trust him? Well, he knows the binaries reflect the open source code because he took the source code, ran a command, and got the binary. In other words, he put the code through some other piece of software that produces binaries from the open source software.

But how do you know he actually did that? Here’s where it gets a little bit more complicated. Ideally, what you do is you run the same command and you also compile it, and then hopefully, you get the same result.

Sometimes that works with a specific project, but as the project gets complicated, it often doesn’t work, because what the exact binary file is going to be depends on some very specific details on your computer system.

Take a trivial C++ program:

```
int main() {
  return 0;
}
```

This program exits and returns `0`. It’s more boring than “Hello, World!”^[<https://en.wikipedia.org/wiki/%22Hello,_World!%22_program>]

Say you compile this on a Mac and it produces a 16,536-byte program. When you repeat that on a different Mac, it produces an identical file, as evidenced by its SHA-256^[<https://en.wikipedia.org/wiki/SHA-2>] checksum. But when you compile it on an Ubuntu machine, you get a 15,768-byte result.

All it takes is one changed letter in a computer program, or in its compiled binary, and boom, your checksum doesn’t work anymore.

If the compiled program includes a library (see chapter @sec:libsecp), then the end result depends on the exact library version that happened to be on the developer machine when they created the binary.

So when you download the latest Bitcoin Core from its website and you compare it to what you compiled yourself, it’s going to have a different checksum. Perhaps the difference is due to you having a more recent version of some library, or perhaps it’s due to a subtle difference between your system and Wladimir’s.

As mentioned above, if you’re one of those lucky people who can compile code yourself, this isn’t a big deal. What’s more likely, however, is that your security depends on the hope that somebody else will do this check for you. Those people might then sound the alarm if anything is wrong.

But because it’s so difficult to check if the source code matches the downloadable binary, should you really assume that anyone out there does this?

### Fixing the Problem with Gitian

In order to verify no shenanigans happened in the process of converting source code to a compiled binary, you need something called reproducible builds, or deterministic builds.

What deterministic implies is that, given a source, you’re going to get the same binary. And if you change one letter in the source, you’re going to get a different binary, but everybody will get the same result if they make the same change.

In addition, there’s the problem of slight differences in machine configuration leading to a different binary file.

Until mid 2021, the way Bitcoin Core did this was with Gitian.^[<https://gitian.org/>] In short, you’d take a virtual or physical computer, download the installation DVD^[Long ago you might have ordered a CD by snail mail, whereas nowadays, you’ll probably download an image and put it on a USB stick. When you install Ubuntu on a virtual machine, your computer creates a virtual DVD player using the image. <https://ubuntu.com/download/server>] for a very specific Ubuntu version, and install that. This ensures everyone has an identical starting position, and because Ubuntu is widely used, there’s some confidence that there isn’t a Bitcoin backdoor on the installation disk.

Inside that machine, you build another virtual machine, which has been tailormade to ensure it builds identical binaries for everyone using it. For example, it uses a fixed fake time so that if a timestamp ends up in the final binary, it’s going to be the same timestamp no matter what time you ran the compiler. It ensures all the libraries are the exact same versions, it uses a very specific compiler version, etc. And then you build Bitcoin Core inside that virtual machine and look at the checksum. This should now match the downloadable files on bitcoincore.org.

About a dozen developers and other volunteers run this “computer within a computer.” Around each new released version, they all compile the binaries and publish the resulting hashes for others to see. In addition, they sign these hashes with their public PGP keys.

However, while this sounds easy in theory, in practice, it’s always been a huge pain to get the system working. There aren’t many open source projects that use Gitian — as far as we know, only Bitcoin Core and Tor do. Even most, if not all, altcoin forks of the Bitcoin Core software don’t bother with this process.

### Dependencies, Dependencies, Dependencies

However, this isn’t the only problem.

Let’s say you just read the Facebook terms and conditions, but it turns out those terms and conditions point to some other document — perhaps the entirety of US copyright law. So now you have to read that too.

Similarly, just reviewing the Bitcoin Core code isn’t enough, because like most computer programs, it uses all sorts of other things, known as dependencies, mostly in the form of libraries (see chapter @sec:libsecp). And each library might in turn use some other library, and so forth and so on. So you need to inspect all of those too.

One of the constraints Bitcoin Core developers work with is to keep the number of dependencies as small as possible, and also to not update them all the time. Such updates require extra review work. And of course, the people who maintain those dependencies know Bitcoin Core is using them; all the more reason to be somewhat on your toes to make sure that those projects are being scrutinized, too.

And if it turns out that a dependency is corrupt, it could steal your coins. This actually happened in at least one other project in 2018. It involved a dependency of a dependency of a dependency of the Copay wallet. Fortunately, it was detected quickly,^[What happened was they had a piece of software that’s open source, meaning everybody could review it. But it used dependencies, and those dependencies used dependencies, and so on.
\
They were using npm, which is the package manager for Node.js. This is, in turn, a large open source community, and it’s a highly modular system.
\
Every single package links to a repository on GitHub, with its own maintainer who can release updates whenever they want. A typical piece of wallet software might be pulling in 10,000 dependencies indirectly. You might start with five dependencies, and each of those pull in 50 dependencies, and those each pull in another 50 dependencies. If even a single one of the developers or maintainers of any of these packages is corrupted, they could include coin-stealing malware.
\
A JavaScript wallet like Copay stores a user’s private keys somewhere inside the browser memory. Unfortunately, that’s a very egalitarian place, meaning that any piece of JavaScript can access it. This is how malware hidden in a sub-sub-dependency can steal coins.
\
For more information, see this writeup: <https://www.synopsys.com/blogs/software-security/malicious-dependency-supply-chain/>] so it was never exploited in the wild.

A more recent example of casual dependency usage gone horribly wrong is the Log4j saga.^[<https://english.ncsc.nl/topics/log4j-vulnerability>]

### The Solution

This begs the questions of what the solution is, and unfortunately, it’s to not depend on dependencies. If unavoidable, it’s important to use as few as possible, and you especially want to stay away from things that have nested dependencies.

In the case of Bitcoin Core, it’s not too bad, because it doesn’t have many dependencies, and those dependencies don’t have a lot of nested ones. So, it’s not a big tree. It’s relatively shallow, and you’d have to go after those specific dependencies directly to attack.

### Who Builds the Builder?

Earlier in this chapter, we discussed how Gitian helps create deterministic builds. But what if Gitian, or any of the tools it uses, is itself corrupted somehow?

For example, since Gitian uses Ubuntu, somebody might say, “Hey, this Bitcoin project’s pretty cool. This Ubuntu project’s pretty cool. Let me contribute some source to Ubuntu.” Their “contribution” could be a small change to the compiler that’s shipped with Ubuntu. They could modify that compiler, so that whenever it compiles Bitcoin, it sneaks in some code to steal coins, but when it compiles any other software, it behaves normally.

This example is a bit contrived, and someone attempting this is very likely to get caught long before they do any damage; there’s much more scrutiny on compiler software and on Ubuntu than there is in for example the Node.js ecosystem we mentioned above. But the general attack strategy would be the same. And with a trillion dollars at stake, attackers can be very sophisticated and very patient.

Now let’s say everybody runs their Gitian builder, which includes this hypothetical compromised Ubuntu compiler. It would be very, very scary, because it’d still have deterministic builds, because everybody is using the exact same malware to build it.

There are two kinds of dependencies: One is the dependency you’re actively running that’s inside the binary you’re shipping to your customers. But the other dependency, and it’s no less a can of worms, is all the tools you’re using to produce the binary, and even to download the binary.

So if even a single one of the tools that developers use to build Bitcoin Core is corrupt, deterministic builds won’t help. Every developer running the Gitian build process would diligently produce the same malware. The binary will not match what’s in the source code.

The hope is that the people who are maintaining all these compilers and all the other things know what they’re doing and would never let any backdoor through. This isn’t just a problem for Bitcoin users. The entire world relies on this scrutiny, which is mostly done by volunteers.

So can we do better?

\newpage
### Enter Guix

The key is to make everything open source and everything a deterministic build. Every library, every printer driver, every compiler — everything. For Bitcoin Core to truly be a deterministic build, each of its dependencies needs to be a deterministic build, as does every tool that’s used to build it, including the compiler. Ideally the hardware is as well, but that’s a whole other can of worms.^[<https://media.ccc.de/v/36c3-10690-open_source_is_insufficient_to_solve_trust_problems_in_hardware>]

This is where Guix^[<https://guix.gnu.org/>] enters the picture. This GNU project has been around for a decade, but a few years ago, Carl Dong^[<https://twitter.com/carl_dong>] from Chaincode Labs^[<https://chaincode.com/>] began work on replacing Gitian with Guix, which finally happened in Bitcoin Core version 22.^[<https://bitcoin.org/en/releases/22.0/>] This involved making changes on both the Bitcoin Core and the Guix side of things.

The ambition of Guix is roughly as follows:^[See also Carl Dong’s presentation: <https://www.youtube.com/watch?v=I2iShmUTEl8>] You start with a few hundred bytes of actual machine code. That’s the binary code that you must trust.^[Even binary code can be seen as source code. It’s simply a series of instructions for the CPU. And this particular machine code is well documented: <https://git.savannah.nongnu.org/cgit/stage0.git/tree/README.org>] From there on, all it does is read source code and compile it. But how do you do that when there isn’t a compiler?

Well, this first binary blob performs a bootstrap.^[In theory, but we’re not there yet. In 2020 Guix project came with 60 MB worth of binary stuff you have to trust. That’s a big improvement over the 1+ GB Ubuntu download used by Gitian: <https://guix.gnu.org/en/blog/2020/guix-further-reduces-bootstrap-seed-to-25/>] It’s able to read some machine code that’s typed by a human, or entered some other way, and then run that program.

The first program you feed it is an extremely simple compiler.^[A compiler turns human-readable code into a binary that the computer can read. This implies that the first compiler is written in binary, which would have to be as small and well documented as possible.] And once it has the rudimentary compiler, this new compiler reads another piece of source code, which then builds a slightly more complicated compiler. And then that slightly more complicated compiler builds another compiler. And this goes on for quite a while, until eventually, you have the modern C compiler that we all know and love, which is itself, of course, open source.

This compiler then builds a bunch of tools from source, ultimately producing a system very similar to Gitian — that is, a build system that avoids timestamps, doesn’t use anything else from your computer, etc. In principle, it could build an entire operating system. So then your virtual machine or your physical machine would be running an operating system that you’ve built from scratch. But in this case, it just builds the compiler tools, and once those compiled tools are there, it can just start building Bitcoin Core as it would otherwise do.

This solves two things. First, it has no untrusted dependencies, so it’s not using random libraries. And second, it’s always using the same versions of libraries, which means that everybody can produce the same result.

Note that Guix doesn’t solve the problem of dependencies entirely. It’s a significant improvement over Gitian, but it’s still critical to keep the number of dependencies small. Where Guix really shines is in mitigating the problem of trusting the build system.
