---
layout: post
title: Unity中wav原始音频数据处理和转换
categories: [Unity]
description: wav,音频
keywords: Unity,音频数据
---

有时候拿到一串未经处理的原始二进制音频数据时，需要在Unity中播放。需要有添加头，转换成clip 两个步骤。

#### 将未处理的原始音频二进制流转换，添加头

```
public static class WavAudioInit
{
    struct HeaderType
    {
        public byte[] riff;                 /*RIFF类资源文件头部 4byte*/
        public uint file_len;                /*文件长度4byte*/
        public byte[] wave;                  /*"WAVE"标志4byte*/
        public byte[] fmt;                   /*"fmt"标志4byte*/
        public uint NI1;                     /*过渡字节4byte*/
        public ushort format_type;           /*格式类别(10H为PCM形式的声音数据)2byte*/
        public ushort Channels;              /*Channels 1 = 单声道; 2 = 立体声2byte*/
        public uint frequency;               /*采样频率4byte*/
        public uint trans_speed;             /*音频数据传送速率4byte*/
        public ushort dataBlock;             /*数据块的调整数（按字节算的）2byte*/
        public ushort sample_bits;           /*样本的数据位数(8/16) 2byte*/
        public byte[] data;                  /*数据标记符"data" 4byte*/
        public uint wav_len;                 /*语音数据的长度 4byte*/
    }

    private static HeaderType wavHeader;        

    /// <summary>
    /// 将纯音频流信息转化成带头部参数的可以供unity使用的音频流信息
    /// </summary>
    /// <param name="data"></param>
    /// <returns></returns>
    public static byte[] ConvertToWav(byte[] data)
    {
        var databuff = new byte[data.Length + 44];
        Array.Copy(data, 0, databuff, 44, data.Length);
        return InitHeader(databuff);
    }

    /// <summary>
    /// 初始化结构体中的数组长度，分配内存
    /// </summary>
    static WavAudioInit()
    {
        wavHeader.riff = new byte[4];    //RIFF
        wavHeader.wave = new byte[4];    //WAVE
        wavHeader.fmt = new byte[4];     //fmt 
        wavHeader.data = new byte[4];    //data
    }
    
    /// <summary>
    /// 为音频文件流添加 头部 信息 使之成为可以播放的 音频流信息
    /// </summary>
    /// <returns>构建成功返回真</returns>
    private static byte[] InitHeader(byte[] databuff)
    {
        wavHeader.riff = Encoding.ASCII.GetBytes("RIFF");   /*RIFF类资源文件头部 4byte*/
        wavHeader.file_len = (uint)(databuff.Length);              /*文件长度4byte*/
        wavHeader.wave = Encoding.ASCII.GetBytes("WAVE");     /*"WAVE"标志4byte*/
        wavHeader.fmt = Encoding.ASCII.GetBytes("fmt ");      /*"fmt"标志4byte*/
        wavHeader.NI1 = 0x10;                               /*过渡字节4byte*/
        wavHeader.format_type = 0x01;                       /*格式类别(10H为PCM形式的声音数据)2byte*/
        wavHeader.Channels = 0x01;                          /*Channels 1 = 单声道; 2 = 立体声2byte*/
        wavHeader.frequency = 0x3E80;                       /*采样频率4byte 16000 */
        wavHeader.trans_speed = 0x7d00;                     /*音频数据传送速率4byte 32000 */
        wavHeader.dataBlock = 0x02;                         /*数据块的调整数（按字节算的）2byte*/
        wavHeader.sample_bits = 0x10;                       /*样本的数据位数(8/16) 2byte*/
        wavHeader.data = Encoding.ASCII.GetBytes("data");   /*数据标记符"data" 4byte*/
        wavHeader.wav_len = (uint)(databuff.Length - 44);                /*语音数据的长度 4byte*/
        byte[] byt2;//临时变量 ，保存2位的整数
        byte[] byt4;//临时变量， 保存4位的整数
        Encoding.ASCII.GetBytes(Encoding.ASCII.GetString(wavHeader.riff), 0, 4, databuff, 0);/*RIFF类资源文件头部 4byte*/
        byt4 = BitConverter.GetBytes(wavHeader.file_len); /*文件长度4byte*/
        Array.Copy(byt4, 0, databuff, 4, 4);
        Encoding.ASCII.GetBytes(Encoding.ASCII.GetString(wavHeader.wave), 0, 4, databuff, 8);/*"WAVE"标志4byte*/
        Encoding.ASCII.GetBytes(Encoding.ASCII.GetString(wavHeader.fmt), 0, 4, databuff, 12);/*"fmt"标志4byte*/
        byt4 = BitConverter.GetBytes(wavHeader.NI1);/*过渡字节4byte*/
        Array.Copy(byt4, 0, databuff, 16, 4);
        byt2 = BitConverter.GetBytes(wavHeader.format_type);/*格式类别(10H为PCM形式的声音数据)2byte*/
        Array.Copy(byt2, 0, databuff, 20, 2);
        byt2 = BitConverter.GetBytes(wavHeader.Channels);/*Channels 1 = 单声道; 2 = 立体声2byte*/
        Array.Copy(byt2, 0, databuff, 22, 2);
        byt4 = BitConverter.GetBytes(wavHeader.frequency);/*采样频率4byte*/
        Array.Copy(byt4, 0, databuff, 24, 4);
        byt4 = BitConverter.GetBytes(wavHeader.trans_speed);/*音频数据传送速率4byte*/
        Array.Copy(byt4, 0, databuff, 28, 4);
        byt2 = BitConverter.GetBytes(wavHeader.dataBlock);/*数据块的调整数（按字节算的）2byte*/
        Array.Copy(byt2, 0, databuff, 32, 2);
        byt2 = BitConverter.GetBytes(wavHeader.sample_bits);/*样本的数据位数(8/16) 2byte*/
        Array.Copy(byt2, 0, databuff, 34, 2);
        Encoding.ASCII.GetBytes(Encoding.ASCII.GetString(wavHeader.data), 0, 4, databuff, 36);/*数据标记符"data" 4byte*/
        byt4 = BitConverter.GetBytes(wavHeader.wav_len); /*语音数据的长度 4byte*/
        Array.Copy(byt4, 0, databuff, 40, 4);
        return databuff;
    }
    
    private static byte[] buff = new byte[44];
    
    /// <summary>
    /// 读取WAVE文件，包括文件头和数据部分
    /// 用于测试 读取正确音频文件 查看头文件参数
    /// </summary>
    /// <param name="filepath">文件路径</param>
    /// <returns>读取成功返回真</returns>
    public static bool readWav(string filepath)
    {
        String fileName = filepath;//保存文件名
        if (File.Exists(fileName) == false)//文件不存在
        {
            throw new Exception("1");
        }
        //只读方式打开文件
        FileStream file = new FileStream(fileName, FileMode.Open, FileAccess.Read, FileShare.None);
        if (file == null || file.Length < 44) //长度少于44，或者打开失败
        {
            file.Close();//
            throw new Exception("s2");
        }
        file.Read(buff, 0, 44);//读取文件头
        if (fixedData(buff) == false)//按位置保存文件头信息到结构体中
            throw new Exception("3");
            file.Close();//关闭文件
        return false;
    }
    
    /// <summary>
    /// 把文件头数组信息保存到结构体中
    /// </summary
    /// <param name="pbuff">文件头数组</param>
    /// <returns>保存成功返回真</returns>
    static bool fixedData(byte[] pbuff)
    {
        Array.Copy(pbuff, 0, wavHeader.riff, 0, 4);//
        if (Encoding.ASCII.GetString(wavHeader.riff) != "RIFF")//确定文件是WAVA类型
            return false;
        wavHeader.file_len = BitConverter.ToUInt32(pbuff, 4);
        Array.Copy(pbuff, 8, wavHeader.wave, 0, 4);
        Array.Copy(pbuff, 12, wavHeader.fmt, 0, 4);
        wavHeader.NI1 = BitConverter.ToUInt32(pbuff, 16);
        wavHeader.format_type = BitConverter.ToUInt16(pbuff, 20);
        wavHeader.Channels = BitConverter.ToUInt16(pbuff, 22);
        wavHeader.frequency = BitConverter.ToUInt32(pbuff, 24);
        wavHeader.trans_speed = BitConverter.ToUInt32(pbuff, 28);
        wavHeader.dataBlock = BitConverter.ToUInt16(pbuff, 32);
        wavHeader.sample_bits = BitConverter.ToUInt16(pbuff, 34);
        Array.Copy(pbuff, 36, wavHeader.data, 0, 4);
        wavHeader.wav_len = BitConverter.ToUInt32(pbuff, 40);
        Debug.Log("riff: " + wavHeader.riff);
        Debug.Log("file_len: " + wavHeader.file_len);
        Debug.Log("NI1: " + wavHeader.NI1);
        Debug.Log("format_type: " + wavHeader.format_type);
        Debug.Log("channels: " + wavHeader.Channels);
        Debug.Log("frequency: " + wavHeader.frequency);
        Debug.Log("trans_speed: " + wavHeader.trans_speed);
        Debug.Log("dataBlock: " + wavHeader.dataBlock);
        Debug.Log("sample_bits: " + wavHeader.sample_bits);
        Debug.Log("wav_len: " + wavHeader.wav_len);
        return true;
    }
}
```

