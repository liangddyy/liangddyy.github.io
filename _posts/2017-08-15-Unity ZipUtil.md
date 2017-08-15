---
layout: post
title: Unity 压缩与解压缩zip包
categories: [Unity]
description: 
keywords: 开发, Unity
---

压缩

```
// 压缩文件
        public static string CompressZip(string filePath, string outFolder, string password = null)
        {
            if (!File.Exists(filePath))
            {
                UnityEngine.Debug.LogError("需要压缩的文件不存在");
                return null;
            }

            string zipFileName = outFolder + Path.GetFileNameWithoutExtension(filePath) + ".zip";

            using (FileStream fs = File.Create(zipFileName))
            {
                using (ZipOutputStream zipStream = new ZipOutputStream(fs))
                {
                    zipStream.Password = password;
                    using (FileStream stream = new FileStream(filePath, FileMode.Open, FileAccess.Read))
                    {
                        string fileName = Path.GetFileName(filePath);
                        ZipEntry zipEntry = new ZipEntry(fileName);
                        zipStream.PutNextEntry(zipEntry);
                        byte[] buffer = new byte[1024];
                        int sizeRead = 0;
                        try
                        {
                            do
                            {
                                sizeRead = stream.Read(buffer, 0, buffer.Length);
                                zipStream.Write(buffer, 0, sizeRead);
                            } while (sizeRead > 0);
                        }
                        catch (Exception e)
                        {
                            Debug.LogException(e);
                        }
                        stream.Close();
                    }
                    zipStream.Finish();
                    zipStream.Close();
                }
                fs.Close();
                return zipFileName;
            }
        }
```

解压文件

```
        // 解压文件
        public static string DecompressZip(string filePath, string outFolder, string password = null)
        {
            if (string.IsNullOrEmpty(filePath) || File.Exists(filePath))
            {
                return null;
            }
            return DecompressZip(outFolder, File.OpenRead(filePath), password);
        }
```

解压byte[]，可以下载完后WWW.bytes直接解压。不用再保存成文件。

```
        // 解压下载的byte[] 
        public static string DecompressZip(string outFolder, byte[] ZipByte, string password)
        {
            return DecompressZip(outFolder, new MemoryStream(ZipByte), password);
        }
```

解压部分 详细

```
private static string DecompressZip(string outFolder, Stream stream, string password)
        {
            string outPath = null;
            FileStream fs = null;
            ZipInputStream zipStream = null;
            ZipEntry ent = null;
            string fileName = null;

            outFolder = Application.persistentDataPath + "/" + outFolder;

            if (!Directory.Exists(outFolder))
            {
                Directory.CreateDirectory(outFolder);
            }

            try
            {
                zipStream = new ZipInputStream(stream);

                if (!string.IsNullOrEmpty(password))
                {
                    Debug.Log("use password");
                    zipStream.Password = password;
                }

                while ((ent = zipStream.GetNextEntry()) != null)
                {
                    if (!string.IsNullOrEmpty(ent.Name))
                    {
                        fileName = Path.Combine(outFolder, ent.Name);

                        #region Android
                        fileName = fileName.Replace('\\', '/');

                        if (fileName.EndsWith("/"))
                        {
                            Directory.CreateDirectory(fileName);
                            continue;
                        }
                        #endregion

                        fs = File.Create(fileName);

                        int size = 2048;
                        byte[] data = new byte[size];


                        while (true)
                        {
                            size = zipStream.Read(data, 0, data.Length);

                            if (size > 0)
                            {
                                fs.Write(data, 0, data.Length);
                            }
                            else
                                break;
                        }
                    }
                }
                outPath = fileName;
            }
            catch (Exception e)
            {
                Debug.Log(e.ToString());
                outPath = null;
            }
            finally
            {
                if (fs != null)
                {
                    fs.Close();
                    fs.Dispose();
                }
                if (zipStream != null)
                {
                    zipStream.Close();
                    zipStream.Dispose();
                }
                if (ent != null)
                {
                    ent = null;
                }
                GC.Collect();
                GC.Collect(1);
            }
            return outPath;
        }
```

