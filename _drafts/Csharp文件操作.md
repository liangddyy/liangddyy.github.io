System.IO.Directory

```
获取path目录下所有目录和文件的路径 
Directory.GetFileSystemEntries(path);
获取path目录下所有目录的路径
Directory.GetDirectories(path);

```

System.IO.Path

```
// 获取文件名（不带后缀）
// 可用来获取目录名（剔除路径）
Path.GetFileNameWithoutExtension()
```

删除文件夹1

```
DirectoryInfo dir = new DirectoryInfo(Path.Combine(UnoPkgFloderPath, model.ID));
dir.Delete(true);	// true 删除文件夹目录及其子目录
```

删除非空文件夹

```
Directory.Delete(path);
```

删除文件夹2

```
/// <summary>
/// 删除文件夹（及文件夹下所有子文件夹和文件）
/// </summary>
/// <param name="directoryPath"></param>
public static void DeleteFolder(string directoryPath)
{
    foreach (string d in Directory.GetFileSystemEntries(directoryPath))
    {
        if (File.Exists(d))
        {
            FileInfo fi = new FileInfo(d);
            if (fi.Attributes.ToString().IndexOf("ReadOnly") != -1)
                fi.Attributes = FileAttributes.Normal;
            File.Delete(d); //删除文件   
        }
        else
            DeleteFolder(d); //删除文件夹
    }
    Directory.Delete(directoryPath); //删除空文件夹
}
```