#### 将带头结点的二进制数据转换成Unity可以使用的Clip

```
public class WavUtility
{
    // Force save as 16-bit .wav
    const int BlockSize_16Bit = 2;

    /// <summary>
    /// Load PCM format *.wav audio file (using Unity's Application data path) and convert to AudioClip.
    /// </summary>
    /// <returns>The AudioClip.</returns>
    /// <param name="filePath">Local file path to .wav file</param>
    public static AudioClip ToAudioClip(string filePath)
    {
        if (!filePath.StartsWith(Application.persistentDataPath) && !filePath.StartsWith(Application.dataPath))
        {
            Debug.LogWarning(
                "This only supports files that are stored using Unity's Application data path. \nTo load bundled resources use 'Resources.Load(\"filename\") typeof(AudioClip)' method. \nhttps://docs.unity3d.com/ScriptReference/Resources.Load.html");
            return null;
        }

        byte[] fileBytes = File.ReadAllBytes(filePath);
        return ToAudioClip(fileBytes, 0);
    }
    
    /// <summary>
    /// 将带头节点参数的音频信息转化成unity 的 clip
    /// </summary>
    private static AudioClip ToAudioClip(byte[] fileBytes, int offsetSamples = 0, string name = "wav")
    {
        //string riff = Encoding.ASCII.GetString (fileBytes, 0, 4);
        //string wave = Encoding.ASCII.GetString (fileBytes, 8, 4);
        int subchunk1 = BitConverter.ToInt32(fileBytes, 16);
        UInt16 audioFormat = BitConverter.ToUInt16(fileBytes, 20);

        // NB: Only uncompressed PCM wav files are supported.
        string formatCode = FormatCode(audioFormat);
        Debug.AssertFormat(audioFormat == 1 || audioFormat == 65534,
            "Detected format code '{0}' {1}, but only PCM and WaveFormatExtensable uncompressed formats are currently supported.",
            audioFormat, formatCode);

        UInt16 channels = BitConverter.ToUInt16(fileBytes, 22);
        int sampleRate = BitConverter.ToInt32(fileBytes, 24);
        //int byteRate = BitConverter.ToInt32 (fileBytes, 28);
        //UInt16 blockAlign = BitConverter.ToUInt16 (fileBytes, 32);
        UInt16 bitDepth = BitConverter.ToUInt16(fileBytes, 34);

        int headerOffset = 16 + 4 + subchunk1 + 4;
        int subchunk2 = BitConverter.ToInt32(fileBytes, headerOffset);
        //Debug.LogFormat ("riff={0} wave={1} subchunk1={2} format={3} channels={4} sampleRate={5} byteRate={6} blockAlign={7} bitDepth={8} headerOffset={9} subchunk2={10} filesize={11}", riff, wave, subchunk1, formatCode, channels, sampleRate, byteRate, blockAlign, bitDepth, headerOffset, subchunk2, fileBytes.Length);

        float[] data;
        switch (bitDepth)
        {
            case 8:
                data = Convert8BitByteArrayToAudioClipData(fileBytes, headerOffset, subchunk2);
                break;
            case 16:
                data = Convert16BitByteArrayToAudioClipData(fileBytes, headerOffset, subchunk2);
                break;
            case 24:
                data = Convert24BitByteArrayToAudioClipData(fileBytes, headerOffset, subchunk2);
                break;
            case 32:
                data = Convert32BitByteArrayToAudioClipData(fileBytes, headerOffset, subchunk2);
                break;
            default:
                throw new Exception(bitDepth + " bit depth is not supported.");
        }

        AudioClip audioClip = AudioClip.Create(name, data.Length, (int) channels, sampleRate, false);
        audioClip.SetData(data, 0);
        return audioClip;
    }

    #region wav file bytes to Unity AudioClip conversion methods

    private static float[] Convert8BitByteArrayToAudioClipData(byte[] source, int headerOffset, int dataSize)
    {
        int wavSize = BitConverter.ToInt32(source, headerOffset);
        headerOffset += sizeof(int);
        Debug.AssertFormat(wavSize > 0 && wavSize == dataSize,
            "Failed to get valid 8-bit wav size: {0} from data bytes: {1} at offset: {2}", wavSize, dataSize,
            headerOffset);

        float[] data = new float[wavSize];

        sbyte maxValue = sbyte.MaxValue;

        int i = 0;
        while (i < wavSize)
        {
            data[i] = (float) source[i] / maxValue;
            ++i;
        }

        return data;
    }

    private static float[] Convert16BitByteArrayToAudioClipData(byte[] source, int headerOffset, int dataSize)
    {
        int wavSize = BitConverter.ToInt32(source, headerOffset);
        headerOffset += sizeof(int);
        Debug.AssertFormat(wavSize > 0 && wavSize == dataSize,
            "Failed to get valid 16-bit wav size: {0} from data bytes: {1} at offset: {2}", wavSize, dataSize,
            headerOffset);

        int x = sizeof(Int16); // block size = 2
        int convertedSize = wavSize / x;

        float[] data = new float[convertedSize];

        Int16 maxValue = Int16.MaxValue;

        int offset = 0;
        int i = 0;
        while (i < convertedSize)
        {
            offset = i * x + headerOffset;
            data[i] = (float) BitConverter.ToInt16(source, offset) / maxValue;
            ++i;
        }

        Debug.AssertFormat(data.Length == convertedSize, "AudioClip .wav data is wrong size: {0} == {1}",
            data.Length, convertedSize);

        return data;
    }

    private static float[] Convert24BitByteArrayToAudioClipData(byte[] source, int headerOffset, int dataSize)
    {
        int wavSize = BitConverter.ToInt32(source, headerOffset);
        headerOffset += sizeof(int);
        Debug.AssertFormat(wavSize > 0 && wavSize == dataSize,
            "Failed to get valid 24-bit wav size: {0} from data bytes: {1} at offset: {2}", wavSize, dataSize,
            headerOffset);

        int x = 3; // block size = 3
        int convertedSize = wavSize / x;

        int maxValue = Int32.MaxValue;

        float[] data = new float[convertedSize];

        byte[]
            block = new byte[sizeof(int)]; // using a 4 byte block for copying 3 bytes, then copy bytes with 1 offset

        int offset = 0;
        int i = 0;
        while (i < convertedSize)
        {
            offset = i * x + headerOffset;
            Buffer.BlockCopy(source, offset, block, 1, x);
            data[i] = (float) BitConverter.ToInt32(block, 0) / maxValue;
            ++i;
        }

        Debug.AssertFormat(data.Length == convertedSize, "AudioClip .wav data is wrong size: {0} == {1}",
            data.Length, convertedSize);

        return data;
    }

    private static float[] Convert32BitByteArrayToAudioClipData(byte[] source, int headerOffset, int dataSize)
    {
        int wavSize = BitConverter.ToInt32(source, headerOffset);
        headerOffset += sizeof(int);
        Debug.AssertFormat(wavSize > 0 && wavSize == dataSize,
            "Failed to get valid 32-bit wav size: {0} from data bytes: {1} at offset: {2}", wavSize, dataSize,
            headerOffset);

        int x = sizeof(float); //  block size = 4
        int convertedSize = wavSize / x;

        Int32 maxValue = Int32.MaxValue;

        float[] data = new float[convertedSize];

        int offset = 0;
        int i = 0;
        while (i < convertedSize)
        {
            offset = i * x + headerOffset;
            data[i] = (float) BitConverter.ToInt32(source, offset) / maxValue;
            ++i;
        }

        Debug.AssertFormat(data.Length == convertedSize, "AudioClip .wav data is wrong size: {0} == {1}",
            data.Length, convertedSize);

        return data;
    }

    #endregion

    public static byte[] FromAudioClip(AudioClip audioClip)
    {
        string file;
        return FromAudioClip(audioClip, out file, false);
    }

    public static byte[] FromAudioClip(AudioClip audioClip, out string filepath, bool saveAsFile = true,
        string dirname = "recordings")
    {
        MemoryStream stream = new MemoryStream();

        const int headerSize = 44;

        // get bit depth
        UInt16 bitDepth = 16; //BitDepth (audioClip);

        // NB: Only supports 16 bit
        //Debug.AssertFormat (bitDepth == 16, "Only converting 16 bit is currently supported. The audio clip data is {0} bit.", bitDepth);

        // total file size = 44 bytes for header format and audioClip.samples * factor due to float to Int16 / sbyte conversion
        int fileSize = audioClip.samples * BlockSize_16Bit + headerSize; // BlockSize (bitDepth)

        // chunk descriptor (riff)
        WriteFileHeader(ref stream, fileSize);
        // file header (fmt)
        WriteFileFormat(ref stream, audioClip.channels, audioClip.frequency, bitDepth);
        // data chunks (data)
        WriteFileData(ref stream, audioClip, bitDepth);

        byte[] bytes = stream.ToArray();

        // Validate total bytes
        Debug.AssertFormat(bytes.Length == fileSize, "Unexpected AudioClip to wav format byte count: {0} == {1}",
            bytes.Length, fileSize);

        // Save file to persistant storage location
        if (saveAsFile)
        {
            filepath = string.Format("{0}/{1}/{2}.{3}", Application.streamingAssetsPath, dirname,
                DateTime.UtcNow.ToString("yyMMdd-HHmmss-fff"), "wav");
            Directory.CreateDirectory(Path.GetDirectoryName(filepath));
            File.WriteAllBytes(filepath, bytes);
            Debug.Log("Auto-saved .wav file: " + filepath);
        }
        else
        {
            filepath = null;
        }

        stream.Dispose();

        return bytes;
    }

    #region write .wav file functions

    private static int WriteFileHeader(ref MemoryStream stream, int fileSize)
    {
        int count = 0;
        int total = 12;

        // riff chunk id
        byte[] riff = Encoding.ASCII.GetBytes("RIFF");
        count += WriteBytesToMemoryStream(ref stream, riff, "ID");

        // riff chunk size
        int chunkSize = fileSize - 8; // total size - 8 for the other two fields in the header
        count += WriteBytesToMemoryStream(ref stream, BitConverter.GetBytes(chunkSize), "CHUNK_SIZE");

        byte[] wave = Encoding.ASCII.GetBytes("WAVE");
        count += WriteBytesToMemoryStream(ref stream, wave, "FORMAT");

        // Validate header
        Debug.AssertFormat(count == total, "Unexpected wav descriptor byte count: {0} == {1}", count, total);

        return count;
    }

    private static int WriteFileFormat(ref MemoryStream stream, int channels, int sampleRate, UInt16 bitDepth)
    {
        int count = 0;
        int total = 24;

        byte[] id = Encoding.ASCII.GetBytes("fmt ");
        count += WriteBytesToMemoryStream(ref stream, id, "FMT_ID");

        int subchunk1Size = 16; // 24 - 8
        count += WriteBytesToMemoryStream(ref stream, BitConverter.GetBytes(subchunk1Size), "SUBCHUNK_SIZE");

        UInt16 audioFormat = 1;
        count += WriteBytesToMemoryStream(ref stream, BitConverter.GetBytes(audioFormat), "AUDIO_FORMAT");

        UInt16 numChannels = Convert.ToUInt16(channels);
        count += WriteBytesToMemoryStream(ref stream, BitConverter.GetBytes(numChannels), "CHANNELS");

        count += WriteBytesToMemoryStream(ref stream, BitConverter.GetBytes(sampleRate), "SAMPLE_RATE");

        int byteRate = sampleRate * channels * BytesPerSample(bitDepth);
        count += WriteBytesToMemoryStream(ref stream, BitConverter.GetBytes(byteRate), "BYTE_RATE");

        UInt16 blockAlign = Convert.ToUInt16(channels * BytesPerSample(bitDepth));
        count += WriteBytesToMemoryStream(ref stream, BitConverter.GetBytes(blockAlign), "BLOCK_ALIGN");

        count += WriteBytesToMemoryStream(ref stream, BitConverter.GetBytes(bitDepth), "BITS_PER_SAMPLE");

        // Validate format
        Debug.AssertFormat(count == total, "Unexpected wav fmt byte count: {0} == {1}", count, total);

        return count;
    }

    private static int WriteFileData(ref MemoryStream stream, AudioClip audioClip, UInt16 bitDepth)
    {
        int count = 0;
        int total = 8;

        // Copy float[] data from AudioClip
        float[] data = new float[audioClip.samples * audioClip.channels];
        audioClip.GetData(data, 0);

        byte[] bytes = ConvertAudioClipDataToInt16ByteArray(data);

        byte[] id = Encoding.ASCII.GetBytes("data");
        count += WriteBytesToMemoryStream(ref stream, id, "DATA_ID");

        int subchunk2Size = Convert.ToInt32(audioClip.samples * BlockSize_16Bit); // BlockSize (bitDepth)
        count += WriteBytesToMemoryStream(ref stream, BitConverter.GetBytes(subchunk2Size), "SAMPLES");

        // Validate header
        Debug.AssertFormat(count == total, "Unexpected wav data id byte count: {0} == {1}", count, total);

        // Write bytes to stream
        count += WriteBytesToMemoryStream(ref stream, bytes, "DATA");

        // Validate audio data
        Debug.AssertFormat(bytes.Length == subchunk2Size, "Unexpected AudioClip to wav subchunk2 size: {0} == {1}",
            bytes.Length, subchunk2Size);

        return count;
    }

    private static byte[] ConvertAudioClipDataToInt16ByteArray(float[] data)
    {
        MemoryStream dataStream = new MemoryStream();

        int x = sizeof(Int16);

        Int16 maxValue = Int16.MaxValue;

        int i = 0;
        while (i < data.Length)
        {
            dataStream.Write(BitConverter.GetBytes(Convert.ToInt16(data[i] * maxValue)), 0, x);
            ++i;
        }

        byte[] bytes = dataStream.ToArray();

        // Validate converted bytes
        Debug.AssertFormat(data.Length * x == bytes.Length,
            "Unexpected float[] to Int16 to byte[] size: {0} == {1}", data.Length * x, bytes.Length);

        dataStream.Dispose();

        return bytes;
    }

    private static int WriteBytesToMemoryStream(ref MemoryStream stream, byte[] bytes, string tag = "")
    {
        int count = bytes.Length;
        stream.Write(bytes, 0, count);
        //Debug.LogFormat ("WAV:{0} wrote {1} bytes.", tag, count);
        return count;
    }

    #endregion

    /// <summary>
    /// Calculates the bit depth of an AudioClip
    /// </summary>
    /// <returns>The bit depth. Should be 8 or 16 or 32 bit.</returns>
    /// <param name="audioClip">Audio clip.</param>
    public static UInt16 BitDepth(AudioClip audioClip)
    {
        UInt16 bitDepth =
            Convert.ToUInt16(audioClip.samples * audioClip.channels * audioClip.length / audioClip.frequency);
        Debug.AssertFormat(bitDepth == 8 || bitDepth == 16 || bitDepth == 32,
            "Unexpected AudioClip bit depth: {0}. Expected 8 or 16 or 32 bit.", bitDepth);
        return bitDepth;
    }

    private static int BytesPerSample(UInt16 bitDepth)
    {
        return bitDepth / 8;
    }

    private static int BlockSize(UInt16 bitDepth)
    {
        switch (bitDepth)
        {
            case 32:
                return sizeof(Int32); // 32-bit -> 4 bytes (Int32)
            case 16:
                return sizeof(Int16); // 16-bit -> 2 bytes (Int16)
            case 8:
                return sizeof(sbyte); // 8-bit -> 1 byte (sbyte)
            default:
                throw new Exception(bitDepth + " bit depth is not supported.");
        }
    }

    private static string FormatCode(UInt16 code)
    {
        switch (code)
        {
            case 1:
                return "PCM";
            case 2:
                return "ADPCM";
            case 3:
                return "IEEE";
            case 7:
                return "μ-law";
            case 65534:
                return "WaveFormatExtensable";
            default:
                Debug.LogWarning("Unknown wav code format:" + code);
                return "";
        }
    }
}
```

#### 将百度语音识别返回的原始音频数据、或其他未处理的原始音频 转换成Clip

```
    public static AudioClip ByteToAudioClip(byte[] data, int offsetSamples = 0, string name = "wav")
    {
        if (data == null)
            return null;
        var fileBytes = WavAudioInit.ConvertToWav(data);
        return ToAudioClip(fileBytes);
    }
```

> 参考：[WAV格式说明 Creating a WAV (RIFF) file](<http://www.topherlee.com/software/pcm-tut-wavformat.html>)
