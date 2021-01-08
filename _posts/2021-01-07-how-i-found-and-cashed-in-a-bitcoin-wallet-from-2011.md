---
layout: post
title: How I found and cashed in a bitcoin wallet from 2011
comments: true
tags: [bitcoin, BTC, electrum, wallet.dat]
---
A few days ago I found a Bitcoin wallet.dat on a Macbook from 2011. Here is how I managed to cash in on it.<span class="more"></span>

# A Bitcoin wallet from 2011...
In 2011 my employer sent me and a co-worker to the Netherlands for two weeks to work on a research project. There, for the first time, someone told me about Bitcoin. While I found the ideas fascinating I could not make a lot of sense of it. And certainly there was no way how to ever either buy something or transfer any bitcoins to real fiat money. Nevertheless, I installed Bitcoin core v0.3.19 BETA on my MacBook Pro, and somehow managed to get my hands on a fraction of a bitcoin. Today, the value of that fraction is enough to buy me new laptop or go on a vacation for some days. I don't quite remember whether I managed to get those fractions through mining, or whether one of the members of the research project transferred it to my account as a small gift. Certainly, back then it did not have a lot of value.


# ...appears again in 2021.
Fast forward. I am still using said MacBook from 2011 to watch movies every now and then as it has an in-built DVD player and actually works more or less flawlessly still today. I never bothered to re-format its hard disk, and that's my luck. While opening the DVD player application I suddenly spotted an installation of Bitcoin Core that I have entirely managed to ignore throughout all those years. I started the application and found an address. With 0 bitcoins. I browsed a bit further and realized that there was also a second address, not displayed with default settings, and that second address claimed to contain said fractions of a bitcoin.

I was thrilled. I went to the internet to check. The address indeed contained some fractions of a bitcoin. I searched my file system for a wallet.dat file and got lucky. There it was, under <code>~/Library/Application Support/Bitcoin/</code>.

First, I made sure all my Wifi and Bluetooth connections on the laptop were turned off, and the laptop was not connected to the internet. Better safe than sorry. Second, I created a safety copy of the entire directory and put it on a memory stick. Third, I removed all access rights from the wallet.dat file using <code>sudo chmod a-rwx wallet.dat</code> to ensure nobody, except myself, could by mistake or with bad intent read out or modify the wallet.

