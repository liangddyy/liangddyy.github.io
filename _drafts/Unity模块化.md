Vs

#### 生成dll

文件，新建，项目，Visual C#，类库(.NET Framework)

添加引用

```
C:\Program Files\Unity5.4.3\Editor\Data\Managed
```

```
UnityEngine.dll
UnityEditor.dll
```

项目属性，应用程序，目标框架，framework3.5 。否则会报错:

```
TypeLoadException: Could not load type 'System.Runtime.Versioning.TargetFrameworkAttribute' from assembly '...
```

GUID

参考：

http://homans.nhlrebel.com/2014/12/09/unity-4-6-and-modules/

