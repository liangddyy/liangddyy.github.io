---
layout: post
title: "Unity Sqlite的使用与简单封装"
keywords: 
description: ""
category: Unity
tags: Sqlite
---

<!--markdown-->并不是要介绍网上一搜一大片的Sqlite在Unity中的用法，而是想介绍下不一样的封装。另外对Unity和C#中的反射也并不熟练，所以难免疏漏。  

Unity中使用Sqlite  
  
## 所需文件  
  
```  
Mono.Data.Sqlite.dll  
Mono.Data.dll
Sqlite3.dll       （32位/64位）  
```  

前两个文件在 `\Unity\Editor\Data\MonoBleedingEdge\lib\mono\2.0` 这个目下可以找到  
  
后一个文件Sqlite 可以在这儿下载 [https://sourceforge.net/projects/sqlitebrowser/][1] 	需要注意下载对应的32位或者64位  
  
将上面的三个文件拷贝至文件夹 Asset\Plugins 下。如果是用在Android上，还需要一个 libsqlite3.so 文件，放在 `Asset\Plugins\Android\ ` 下，文件可以在下面的链接的文章中下载到。[http://blog.csdn.net/qinyuanpei/article/details/46812655#t1][2] 
  
不知道为啥编辑器没有自动添加引用，所以使用时记得添加下面的几个引用。  
  
```  
：unity`SqliteConnection' could not be found.  
using Mono.Data.Sqlite; 
  
：The type ornamespace name `Exception' could not be found.  
using System;  
  
：The type ornamespace name `SqliteDataReader' could not be found.  
using System.IO;  
```  
  
## 简单使用  
  
如果只是想简单的用用，下面的代码是一个使用示例。  
  
来源：[http://answers.unity3d.com/questions/743400/database-sqlite-setup-for-unity.html][3]  

```  
	string conn = "URI=file:" + Application.dataPath + "/PickAndPlaceDatabase.s3db"; //Path to database.  
     SqliteConnection dbconn;  
     dbconn =  new SqliteConnection(conn);  
     dbconn.Open(); //Open connection to the database.  
     SqliteCommand dbcmd = dbconn.CreateCommand();  
     string sqlQuery = "SELECT value,name, randomSequence " + "FROM PlaceSequence";  
     dbcmd.CommandText = sqlQuery;  
     SqliteDataReader reader = dbcmd.ExecuteReader();  
     while (reader.Read())  
     {
         int value = reader.GetInt32(0);  
         string name = reader.GetString(1);  
         int rand = reader.GetInt32(2);  
        
         Debug.Log( "value= "+value+"  name ="+name+"  random ="+  rand);  
     }  
     reader.Close();  
     reader = null;
     dbcmd.Dispose();  
     dbcmd = null;  
     dbconn.Close();
     dbconn = null;  
 }
```
  
## 封装 DbUtil (反射)
  
如何在Unity上使用用上Sqlite 在这里不做过多的讨论，因为上面的两个链接里已经有详细的描述了。这里讨论下另外的写法。  

保存数据时，我并不希望每次保存数据时都去写一遍Sql，而是希望更简洁的操作。我不想去关心我是在操作那个类的数据类，里面有哪些字段 等等的封装。  
  
比如创建两张不同的表：  
  
    dbUtil.CreateTable(new Pen());  
    dbUtil.CreateTable(new Person());  
  
插入一条记录：  
  
```  
dbUtil.Insert(new Pen() { Id = 1, name = "钢笔1", color = "red", size = 1.123 });  
```  
  
插入链表：  
  
```  
dbUtil.InsertList(personList);  
dbUtil.InsertList(penList);  
```  
  
不需要再去关心是哪个类，实体类有哪些字段等，就保存到数据库了。以前做Android开发的时候有很多开源框架可以使用，比如 [xUtils](https://github.com/wyouflf/xUtils) 等。在Unity这用Sqlite似乎需求比较少，所以自己动手写个简单点的。用反射似乎效率会比较低，反射虽然慢，但对我们来说足以够快了。况且 数据库操作不都是放到另外的线程里去的么？  
  
### Column.cs  
  
首先要完成上面的操作，重要的一点就是拼接Sql语句，那就要先拿到类的相关信息。Sqlite 支持四中类型，blob用的少，不管了。另外三个 int对应integer，string对应text，double对应real。代码如下：  
  
```  
namespace Liangddyy.Util  
{  
    /// <summary>  
    /// 反射字段  
    /// </summary>  
    public class Column  
    {  
        //Column Value  
        private readonly MyProperty property;  
        public string columnName;  
        public Type columnType;  
  
        public Column(Type ColumnType, string ColumnName, object value)  
        {  
            this.columnType = ColumnType;  
  
            this.columnName = ColumnName;  
  
            if (ColumnType == typeof(string))  
            {  
                string v = Convert.ToString(value);  
  
                property = new MyProperty<string>();  
                ((MyProperty<string>) property).SetValue(v);  
            }  
            else if (ColumnType == typeof(double))  
            {  
                double v = Convert.ToDouble(value);  
  
                property = new MyProperty<double>();  
                ((MyProperty<double>) property).SetValue(v);  
            }  
            else if (ColumnType == typeof(int))  
            {  
                int v = Convert.ToInt32(value);  
  
                property = new MyProperty<int>();  
                ((MyProperty<int>) property).SetValue(v);  
            }  
        }  
        
        public string TableType  
        {  
            get  
            {  
                if (IsPrimaryKey)  
                    return "INTEGER PRIMARY KEY";  
                return To_TableType(columnType);  
            }  
        }  
  
        public bool IsPrimaryKey  
        {  
            // 转换小写  
            get  
            {  
                return (columnName.ToLower().StartsWith("id") || columnName.ToLower().StartsWith("_id")) &&  
                       (columnType == typeof(int));  
            }  
        }  
  
        public object ColumnValue  
        {  
            get  
            {  
                if (property == null) return null;  
  
                if (columnType == typeof(double))  
                    return ((MyProperty<double>) property).GetValue();  
                if (columnType == typeof(string))  
                    return ((MyProperty<string>) property).GetValue();  
                if (columnType == typeof(int))  
                    return ((MyProperty<int>) property).GetValue();  
                return null;  
            }  
        }  
  
        public string To_TableType(Type type)  
        {  
            if (type == typeof(double)) return ColType.REAL.ToString();  
  
            if (type == typeof(int)) return ColType.INTEGER.ToString();  
  
            if (type == typeof(string)) return ColType.TEXT.ToString();  
  
            throw new Exception("Wrong Type");  
        }  
    }  
  
    /// <summary>  
    /// 枚举数据库Sqlite支持的几种类型  
    /// </summary>  
    public enum ColType  
    {  
        INTEGER,  
        TEXT,  
        REAL,  
        BLOB  
    }  
  
    public class MyProperty  
    {  
    }  
  
    public class MyProperty<T> : MyProperty  
    {  
        private readonly bool _isValue;  
        private bool _changing;  
        private T _value;  
  
        public MyProperty()  
        {  
#if UNITY_FLASH  
            _isValue = false;  
#else  
            _isValue = typeof(T).IsValueType;  
#endif  
        }  
  
        public MyProperty(T value)  
            : this()  
        {  
            _value = value;  
        }  
  
        public bool IsOfType(Type t)  
        {  
            return t == typeof(T);  
        }  
  
        public T GetValue()  
        {  
            return _value;  
        }  
  
        protected virtual bool IsValueDifferent(T value)  
        {  
            return !_value.Equals(value);  
        }  
  
        private bool IsClassDifferent(T value)  
        {  
            return !_value.Equals(value);  
        }  
  
        public virtual void SetValue(T value)  
        {  
            if (_changing)  
                return;  
            _changing = true;  
  
            bool changed;  
  
            if (_isValue)  
                changed = IsValueDifferent(value);  
            else  
                changed = ((value == null) && (_value != null)) ||  
                          ((value != null) && (_value == null)) ||  
                          ((_value != null) && IsClassDifferent(value));  
            if (changed)  
                _value = value;  
            _changing = false;  
        }  
    }  
}  
```  
  
### DbUtil.cs  
  
注意事项写在后面。  
  
```  
namespace Liangddyy.Util  
{  
    /// <summary>  
    ///     自己写的Sqlite封装类  
    /// </summary>  
    public class DbUtil  
    {  
#if UNITY_ANDROID  
    private static string path = "URI=file:" + Application.dataPath;  
#endif  
#if UNITY_IPHONE  
    private static string path = "data source=" + Application.persistentDataPath;  
#endif  
#if UNITY_STANDALONE_WIN  
        private static readonly string path = "data source=" + Application.dataPath;  
#endif  
        private static readonly string dbName = "/test01.db";  
        private static DbUtil dbUtil;  
  
        private SqliteConnection dbConnection;  
        private SqliteCommand dbCmd;  
        private SqliteTransaction dbTransaction;  
        private SqliteDataReader dbReader;  
  
  
        public static DbUtil getInstance()  
        {  
            if (dbUtil == null)  
                dbUtil = new DbUtil(path + dbName);  
            return dbUtil;  
        }  
  
        private DbUtil(string connectionStr)  
        {  
            try  
            {  
                dbConnection = new SqliteConnection(connectionStr);  
            }  
            catch (Exception e)  
            {  
                Debug.LogError(e.Message);  
            }  
        }  
  
        public SqliteDataReader ExecuteQuery(string queryString)  
        {  
            dbCmd = dbConnection.CreateCommand();  
            dbCmd.CommandText = queryString;  
  
            dbReader = dbCmd.ExecuteReader();  
  
            return dbReader;  
        }  
  
        public int ExecuteSql(string sqlStr)  
        {  
            if (dbConnection.State == ConnectionState.Closed)  
                dbConnection.Open();  
  
            dbCmd = dbConnection.CreateCommand();  
            dbCmd.CommandText = sqlStr;  
            return dbCmd.ExecuteNonQuery();  
        }  
  
        private int ExecuteSql(SqliteCommand sqlCmd)  
        {  
            return dbCmd.ExecuteNonQuery();  
        }  
  
        public void CreateTable<T>(T arg)  
        {  
            this.ExecuteSql(GetCmdCreateTable(arg));  
        }  
  
        public void Insert<T>(T record)  
        {  
            dbCmd = dbConnection.CreateCommand();  
            this.ExecuteSql(GetCmdInsert<T>(record, dbCmd));  
        }  
  
        public void InsertList<T>(List<T> Records)  
        {  
            dbTransaction = dbConnection.BeginTransaction();  
            try  
            {  
                for (var i = 0; i < Records.Count; i++)  
                    Insert(Records[i]);  
                dbTransaction.Commit();  
            }  
            catch (Exception)  
            {  
                dbTransaction.Rollback();  
                throw;  
            }  
        }  
  
        public void InsertArray<T>(T[] Records)  
        {  
            dbTransaction = dbConnection.BeginTransaction();  
            try  
            {  
                for (var i = 0; i < Records.Length; i++)  
                    Insert(Records[i]);  
                dbTransaction.Commit();  
            }  
            catch (Exception)  
            {  
                dbTransaction.Rollback();  
                throw;  
            }  
        }  
  
        public void Update<T>(T arg)  
        {  
            dbCmd = dbConnection.CreateCommand();  
            this.ExecuteSql(GetCmdUpdate(arg, dbCmd));  
        }  
  
  
        /// <summary>  
        ///     关闭数据库连接 销毁连接  
        /// </summary>  
        public void CloseConnection()  
        {  
            //销毁Command  
            if (dbCmd != null)  
                dbCmd.Cancel();  
            dbCmd = null;  
  
            //销毁Reader  
            if (dbReader != null)  
                dbReader.Close();  
            dbReader = null;  
  
  
            //销毁Connection  
            if (dbConnection != null)  
                dbConnection.Close();  
            dbConnection = null;  
        }  
  
        /// <summary>  
        /// 更新的sql语句  
        /// </summary>  
        /// <typeparam name="T"></typeparam>  
        /// <param name="arg">The argument.</param>  
        /// <param name="cmd">The command.</param>  
        /// <returns></returns>  
        private SqliteCommand GetCmdUpdate<T>(T arg, SqliteCommand cmd)  
        {  
            Type type = arg.GetType();  
            var TableName = type.Name;  
  
            Column[] Columns = GetColumnsByType(arg);  
            if ((Columns == null) || (Columns.Length < 1))  
            {  
                Debug.LogError("Type is Wrong");  
                return null;  
            }  
  
            Column key = null;  
            Column[] normal = null;  
            //Debug.Log("Now Insert:" + TableName);  
            
            var RowLength = Columns.Length;  
            normal = GetNormalColumn(Columns);  
            cmd.CommandText = "UPDATE " + TableName + " SET " + normal[0].columnName + " = @" + normal[0].columnName;  
  
            for (var i = 1; i < normal.Length; i++)  
                cmd.CommandText += "," + normal[i].columnName + " = @" + normal[i].columnName;  
  
            key = GetPrimaryColumn(Columns);  
            cmd.CommandText += " WHERE " + key.columnName + " = @" + key.columnName + ";";  
  
            for (var i = 0; i < RowLength; i++)  
                cmd.Parameters.Add(new SqliteParameter("@" + Columns[i].columnName, Columns[i].ColumnValue));  
            return cmd;  
        }  
  
        /// <summary>  
        ///     获取主键  
        /// </summary>  
        /// <param name="Columns">The columns.</param>  
        /// <returns></returns>  
        private Column GetPrimaryColumn(Column[] Columns)  
        {  
            foreach (var item in Columns)  
                if (item.IsPrimaryKey) return item;  
            return null;  
        }  
  
        /// <summary>  
        ///     获取普通字段  
        /// </summary>  
        /// <param name="Columns">The columns.</param>  
        /// <returns></returns>  
        private Column[] GetNormalColumn(Column[] Columns)  
        {  
            var tmp = new List<Column>();  
  
            foreach (var item in Columns)  
                if (!item.IsPrimaryKey)  
                    tmp.Add(item);  
            return tmp.ToArray();  
        }  
  
        /// <summary>  
        ///     获取类的相关字段信息。  
        /// </summary>  
        /// <param name="arg">The argument.</param>  
        /// <returns></returns>  
        private Column[] GetColumnsByType<T>(T arg)  
        {  
            FieldInfo[] fieldsThis =  
                arg.GetType().GetFields(BindingFlags.NonPublic | BindingFlags.Public | BindingFlags.Instance);  
            FieldInfo[] fieldsBase =  
                arg.GetType().BaseType.GetFields(BindingFlags.NonPublic | BindingFlags.Public | BindingFlags.Instance);  
  
            FieldInfo[] fields = new FieldInfo[fieldsBase.Length + fieldsThis.Length];  
  
            Array.Copy(fieldsBase, 0, fields, 0, fieldsBase.Length);  
            Array.Copy(fieldsThis, 0, fields, fieldsBase.Length, fieldsThis.Length);  
  
            int field_len = fields.Length;  
  
            if ((fields == null) || (field_len < 1))  
                return null;  
  
            Column[] tableType = new Column[field_len];  
            for (int i = 0; i < field_len; i++)  
            {  
                String Cur_Name = fields[i].Name;  
  
                Type Cur_Type = fields[i].FieldType;  
  
                var Value = fields[i].GetValue(arg);  
  
                tableType[i] = new Column(Cur_Type, Cur_Name, Value);  
            }  
            return tableType;  
        }  
  
        /// <summary>  
        /// 建表Sql  
        /// </summary>  
        /// <typeparam name="T"></typeparam>  
        /// <param name="arg">The argument.</param>  
        /// <returns></returns>  
        private string GetCmdCreateTable<T>(T arg)  
        {  
            Type type = arg.GetType();  
            string TableName = type.Name;  
  
            Column[] columns = GetColumnsByType(arg);  
  
            if ((columns == null) || (columns.Length < 1))  
            {  
                Debug.LogError("Type is Wrong");  
                //todo 异常处理  
                return null;  
            }  
  
            Debug.Log("NowCreate:" + TableName);  
  
            string queryCreate = "CREATE TABLE IF NOT EXISTS " + TableName + "(" + columns[0].columnName + " " +  
                                 columns[0].TableType;  
  
            for (var i = 1; i < columns.Length; i++)  
                queryCreate += ", " + columns[i].columnName + " " + columns[i].TableType;  
            queryCreate += ");";  
  
            return queryCreate;  
        }  
  
        /// <summary>  
        /// 获取一条插入sql语句  
        /// </summary>  
        /// <typeparam name="T"></typeparam>  
        /// <param name="record">The record.</param>  
        /// <param name="cmd">The command.</param>  
        /// <returns></returns>  
        private SqliteCommand GetCmdInsert<T>(T record, SqliteCommand cmd)  
        {  
            Type type = record.GetType();  
            string TableName = type.Name;  
  
            Column[] columns = GetColumnsByType(record);  
            if ((columns == null) || (columns.Length < 1))  
            {  
                Debug.LogError("Type is Wrong");  
                return null;  
            }  
  
            int RowLength = columns.Length;  
  
            cmd.CommandText = "INSERT INTO " + TableName + "(" + columns[0].columnName;  
            for (var i = 1; i < RowLength; i++)  
                cmd.CommandText += "," + columns[i].columnName;  
            cmd.CommandText += ") VALUES (@" + columns[0].columnName;  
  
            for (var i = 1; i < RowLength; i++)  
                cmd.CommandText += ",@" + columns[i].columnName;  
            cmd.CommandText += ");";  
  
            for (var i = 0; i < RowLength; i++)  
                cmd.Parameters.Add(new SqliteParameter("@" + columns[i].columnName, columns[i].ColumnValue));  
            return cmd;  
        }  
    }  
}  
```  
  
### 注意事项  
  
1 不同平台下数据库文件的路径不同，所以要判断一下。  
  
```  
#if UNITY_ANDROID  
    private static string path = "URI=file:" + Application.dataPath;  
#endif  
#if UNITY_IPHONE  
    private static string path = "data source=" + Application.persistentDataPath;  
#endif  
#if UNITY_STANDALONE_WIN  
        private static readonly string path = "data source=" + Application.dataPath;  
#endif  
```  
  
2 获取当前类的字段信息  
  
```  
GetType().GetFields(BindingFlags.NonPublic | BindingFlags.Public | BindingFlags.Instance);  
```  
  
  获取父类的字段信息（BaseType）  
  
```  
arg.GetType().BaseType.GetFields(BindingFlags.NonPublic | BindingFlags.Public | BindingFlags.Instance);  
```  
  
主要注意的是：  
  
`BindingFlags.Public` 只能拿到public字段，而往往我们写实体类时，都是私有属性，然后Get/Set 所以必须加上`BindingFlags.NonPublic` 才能获取私有属性。  
  
3 拼接Sql语句时切忌直接用String 拼接。  
  
否则可能会有下面类似的报错：  
  
```  
SqliteException: SQLite error Insufficient parameters supplied to the command ...  
```  
  
正确写法类似于下面或者上面贴出的代码中拼接Sql部分。  
  
```  
command.CommandText = "INSERT INTO [User] (UserPK ,Name ,Password 	,Category ,ContactFK) VALUES ( @UserPK , @Name , @Password , 		@Category , @ContactFK)";  
command.Parameters.Add(new SqliteParameter("@Name", "Has"));  
...  
```  
  
## 优雅的使用  
  
![BaiduShurufa_2016-12-9_15-22-22](http://539go.com/usr/uploads/2016/12-09/BaiduShurufa_2016-12-9_15-22-22.png)  
  
（截图是因为这里不必要的代码很长，这个代码又没有什么意思。）  
  
建了三个类，为了区别，另外两个类属性一个是private，一个是public。（Get/Set隐藏了）。  
  
随便绑定个脚本Start里放上下面的代码 测试一下  
  
```  
DbUtil dbUtil = DbUtil.getInstance();  
        dbUtil.CreateTable(new Person());//建表  
        dbUtil.CreateTable(new Pen());  
  
        dbUtil.Insert(new Pen() { Id = 1, name = "钢笔1", color = "red", size = 1.123 });//插入  
        dbUtil.Insert(new Person() { Id = 1, Name = "liangddyy", Email = "hehe" });  
  
        List<Pen> penList = new List<Pen>();  
        List<Person> personList = new List<Person>();  
        for (int i = 5; i < 20; i++)  
        {  
            Pen pen = new Pen() {Id = i, name = "钢笔1", color = "red", size = 1.123};  
            Person person = new Person() {Id = i, Name = "liang", Email = "你好"};  
            personList.Add(person);  
            penList.Add(pen);  
        }  
        dbUtil.InsertList(penList);//插入链表  
        dbUtil.InsertList(personList);  
        dbUtil.Update(new Person() { Id =  1, Name = "梁先生", Email = "这条记录被修改了哈哈" });  
  
        dbUtil.CloseConnection();  
```  
  
打开 Sqlite数据库文件  
  
![123](http://539go.com/usr/uploads/2016/12-09/123.png)![122](http://539go.com/usr/uploads/2016/12-09/122.png)  
  
搞定。  
另外，我用的Sqlite可视化工具是 Navicat Premium，各种数据库都能连接。神器。  
  
## 需完善的地方  
  
1. 查询部分。因为DbModel还没写。  
2. 异常处理。这个不太擅长。  
3. SaveOrUpdate() 和SaveOrUpdateList() 方法。  
4. Android里面比较流行用注解的方式，C#貌似是用自定义特性么？总之是想实现通过标注的方式来获取字段的特征，如是否非空等或者说通过实体类自定义字段名等。  
  
  
有时间再完善吧。  
  
[1]: https://sourceforge.net/projects/sqlitebrowser/  
[2]: http://blog.csdn.net/qinyuanpei/article/details/46812655#t1  
[3]: http://answers.unity3d.com/questions/743400/database-sqlite-setup-for-unity.html  