And then the real work began. Note: I have never really looked into the whole Bitcoin affair more in-depth. I'm actually skeptical that it's so secure as many people believe simply due to the fact that the big miners actually build something sort of [an oligopoly](https://www.blockchain.com/pools?). [Some argue](https://www.forbes.com/sites/rogerhuang/2021/12/29/the-chinese-mining-centralization-of-bitcoin-and-ethereum/?sh=55c0871a2f66) the large-scale bitcoin miners in China/Asia may not be able or willing to manipulate the blockchain despite the fact there are not too many of them left. But that's a different topic. What I wanted was to get cash out of the BTCs in my wallet. How is it done?

The first problem I faced was that the installed version of Bitcoin Core v0.3.19 BETA, irrespective of whether it would have worked still or not (probably not, but I did not even bother to check), I did not consider to be secure anymore. Also my laptop did not have any updates in years, so I did not consider the laptop to be secure neither.

And, to complicate things, I was not even sure whether my wallet was encrypted or not, and if so, what the password would have been. I might have been able to guess it, but I was not sure.

I own a new MacBook Pro, so in theory I could have installed a new version of Bitcoin Core there and then try to import the old wallet.dat. But the problem was that Bitcoin Core these days require at least 320 GB of disk space, and I only have 512 GB available, with roughly half of it already filled. So, there simply was not enough disk space available.

After much reading I decided to install [Electrum](https://electrum.org/) wallet instead. The difference between Bitcoin Core and Electrum is that the latter is a so-called [simplified payment verification (SPV) wallet](https://en.bitcoinwiki.org/wiki/Simplified_Payment_Verification). In short, the consequence is that Electrum does not require to download the total 320 GB of blockchain history, but instead is a fully functional wallet that is still sufficiently secure for the majority of purposes, although not exactly as secure as the Bitcoin Core wallet. At least for the fraction of bitcoins I owned it seemed to be secure enough.
I was a bit paranoid, so I even checked the checksum using [GPG Suite](https://gpgtools.org/) after downloading it to ensure it hadn't been fiddled around with. Apparently, some time ago there had existed a fake website called electrum.li which offered a manipulated version of Electrum, so some paranoia might actually be justified.

Next problem was that I had no clue how to import my 2011 Bitcoin wallet.dat into Electrum. I searched around and found [this medium.com post from 2017](https://medium.com/@simonvc/i-found-an-old-bitcoin-wallet-1aeb2f32387a) that in the end turned out to be quite helpful yet leaving out some details. From what I understood from this site and other related material there was simply no way how to import my old wallet.dat file into Electrum, they were incompatible. 

Instead, it was suggested to use _pywallet_, a Python 2.7 program that could read out old wallet.dat files from 2011 such as mine. The author of referred medium article had used a version of _pywallet_ available from https://github.com/jackjack-jj/pywallet/blob/master/pywallet.py. I cloned it into my local file system...

...and faced the next problem. According to the [installation instructions](https://github.com/jackjack-jj/pywallet) it was required to pip install several Python 2.7 libraries that were entirely straight forward to install. My version of Mac OS X Catalina (10.15.7) is shipped with a Python version 2.7. However, I did not want to fiddle around with that version to avoid overwriting some system libraries with newer versions by mistake and potentially cause some Mac OS X processes to fail. Unfortunately, Python 2.7 does not properly support virtual environments. So, I decided to follow the MacPorts route.

This is from pywallet's installation instructions:
<code>
Mac OS X:
1. Install MacPorts from http://www.macports.org/
2. sudo port install python27 py27-twisted py27-pip py-bsddb python_select
3. sudo port select --set python python27
4. sudo easy_install ecdsa
</code>

You might also run into issues if you don't have [xcode](https://apps.apple.com/us/app/xcode/id497799835?mt=12) installed, which I fortunately had and did not have to bother about that any further.

I decided not to actually switch the Python version with MacPorts, but simply directly refer to the installed Python 2.7 version that is different from my system's pre-installed one. This new installation was to be found in <code>/opt/local/Library/Frameworks/Python.framework/Versions/2.7/</code>, and there was a python2 executable in the <code>./bin</code> folder.

On my new laptop I had created a copy of my old laptop's whole directory with the wallet.dat file. Then I ran pywallet:

<code>/opt/local/Library/Frameworks/Python.framework/Versions/2.7/bin/python2 ~/<path to pywallet>/pywallet.py --dumpwallet --datadir=~/<path to copy of Bitcoin wallet directory> > ~/wallet.dat.json</code>

I was not sure this would work, because the wallet.dat might have been password protected. I decided to try my luck...

...and was lucky! It worked like a charm. As it turned out, most wallets from Bitcoin Core from 2011 are actually not encrypted. Apparently, this was not enforced back then by Bitcoin Core v0.3.19 BETA, and I was simply able to open my json file and read out its content.

The json file contained lots and lots of different addresses, including their private and public keys. What a mess. A typical entry looked like so:

<code>
{
"addr": "<this is an address>", 
"compressed": false, 
"hexsec": "<64 hex char string>", 
"private": "<558 hex char string>", 
"pubkey": "<130 hex char string>", 
"reserve": 1, 
"sec": "<51 char string>", 
"secret": "<64 char string>"
}
</code>

After plenty of reading up I finally understood that Electrum required the 51 character string in the "sec" field. This is in essence a _wallet import format_ (WIF) Base58 encoded private key. In my situation, having still access to the old Bitcoin Core installation from 2011 I also knew exactly which address would contain the bitcoins. If I had not known that then I would have needed to go through the pain of extracting all addresses with a script and checking all of against some public website or service to see which one actually contains the bitcoins. This is described in just superficial details in already mentioned [medium.com article](https://medium.com/@simonvc/i-found-an-old-bitcoin-wallet-1aeb2f32387a). Pywallet actually contains a command for this purpose, where the blockchain address corresponds to the value of the "addr" field above.

<code>python2 pywallet.py --balance=<blockchain address></code>

Also, you might need to build in a small waiting time between requests to not get blocked by the blockchain information provider service.

In my case I could simply search for the corresponding entry in the json file.

With that information I could not start my Electrum application. I had installed Electrum v4.x. After starting, I first selected _create new wallet_, provided a name, and then selected _Import Bitcoin addresses or private keys_. At some point in the process I also had to select a strong password if I remember correctly.

![Import private key into Electrum](/public/img/2021-01-07-electrum-import-private-key.png)

In the input field I only needed to enter the 51 character long WIF Base58 encoded private key, then I could click the _next_ button.

Now I was ready. What confused me first was that Electrum does not show the addresses themselves, you need to explicitly enable those in the menu under _> View > Show Addresses_.

![Show addresses in Electrum](/public/img/2021-01-07-electrum-show-addresses.png)

Great. No import of 320 GB data needed at only minimal loss of trust. (It might be wise for you to read up on Electrum's security first and which servers it connects to and what further options it offers you.)

Now I needed to find an address to send my BTCs to, then exchange the BTCs to my local currency, and finally withdraw the money. The situation has changed drastically from 2011 - but it's still not a painfree process.

First, I checked some local bitcoin ATMs that I had read about in the press. Well, if you are prepared to spend some outrageous commissions of 3.5% - 5.7% of the amount of money withdrawn PLUS additional fees for whatever PLUS accept that sometimes the ATMs are just empty without notifying you PLUS accept that some of them don't let you withdraw more money than ca. 1000 CHF at a time PLUS some random error messages, then this might be a truly great option for you. For me it wasn't.
I then began researching on trading platforms and how they work. A co-worker suggested me to try out [Lykke](https://lykke.com/) claiming they did not charge extra fees to transfer fiat money to any bank account through the SWIFT payment system. I briefly also checked out Bitcoinsuisse.com who notified me at some stages during the registration process that the minimum deposit be 10k CHF. Thanks, but no thanks. I also checked out kraken.com but finally decided to set up an account on lykke.com.
Registration process went more or less smooth, just some minor hiccups that apparently my free email I've been using for years was silently not accepted without any error messages being displayed anywhere. This caused me to scratch my head a bit, but ultimately I settled for a gmail address instead and that worked. But that only led me to the very basic account with which you can do essentially nothing. To be able to trade bitcoins to fiat money and then to withdraw I had to provide a foto of my identity card, upload some bank statement as proof, take a selfie, give them my fingerprints and DNA sample and I was good to go. Well, the last two are an exaggeration, but you get the drill.

With all that done I waited for ca. 4 hours, then received another email that my application was accepted and voilà.

The web tool was really easy to use. There was a blockchain address given to me to send my bitcoins to, and then, with much trembling and praying, from my Electrum application I sent all my bitcoins to said blockchain address in my Lykke account. This cost me a small fraction of my BTCs in accordance to how the whole blockchain protocol for bitcoin works. (If you don't know what I'm talking about here, you may want to read up on [Electrum ETA](https://bitcoinelectrum.com/how-to-manually-set-transaction-fees/). Transactions on blockchain are not free, but they are not too expensive neither.) Maybe I should have started with sending only a fraction of it, I pondered afterwards, while waiting until the blockchain had accepted my payment. But after ca. 30 minutes I had enough confirmations and the bitcoins did arrive at my Lykke account. :D

Next step was to trade them from BTCs to CHFs. Trading tool was easy to use if you are used to that kind of thing. If not, I highly recommend you get some help. There probably is not too much you can do wrong - other than inadvertedly buying or selling the wrong crypto-currency or the wrong type of fiat money. The trade took ca. 30 minutes to be settled, with many partial fillings until someone purchased the big rest of my advertised BTCs.

And now I had fiat money on my Lykke account. I haven't yet tried transfering it with SWIFT to my bank account, but the functionality is really simply, and from what I've understood there should not arise any additional charges. In comparison to the ATMs the whole process was nearly free.
