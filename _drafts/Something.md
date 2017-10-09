> Access to the path "C:\_workspaceC\BabybusFrame\New Unity Project\Assets\UnoLibs\bb-product\.git\objects\pack\pack-96dba399221f53f8474e2415bbe3259fe0c1fe23.idx" is denied.

catch (UnauthorizedAccessException e)

```
FileInfo fi = new FileInfo(pathTmp);
fi.Attributes = fi.Attributes & ~FileAttributes.ReadOnly & ~FileAttributes.Hidden;
fi.Delete();
```





1. 删除隐藏文件夹（权限问题）。
2. 反射调用。

```
System.Reflection.MethodInfo api = typeof(AssetDatabase).GetMethod("ImportPackageImmediately",
                System.Reflection.BindingFlags.Static | System.Reflection.BindingFlags.NonPublic);
api.Invoke(null, new object[] {result.FilePath});
```

