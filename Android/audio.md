## 音频处理基础

### 采样和采样频率

未经处理的音频是一种模拟信号，而手机等设备在进行音频处理时，通常需要将模拟信号转换成数字信号，这一过程被称为A（Analog）/D（Digital）转换（反过来就是D/A转换）。A/D转换一般有专门的芯片来处理。A/D转换过程需要进行采样（也叫抽样），每秒采样次数就是采样频率。

根据[Nyquist采样定理](https://www.techtarget.com/whatis/definition/Nyquist-Theorem)，要想重建原始信号，**采样频率至少为信号中最高频率的2倍**。采样频率越高，则重建出来的信号就越接近原始信号，但同时也会增大运算处理的复杂度。一般音乐的采样频率为<font color=red>44.1kHz</font>，当然也有48kHz、96kHz甚至192kHz等更高采样频率的；而一般的语音采样频率就低得多了，主流为<font color=red>16kHz</font>，也就是所谓的高清语音。

### 采样位数

由于数字信号是用0和1来表示的，这就引出了采样位数（也称采样值、采样精度）的概念。采样位数是对**音频强度**的一种量化，决定了处理后的音频信号有多接近真实声音。一般而言，采样位数越多，能够表示出来的音频强度等级就越细致、越多，从而越能接近真实的音频强度。主流的采样位数是<font color=red>16 bit</font>，也就是大概能将声音强度表示成2<sup>16</sup>个等级，对于一般的人耳来说已经完全足够使用了，要区分$\frac{1}{2^{16} - 1}$的音强差异，恐怕不是常人所能做到的。

### 编解码

如果把采样好的原始数字信号直接保存或者发送，会占用很大的存储空间或者很大的流量，因此通常需要把采样后的数字信号进行压缩。压缩采样值以形成比特流（bitstream）的过程就称作编码（encode），还原比特流以获得采样值的过程就称作解码（decode），两个过程统称为编解码（codec）。

音频采样过程一般也叫脉冲编码调制（Pulse Code Modulation，PCM）编码，对应的采样值就是所谓的PCM值。压缩PCM值是有相应技术标准的，否则压缩出来的比特流就没法解码。目前主要有三大技术标准负责制定压缩标准：

+ ITU（International Telecommunication Union），主要制定**有线语音**的压缩标准，如[G.711](https://www.itu.int/rec/T-REC-G.711/)、[G.722](https://www.itu.int/rec/T-REC-G.722/e)以及[G.729](https://www.itu.int/rec/T-REC-G.729)等。

+ 3GPP（3rd Generation Partnership Project），主要制定**无线语音**的压缩标准，如[AMR-NB](https://wiki.multimedia.cx/index.php/AMR-NB)和[AMR-WB](https://voiceage.com/AMR-WB.G.722.2.html)。其中AMR-WB已经被ITU吸纳形成了G.722.2。

+ MPEG（Moving Picture Experts Group），主要制定**音乐制品**的压缩标准，如ISO/IEC 11172-3、ISO/IEC 13818-3以及ISO/IEC 13818-7等。

将PCM数据压缩后无任何损伤的称为**无损压缩**，反之称为**有损压缩**。前者具有高保真度，但是压缩程度低，后者则与之相反。需要注意的是，有损和无损的概念，仅仅是<font color=red>相对于原始PCM数据而言</font>，不是跟原始的音频模拟信号去做比较。因为A/D转换必然会带来信息的损失，不可能保持原样。

对音频编码前的PCM数据进行处理的过程，被称作**音频前处理**，主要用于语音中，通过去除各种干扰来使得声音更加清晰，常见的操作包括回声消除、噪声抑制以及增益控制等；对音频解码后的PCM数据进行处理的过程，就称为**音频后处理**，主要用于音乐播放中，给音乐加上各种音效，常见操作包括均衡器和混响等。

### 声道

声道是指声音在录制或播放时，于**不同空间位置**采集或回放的相互独立的音频信号。通俗来讲，声道数量就是录制时的音源数量或是播放时的扬声器数量。

声道可分为单声道（mono）和多声道。单声道通常用在语音通话中，多声道的应用范围比较广。如果多声道只有左右两个声道，那就称为立体声（stereo），更多声道就是环绕立体声。比如2.1声道、3.1声道、4声道或者5.1声道之类的概念，实际上就是不同的环绕立体声实现方案。

### 码率

码率是指每秒录制的音频资源大小，理论上码率和音频质量成正比。码率的计算公式为$$码率 = 采样频率 × 采样位数 × 声道数$$

例如，一段音频的采样频率为44.1kHz，采样位数为16 bit，采用双声道，则其未压缩的码率为$44.1\ kHz × 16\ bit × 2 = 1411.2\ kb/s = 176.4\ KiB/s$。

### 常用音频格式

目前市面上常见的音频格式有`.wav`、`.mp3`、`.flac`、`.ogg`、`.aac`、`.wma`以及`.ape`等，

#### `.wav`格式

`.wav`是微软公司开发的一种音频文件格式。它符合RIFF（Resource Interchange File Format）文件规范，用于保存Windows平台的音频信息资源，被Windows平台及其应用程序所广泛支持。该格式也支持MSADPCM等多种压缩算法，支持多种音频数字、取样频率和声道。标准格式化的`.wav`文件和CD格式一样，也是44.1kHz采样频率和16位采样，因此声音文件质量和CD相差无几，接近于PCM原始数据，也被视作无损格式。

#### `.mp3`格式

`.mp3`是一种利用MPEG Audio Layer 3的技术，将音乐以1:10甚至1:1 的压缩率压缩成较小容量文件的音频文件格式。`.mp3`的最大优势为，尽管采用了有损压缩，但却能在较好保持原有音质的基础上，把文件压缩到更小的程度。

MP3进行压缩的原理是将原始PCM数值当中对人类听觉不重要的数据（也就是12 kHz ~16 kHz的高频部分）剔除掉，从而极大减少数据量。当然，它也可以按照不同码率进行压缩，在数据大小和声音质量之间进行权衡。另外，MP3还使用了混合转换机制，将时域信号转换成频域信号。

#### `.flac`格式

`.flac`是一种利用FLAC（Free Lossless Audio Codec）技术对音频进行**高压缩率无损压缩**的音频文件格式。不同于其他有损压缩方案，FLAC不会破坏任何原有的音频信息，因此可以还原CD音质。换句话说，FLAC可以理解成无损压缩的MP3。FLAC的主要优势在于免费，并且支持大多数的操作系统，包括Windows和类Unix系统等。

#### `.ogg`格式

`.ogg`是一种采用[OGG Vorbis](https://xiph.org/vorbis/)技术的音频文件格式，类似于MP3。OGG Vorbis完全开放、免费，并且没有专利限制。就音质而言, 虽然Ogg Vorbis使用了跟MP3完全不同的数学原理，但同样码率的Ogg Vorbis和MP3文件基本上具有相同的声音质量。

#### `.aac`格式

`.aac`是一种采用AAC（Advanced Audio Coding）技术的音频文件格式，最大能容纳48通道的音轨，采样率频可达<font color=red>96 kHz</font>。AAC作为一种高压缩比的音频压缩算法，通常压缩比为18:1（也有资料说为20:1），远远超过了AC-3、MP3等较老的音频压缩算法。一般认为，`.aac`格式在96 Kbps码率的表现超过了128 Kbps的`.mp3`音频。在2000年，随着MPEG-4标准出台，AAC也将其特性进行整合，成为MPEG-4 AAC，对应的文件格式就是`.m4a`。

#### `.wma`格式

`.wma`和`.wav`一样都是微软开发的音频文件格式。`.wma`采用的技术为WMA（Windows Media Audio），和MP3一样也是有损压缩（从WMA 9.0开始支持无损压缩），但是在压缩比和音质方面都超过了MP3，在较低的采样频率下也能产生较好的音质。

#### `.ape`格式

`.ape`是一种采用APE技术的音频文件格式，和`.flac`一样都是无损压缩。APE最初由软件Monkey's audio压制得到，开发者为Matthew T. Ashland，它采用精炼的记录方式（而不是丢弃数据）来缩减体积，以确保还原后数据与源文件一样，从而保证了文件的完整性。在容量方面，APE的文件大小大概为WAV的一半，压缩率约为55%，比FLAC高。相较于`.flac`格式，`.ape`有查错能力但不提供纠错功能，从而保证文件的无损和纯正。

## 音频开发技术/库简介

### OpenSL ES

### AAudio

### Oboe

### LAME

### FFmpeg