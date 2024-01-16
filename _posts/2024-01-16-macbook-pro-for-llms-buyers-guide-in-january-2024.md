---
layout: post
title: Macbook Pro for LLMs - Buyer's Guide in January 2024
comments: true
tags: [Macbook Pro, Apple, Apple Silicon, Apple Metal, LLM, M1, M2, M3]
---
Given my old Macbook Pro with an Intel i7 is ageing (and quite well, I might say) I was contemplating buying a new Macbook Pro with an M2 or M3 processor. Generally speaking my needs are:

1. software development (e.g. Docker),
2. video recording (e.g. OBS Studio) and light video editing,
3. playing around with LLMs running locally mainly to learn and have fun, for more serious work I'd then switch to e.g. Google Colab or Huggingface and rent dedicated GPUs.

I am not very much worried about #1 and #2, but running LLMs? I usually tend to spend a lot of money on a Macbook Pro, but then keep it for quite a few years before making the next purchase. So, which model should I focus on?
<span class="more"></span>

The main questions for me are:

1. Is buying a Macbook Pro even a good idea for playing with LLMs?
2. Which processor?
3. How much memory?
4. How much is it worth spending?

(Note, disk space is not the main need for me, but I don't want to have less than 1 TB these days.)

Here's what I learned from doing a lot of research.

## Is buying a Macbook Pro even a good idea for playing (= inferencing) with LLMs?

If you are only occasionally running the LLM, then yes: you may consider buying a Macbook Pro. With [Llama.cpp](https://github.com/ggerganov/llama.cpp) and/or [LM Studio](https://lmstudio.ai) the model can make use of the power of the MX processors. Make sure you understand quantization of LLMs, though. Quantization refers to the process of using fewer bits per model parameter. This will both decrease the model size, but also lead to quality loss. How much exactly is a matter of ongoing research. Quantization is necessary to make larer language models accessible on consumer-grade GPUs, and without it you would require substantially higher amounts of hardware resources (particularly memory, but also more GPU cores).

If you intend to do serious work or development or run the LLM as a server in the background more continuously, then no: buying a Macbook Pro (or any laptop, for that matter) is not the best idea. LLMs drain your battery and the CPU heats up, which is not optimal for a laptop in the long-run. In this case, you probably want to consider at least an Apple Mac Studio instead, or buy a Windows PC with a modern Nvidia GPU. The Nvidia GPU will use a lot more electricity per hour, though, but you get a significantly more powerful GPU for inferencing.

And, just to be clear: Training a serious LLM from scratch is not possible on consumer-grade hardware. You need a very powerful GPU or GPU cluster. You can get those from through Google Colab or Huggingface as paid services. We are talking of inferencing here only, or possibly fine-tuning of a small model at the very best.
If you want to experiment with actual training from scratch, there are some toy examples like [TinyLLM](https://github.com/zozoheir/tinyllm) or [MiniLLM](https://github.com/kuleshov/minillm) just for learning how everything works.

## Which processor?
Obviously, bigger and newer is better, but which processor represents the most value for the money spent? There is a fantastic overview page that really breaks down the differences. Generally speaking, skip all Intel-based CPUs on Macbook Pros as well as M1, M2 or M3. Rather, aim for an M1 Pro/Max/Ultra, M2 Pro/Max/Ultra or M3 Pro/Max version. (At the time of writing there exists no M3 Ultra yet, but this is expected to be available later throughout the year in Mac Studio). This site has more or less all the info you need: [https://github.com/ggerganov/llama.cpp/discussions/4167](https://github.com/ggerganov/llama.cpp/discussions/4167)

As long as we're only talking about inferencing (and not training) then there are mainly two distinct performances to measure: input prompt processing (PP) and output token generation (TG). The two speeds may diverge, and it of course depends on your use case which is more important to you. But frequently for regular chatting with your LLM the input prompts tend to be short and the output longer. So, in case of doubt, I'd rather optimize for TG than for PP.

Here's the crucial point: *For PP and TG the most important ingredient of the processor is the processor's bandwidth, not the clock speed!* Take a moment to study the table below, it's from the page linked to above. Here's a small excerpt.

* BW = CPU bandwidth, higher is better
* CPU Cores = the GPU cores of the MX processor, higher is better
* F16, Q8, Q4 = Three different quantized models. The Q4 will be the model with the lowest number of bits per model parameter, and thus should be the fastest in terms of tokens per second (t/s) but also yield the lowest quality.
* PP = Prompt processing with number of tokens per second, higher is better
* TG = Token generation with number of tokens per second, higher is better

Effect of number of GPU cores (plus clock speeds):

||BW \[GB/s\]|CPU Cores|F16 PP \[t/s\]|F16 TG \[t/s\]|Q8\_0 PP \[t/s\]|Q8\_0 TG \[t/s\]|Q4\_0 PP \[t/s\]|Q4\_0 TG \[t/s\]|
|:-|:-|:-|:-|:-|:-|:-|:-|:-|
|M1 Max|400|32|599.53|23.03|537.37|40.2|530.06|61.19|
|M2 Max|400|38|755.67|24.65|677.91|41.83|671.31|65.95|
|M3 Max|400|40|779.17|25.09|757.64|42.75|759.76|66.31|

The main point here is: particularly the M2 Max and the M3 Max are astonishingly close to each other - yet their purchase price could easily be 1000$ apart at the time of writing. Do your own math and ask yourself if the gain in PP and TG is significant enough for you to actually spend so much more money.

Which impact does bandwidth have? Let's compare two the lower spec'd M2 Max and M3 Max with only 30 CPU Cores.

Effect of bandwidth:

||BW \[GP/s\]|CPU Cores|F16 PP \[t/s\]|F16 TG \[t/s\]|Q8\_0 PP \[t/s\]|Q8\_0 TG \[t/s\]|Q4\_0 PP \[t/s\]|Q4\_0 TG \[t/s\]|
|:-|:-|:-|:-|:-|:-|:-|:-|:-|
|M1 Max|400|32|599.53|23.03|537.37|40.2|530.06|61.19|
|M2 Max|400|30|600.46|24.16|540.15|39.97|537.6|60.99|
|M3 Max|300|30|589.41|19.54|566.43|34.3|567.59|56.58|

See the difference between M2 Max and M3 Max? The lower spec'd M3 Max with 300 GB/s bandwidth is actually *not* significantly slower/faster than the lower spec'd M2 Max with 400 GB/s - yet again, the price difference for purchasing the more modern M3 Max Macbook Pro is substantial. On the lower spec'd M2 Max and M3 Max you will end up paying a lot more for the latter without any clear gain. Why not save your money and rather rent a cloud GPU in case you ever need it? You get the point.

But that's not the entire story. Take another look and compare the M1 Max 32 cores with the M2 Max 30 cores, both with 400 GB/s of bandwidth, and the M3 Max 30 cores with 300 GB/s bandwidth. This M1 Max is really not very far from the M3 Max neither, and yet you can get it at a significant discount online if you search a little!

Is that all there is to the comparison? Should you save your money and aim for the cheapest model?

Well, consider this: If all you want is to *play occasionally* with an LLM, then the upgrade to an M3 Max might not seem worth it. However, you will of course be doing many other things with your Macbook Pro. Many other applications will run single-threaded on the CPU, and not make a lot of use of the GPU. So, the M3 family might still be worth it to power those use cases that might be more common and relevant for your *daily* work in comparison to *occasional* playing with LLMs. So, choose wisely.

## How much memory, i.e. (V)RAM?
What is the effect of having more (V)RAM on your use cases? The answer can also be found [on the same page](https://github.com/ggerganov/llama.cpp/discussions/4167#discussioncomment-7664186). Have a look at below table. The processor is always M2 Max Macbook Pro 16, 8+4 CPU with 38 cores GPU.

|(V)RAM|model|size|params|backend|ngl|test|t/s|
|:-|:-|:-|:-|:-|:-|:-|:-|
|96 GB|llama 7B mostly Q8\_0|6.67 GiB|6.74 B|Metal|99|tp 512|674.50 ± 0.58|
|96 GB|llama 7B mostly Q8\_0|6.67 GiB|6.74 B|Metal|99|tg 128|41.79 ± 0.04|
|32 GB|llama 7B mostly Q8\_0|6.67 GiB|6.74 B|Metal|99|tp 512|674.37 ± 0.63|
|32 GB|llama 7B mostly Q8\_0|6.67 GiB|6.74 B|Metal|99|tg 128|40.67 ± 0.05|

*There is no difference in inferencing speeds when running an 8-bit quantized Llama 7B model on the same Macbook Pro model with 96 GB vs 32 GB!*

[Here's what Mr Sparc writes](https://github.com/ggerganov/llama.cpp/discussions/4167#discussioncomment-7670917):

>Yes, it is expected that the same cpu/gpu spec will have similar performance values for same models to be compared regardless of RAM, as long as the size of the model to be used can be loaded into memory.The amount of RAM is a limiting factor in the size of the model that can be loaded, as only 75% (by default) of the unified memory can be used as VRAM on the GPU.

In short: The main purpose of having enough memory is to allow loading the entire quantized model into memory. Once this is done it has no further substantial effect on the inferencing speed.

This means: *Buying larger amounts of memory (e.g. 96 GB or even 128 GB) does not speed up your inferencing with LLMs, but it allows you to run a bigger LLM.*

How much memory do the different models need? [This Reddit thread](https://www.reddit.com/r/LocalLLaMA/comments/157d89h/comment/jt43x4d/?utm_source=reddit&utm_medium=web2x&context=3) lists the following file sizes (on disk) for the 16 bit unquantized models. Also, according to [this thread](https://news.ycombinator.com/item?id=37067933) running a 70B unquantized Llama2 model would require ca 160 GB of memory, and a 64 GB Macbook Pro should be able to run a 70B quantized Llama2 model.

|File size on disk|Model|
|:-|:-|
|129 GB|llama-2-70b|
|129 GB|llama-2-70b-chat|
|25 GB|llama-2-13b|
|25 GB|llama-2-13b-chat|
|13 GB|llama-2-7b|
|13 GB|llama-2-7b-chat|

As we can see the disk files are relatively large, but the quantization should decrease those figures.

There are some commands you can run to re-allocate the VRAM assigned to the GPU process on your MX processor, see e.g. [here](https://www.hardware-corner.net/increase-mac-vram/). But keep in mind also that quantization can have a significant impact on the required disk/memory size of the LLM. Once more you should ask yourself whether it's worth all that money just for playing around? You can run the largest LLMs in a rented GPU for some time, that's probably cheaper than spending all that money on the highest amounts of memory on your laptop.

Somewhere else I read that the operating system should get at least 8 GB of memory. Hence, if you go with 16 GB of memory on your Macbook Pro you unfortunately will only be able to run the smallest LLMs currently available. But already with 32 GB you should be able to do something decent, and if you are willing to spend for 48 GB or 64 GB of memory then also mid-sized LLMs will become available to you. See this discussion for some further hints: [https://github.com/ggerganov/llama.cpp/issues/13](https://github.com/ggerganov/llama.cpp/issues/13)



## How much is it worth spending?
Well, as usual, that's hard to tell. I assume that if you are someone looking into running LLMs locally then you are someone who enjoys working with tech and is willing to spend a little more on hardware. Personally, for playing with LLMs I would probably aim for an M2 Max or an M3 Max with e.g. 32 GB of memory. The lower spec'd Macbook Pro with 16'', M3 Max (30 cores), 36 GB memory and 1 TB of SSD will cost you currently 3499$ in the Apple shop. That's of course quite a bit of money, but if you aim instead at an M2 Max you won't sacrifice too much GPU speed yet get a substantially cheaper laptop in comparison. But then again, the single-core speed might be more of a concern for you. Or, if you are someone doing heavy video editing, then maybe none of what I said above is even very relevant to you. But then you also should ask yourself if you should not aim for a Mac Studio instead of a laptop. Sure, the Macbook Pro can take such loads, but the constant heavy lifting may shorten the life expectancy of your battery unnecessarily, and your fans might kick in frequently making considerable noise.

It's tempting to spend lots of money on a luxury Macbook Pro that you probably don't really need. Be clever and don't let Apple drain your wallet.