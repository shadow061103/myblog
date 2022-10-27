---
title: "測試mysql datetime2"
date: 2022-10-27T17:07:35+08:00
draft: true
tags: ["DB"]
type: post
author: "Steve Lin"
description: ""
---


# 測試mysql datetime2

### question
用mysql db 新增欄位時用datetime2會跳錯誤訊息
```
System.ArgumentOutOfRangeException: Index and length must refer to a location within the string. (Parameter 'length')
   at System.String.Substring(Int32 startIndex, Int32 length)
   at MySql.Data.Types.MySqlBinary.MySql.Data.Types.IMySqlValue.WriteValue(MySqlPacket packet, Boolean binary, Object val, Int32 length)
   at MySql.Data.MySqlClient.MySqlParameter.Serialize(MySqlPacket packet, Boolean binary, MySqlConnectionStringBuilder settings)
   at MySql.Data.MySqlClient.Statement.SerializeParameter(MySqlParameterCollection parameters, MySqlPacket packet, String parmName, Int32 parameterIndex)
   at MySql.Data.MySqlClient.Statement.InternalBindParameters(String sql, MySqlParameterCollection parameters, MySqlPacket packet)
   at MySql.Data.MySqlClient.Statement.BindParameters()
   at MySql.Data.MySqlClient.PreparableStatement.Execute()
   at MySql.Data.MySqlClient.MySqlCommand.ExecuteReader(CommandBehavior behavior)
   at MySql.Data.MySqlClient.MySqlCommand.ExecuteReader()
   at MySql.Data.MySqlClient.MySqlCommand.ExecuteNonQuery()
   at System.Data.Common.DbCommand.ExecuteNonQueryAsync(CancellationToken cancellationToken)
--- End of stack trace from previous location ---
   at Dapper.SqlMapper.ExecuteImplAsync(IDbConnection cnn, CommandDefinition command, Object param) in /_/Dapper/SqlMapper.Async.cs:line 646
   at Program.<Main>$(String[] args) in C:\Users\KuanFu\source\repos\ConsoleApp7\ConsoleApp7\Program.cs:line 19
```

### docker安裝DB
```
docker run --rm -d -e SA_PASSWORD=P@ssw0rd -e ACCEPT_EULA=Y -ti -p 1433:1433  mcr.microsoft.com/mssql/server:2019-latest

docker run --name sql2 -p 3306:3306 -e MYSQL_ROOT_PASSWORD=Dev127336 -d mysql:8
```
### program
```C#
using System.Data.SqlClient;
using Dapper;
using MySql.Data.MySqlClient;

Console.WriteLine("Hello, World!");

var connectstr = "Data Source=127.0.0.1,1433;Initial Catalog=Test;Persist Security Info=True;User ID=sa;Password=P@ssw0rd;Pooling=False;MultipleActiveResultSets=False;Connect Timeout=60;TrustServerCertificate=True";
connectstr = "Data Source=127.0.0.1,3306;Initial Catalog=Test;Persist Security Info=True;User ID=root;Password=Dev127336;Pooling=False;Connect Timeout=60;";
try
{
    using (var connection = new MySqlConnection(connectstr))
    {
        var sql = "insert into [MyTable] (Date) values (@Date);";
        sql = "INSERT INTO `Test`.`MyTable` (`Date`) VALUES (@Date);";
        var parameter = new DynamicParameters();
		//這邊可以改成Datetime2
        //parameter.Add("@Date", $"{DateTime.Now:yyyy-MM-dd HH:mm:ss.fffffff}", System.Data.DbType.DateTime2);
        parameter.Add("@Date", DateTime.Now, System.Data.DbType.DateTime);
        await connection.ExecuteAsync(sql, parameter);

    }
}
catch (Exception ex)
{ 
    Console.WriteLine(ex.ToString());
}

Console.ReadKey();
```

### 結論
mysql 沒有datetime2型別可以用
應該是誤用