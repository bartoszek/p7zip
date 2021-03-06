
# A new p7zip fork with additional codecs and improvements (forked from https://sourceforge.net/projects/p7zip/).

This is the place for the active development of p7zip to include major modern codecs such as Brotli, Fast LZMA2, LZ4, LZ5, Lizard and Zstd. In order to support multithreading for those addional codecs, this project depends on the [Multithreading Library](https://github.com/mcmilk/zstdmt).

# status
[![ubuntu-badge]][ubuntu-link]

[ubuntu-link]: https://github.com/szcnick/p7zip/actions?query=workflow%3AUbuntu "UbuntuL"
[ubuntu-badge]: https://github.com/szcnick/p7zip/workflows/Ubuntu/badge.svg "UbuntuB"

## Codec overview
1. [Zstandard] v1.4.5 is a real-time compression algorithm, providing high compression ratios. It offers a very wide range of compression / speed trade-off, while being backed by a very fast decoder.
   - Levels: 1..19

2. [LZ4] v1.9.3 is lossless compression algorithm, providing compression speed at 400 MB/s per core (0.16 Bytes/cycle). It features an extremely fast decoder, with speed in multiple GB/s per core (0.71 Bytes/cycle). A high compression derivative, called LZ4_HC, is available, trading customizable CPU time for compression ratio.
   - Levels: 1..12

3. [Fast LZMA2] v1.0.1 is a LZMA2 compression algorithm, 20% to 100% faster than normal LZMA2 at levels 5 and above, but with a slightly lower compression ratio. It uses a parallel buffered radix matchfinder and some optimizations from Zstandard. The codec uses much less additional memory per thread than standard LZMA2.
   - Levels: 1..9

4. [Brotli] v1.0.9 is a generic-purpose lossless compression algorithm that compresses data using a combination of a modern variant of the LZ77 algorithm, Huffman coding and 2nd order context modeling, with a compression ratio comparable to the best currently available general-purpose compression methods. It is similar in speed with deflate but offers more dense compression.
   - Levels: 0..11

5. [LZ5] v1.5 is a modification of LZ4 which gives a better ratio at cost of slower compression and decompression.
   - Levels: 1..15

6. [Lizard] v1.0 is an efficient compressor with very fast decompression. It achieves compression ratio that is comparable to zip/zlib and zstd/brotli (at low and medium compression levels) at decompression speed of 1000 MB/s and faster.
   - Levels 10..19 (fastLZ4) are designed to give about 10% better decompression speed than LZ4
   - Levels 20..29 (LIZv1) are designed to give better ratio than LZ4 keeping 75% decompression speed
   - Levels 30..39 (fastLZ4 + Huffman) adds Huffman coding to fastLZ4
   - Levels 40..49 (LIZv1 + Huffman) give the best ratio, comparable to zlib and low levels of zstd/brotli, but with a faster decompression speed
   
## Benchmark
We use [silesia](http://sun.aei.polsl.pl/~sdeor/index.php?page=silesia) files(total size 211938580 Byte) for packaging. 
   CPU frequency：2.20GHz
|format|method|encode_size(Byte)|encode_time(ms)|encode_speed(M/s)|decode_time(ms)|decode_speed(M/s)|compression_ratio|
|:--:|:--:|:--:|:--:|:--:|:--:|:--:|:--:|
|7z-zip|deflaut-9|64727443|59170 ms|3.33|1649|122.57|3.27|
|7z-7z|lzma-9|48752521|80680 ms|2.51|3780|53.47|4.35|
|7z-7z|lzma2-9|48749147|81106 ms|2.50|3742|54.01|4.35|
|7z-7z|zstd-22|52739879|134245 ms|1.51|893|226.34|4.02|
|7z-7z|lz4-12|77788386|3548 ms|56.97|409|494.18|2.72|
|7z-7z|flzma2-9|48676481|12619 ms|16.02|3877|52.13|4.35|

### Install CLI
#### (Currently only supports CLI, if you want to do GUI please contact us)
1. git clone https://github.com/szcnick/p7zip.git
2. cd p7zip && make 7z . (OR make 7za)
3. ./bin/7z i . (OR 7za i)

The output should look like this:
```
7-Zip (a) [64] 17.02 : Copyright (c) 1999-2020 Igor Pavlov 
p7zip Version 17.02 (locale=zh_CN.UTF-8,Utf16=on,HugeFiles=on,64 bits,12 CPUs x64)
 
Formats:
...
 0 CK            xz       xz txz (.tar) FD 7 z X Z 00
 0               Z        z taz (.tar)  1F 9D
 0 CK            zstd     zst tzstd (.tar) 0 x F D 2 F B 5 2 5 . . 0 x F D 2 F B 5 2 8 00
 0 C   F         7z       7z            7 z BC AF ' 1C
 0     F         Cab      cab           M S C F 00 00 00 00
...

Codecs:
 0 4ED  303011B BCJ2
 0  ED  3030103 BCJ
 0  ED  3030205 PPC
 0  ED  3030401 IA64
 0  ED  3030501 ARM
 0  ED  3030701 ARMT
 0  ED  3030805 SPARC
 0  ED    20302 Swap2
 0  ED    20304 Swap4
 0  ED    40202 BZip2
 0  ED        0 Copy
 0  ED    40109 Deflate64
 0  ED    40108 Deflate
 0  ED        3 Delta
 0  ED       21 LZMA2
 0  ED    30101 LZMA
 0  ED    30401 PPMD
 0   D    40301 Rar1
 0   D    40302 Rar2
 0   D    40303 Rar3
 0   D    40305 Rar5
 0  ED  4F71104 LZ4
 0  ED  4F71101 ZSTD
 0  ED  6F10701 7zAES
 0  ED  6F00181 AES256CBC
 1 3ED  4F712FF RawSplitter

Hashers:
 0    4        1 CRC32
 0   20      201 SHA1
 0   32        A SHA256
 0    8        4 CRC64
 0   32      202 BLAKE2sp
```

### Usage (codec plugin)

- compression and decompression for [LZ4] [Fast LZMA2] [Brotli] [LZ5] [Lizard] and [Zstandard] within the p7-Zip container format
- you can only create `.7z` files, the files like `.lz4` and `.zst` are not covered by the plugins
- when compressing binaries (*.exe, *.dll), you have to explicitly disable the bcj2 filter via `-m0=bcj`.
- so the usage should look like this:
```
7z a archiv.7z -m0=bcj -m1=zstd -mx1   Fast mode, with BCJ preprocessor on executables
7z a archiv.7z -m0=bcj -m1=zstd -mx..  ...
7z a archiv.7z -m0=bcj -m1=zstd -mx21  2nd Slowest Mode, with BCJ preprocessor on executables
7z a archiv.7z -m0=bcj -m1=zstd -mx22  Ultra Mode, with BCJ preprocessor on executables
7z a archiv.7z -m0=bcj -m1=lz4 -mxN  ...
```

## License and redistribution

- the same as the Mainline [7-Zip], which means GNU LGPL

## Links

- [7-Zip Homepage](https://www.7-zip.org/)

## Version Information

- p7zip Version 17.02
  - [LZ4] Version 1.9.2
  - [Zstandard] Version 1.4.5
  - [Fast LZMA2] Version v1.0.1
  - [Brotli] Version v1.0.9
  - [LZ5] Version v1.5
  - [Lizard] Version 1.0
  
## Working Plan
 - [check here](https://github.com/szcnick/p7zip/tree/dev/DOC)

[7-Zip]:https://www.7-zip.org/
[LZ4]:https://github.com/lz4/lz4/
[Zstandard]:https://github.com/facebook/zstd/
[Fast LZMA2]:https://github.com/conor42/fast-lzma2
[Brotli]:https://github.com/google/brotli/
[LZ5]:https://github.com/inikep/lz5/
[Lizard]:https://github.com/inikep/lizard/
