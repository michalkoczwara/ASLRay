# ASLRay
Linux ELF x32 and x64 ASLR bypass exploit with stack-spraying

Properties:
* ASLR bypass
* Cross-platform
* Minimalistic
* Simplicity
* Unpatchable

Dependencies:
* **Linux 2.6.12+** - will work on any x86-64 Linux-based OS
	- BASH - the whole script

Limitations:
* Stack needs to be executable (-z execstack)
* Binary has to be exploited through arguments locally (not file, socket or input)
* No support for other architectures and OSes (TODO)
* Need to know the buffer limit/size

## How it works
You might have heard of [Heap Spraying](https://www.corelan.be/index.php/2011/12/31/exploit-writing-tutorial-part-11-heap-spraying-demystified/) attack? Well, [Stack Spraying](http://j00ru.vexillium.org/?p=769) is similar, however, it was considered unpractical for most cases, especially [ASLR](https://en.wikipedia.org/wiki/Address_space_layout_randomization) on x86-64.

My work will prove the opposite.

For 32-bit, there are 2^32 (4 294 967 296) theoretical addresses, nevertheless, the kernel will allow to control about only half of bits (2^(32/2) = 65 536) for an execution in a virtualized memory, which means that if we control more that 50 000 characters in stack, we are almost sure to point to our shellcode, regardless the address, thanks to kernel redirection and retranslation.

This can be achieved using shell variables, which aren't really limited to a specific length, but practical limit is about one million, otherwise it will saturate the TTY.

So, in order to exploit successfully, we need to put a [NOP sled](https://en.wikipedia.org/wiki/NOP_slide) following the shellcode into a shell variable and just exploit the binary with a random address.


In 64-bit system the situation is different, but not so much as of my discovery.

Of course, you wouldn't have to cover all 2^64 possibilities, in fact, the kernel allows only 48 bits, plus a part of them are predictable and static, which left us with about 2^(4x8+5) (137 438 953 472) possibilities.

I have mentioned the shell variables size limit, but there is also a count limit, which appears to be about 10, thus allowing us to stock a 10 000 000 character shellcode, living us with just a tenth of thousand possibilities that can be tested rapidly and automatically.

That said, ASLR on both 32 and 64-bits can be easily bypassed in few minutes and with few lines of shell...

### HowTo

If you have exploited at least one buffer overflow in your life, you can skip, but just in case:
```bash
apt install gcc || kill -9 $$
chmod u+x ASLRay.sh
sudo gcc -z execstack test.c -o test
sudo gcc -m32 -z execstack test.c -o test32
sudo chmod +s test test32
source ASLRay.sh test32 1024
source ASLRay.sh test 1024
```
Don't forget to check stack execution and ASLR both set:
```bash
scanelf -e test | grep RWX
or
readelf -l test | grep RWE
cat /proc/sys/kernel/randomize_va_space
```

#### Notes

Always rely on multiple protections and not on a single one.

We need new system security mechanisms.

> "From where we stand the rain seems random. If we could stand somewhere else, we would see the order in it. "

Tony Hillerman, *Coyote Waits*
