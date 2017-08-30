---
layout: post
title: Unity 压缩与解压缩zip包
categories: [Unity]
description: 
keywords: 开发, Unity
---

#### 压缩单个文件

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

#### 压缩文件夹

```
/// <summary>
        /// 压缩多层目录 压缩文件夹
        /// </summary>
        /// <param name="dir">The directory.</param>
        /// <param name="zipFilePath">The ziped file.</param>
        public static void ZipDir(string dir, string zipFilePath)
        {
            using (System.IO.FileStream ZipFile = System.IO.File.Create(zipFilePath))
            {
                using (ZipOutputStream s = new ZipOutputStream(ZipFile))
                {
                    ZipDirDetail(dir, s, "");
                }
            }
        }

        /// <summary>
        /// 递归遍历目录
        /// </summary>
        /// <param name="dir">The directory.</param>
        /// <param name="outStream">The ZipOutputStream Object.</param>
        /// <param name="parentPath">The parent path.</param>
        private static void ZipDirDetail(string dir, ZipOutputStream outStream, string parentPath)
        {
            if (dir[dir.Length - 1] != Path.DirectorySeparatorChar)
            {
                dir += Path.DirectorySeparatorChar;
            }
            Crc32 crc = new Crc32();
//            outStream.SetLevel(6); // 0 - store only to 9 - means best compression  
            Encoding gbk = Encoding.GetEncoding("gbk"); // 防止中文名乱码  
            ICSharpCode.SharpZipLib.Zip.ZipConstants.DefaultCodePage = gbk.CodePage;


            string[] filenames = Directory.GetFileSystemEntries(dir);

            foreach (string file in filenames) // 遍历所有的文件和目录
            {
                if (Directory.Exists(file)) // 先当作目录处理如果存在这个目录就递归Copy该目录下面的文件
                {
                    string pPath = parentPath;
                    pPath += file.Substring(file.LastIndexOf("\\") + 1);
                    pPath += "\\";
                    ZipDirDetail(file, outStream, pPath);
                }

                else // 否则直接压缩文件
                {
                    //打开压缩文件
                    using (FileStream fs = File.OpenRead(file))
                    {
                        byte[] buffer = new byte[fs.Length];
                        fs.Read(buffer, 0, buffer.Length);

                        string fileName = parentPath + file.Substring(file.LastIndexOf("\\") + 1);
                        ZipEntry entry = new ZipEntry(fileName);

                        entry.DateTime = DateTime.Now;
                        entry.Size = fs.Length;

                        fs.Close();

                        crc.Reset();
                        crc.Update(buffer);

                        entry.Crc = crc.Value;
                        outStream.PutNextEntry(entry);

                        outStream.Write(buffer, 0, buffer.Length);
                    }
                }
            }
        }
```

#### 解压 文件

```
// 解压文件
        public static bool DecompressZip(string filePath, string outFolder, string password = null)
        {
            if (string.IsNullOrEmpty(filePath) || File.Exists(filePath))
            {
                return false;
            }
            return DecompressZip(File.OpenRead(filePath), outFolder, password);
        }
```

#### 解压 byte[]

可以下载完后WWW.bytes直接解压。不用再保存成文件。

```
// 解压下载的byte[] 
        public static bool DecompressZip(byte[] ZipByte, string outFolder, string password)
        {
            return DecompressZip(new MemoryStream(ZipByte), outFolder, password);
        }
```

#### 解压 详细

```
private static bool DecompressZip(Stream stream, string outFolder, string password)
        {
            bool result = false;
            string outPath = null;
            FileStream fs = null;
            ZipInputStream zipStream = null;
            ZipEntry ent = null;
            string fileName = null;

            if (!Directory.Exists(outFolder))
            {
                Directory.CreateDirectory(outFolder);
            }
            // 防止中文名乱码  !!!
            Encoding gbk = Encoding.GetEncoding("gbk"); 
            ICSharpCode.SharpZipLib.Zip.ZipConstants.DefaultCodePage = gbk.CodePage;
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

                        if (!Directory.Exists(Path.GetDirectoryName(fileName)))
                        {
                            Directory.CreateDirectory(Path.GetDirectoryName(fileName));
                        }
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
                result = true;
            }
            catch (Exception e)
            {
                Debug.Log(e.ToString());
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
            return result;
        }
```

