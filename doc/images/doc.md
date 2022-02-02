# [.NET6][C#] Sqlite3を使ってみる。

だいぶ前に試してみて記事も上げたような気がするんだけど、見当たらなかったのと.NET6でも試してみようと思って使ってみた。

- Visual Studio 2022を使って新規プロジェクトでC#コンソールアプリを作成する。
- NuGetを使ってMicrosoft.Data.Sqlite.CoreとSQLitePCLRaw.bundle_e_sqlite3をインストールする。
![](images/001.png)

Program.csを次のようにする。

Program.cs
~~~csharp
using Microsoft.Data.Sqlite;
using System.Data;

using (var con = new SqliteConnection("Data Source=test.db"))
{
    con.Open();

    // テーブルを作る
    try
    {
        var command = con.CreateCommand();
        command.CommandText = "create table if not exists test(id INTEGER PRIMARY KEY, name TEXT NOT NULL, val INTEGER NOT NULL)";
        command.ExecuteNonQuery();
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
        return 1;
    }

    // データを追加
    try
    {
        for (var i = 0; i < 10; i++)
        {
            var command = con.CreateCommand();
            command.CommandText = "insert into test(name, val) values(@name, @val)";
            command.Parameters.AddWithValue("@name", $"hoge{i}");
            command.Parameters.AddWithValue("@val", i);
            var result = command.ExecuteNonQuery();
            if (result != 1)
            {
                Console.WriteLine("insert失敗");
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
        return 1;
    }
    // 検索
    try
    {
        var command = con.CreateCommand();
        command.CommandText = "select * from test";
        using (var reader = command.ExecuteReader())
        {
            var datatable = new DataTable();
            datatable.Load(reader);

            foreach (var row in datatable.AsEnumerable())
            {
                var id = row["id"].ToString();
                var name = row["name"].ToString();
                var val = row["val"].ToString();
                Console.WriteLine($"{id}, {name}, {val}");
            }
        }
    }
    catch (Exception ex)
    {
        Console.WriteLine(ex.ToString());
        return 1;
    }

}

return 0;
~~~

これを実行するとtest.dbが作成される。次のようにコンソールに出力される。

![](images/002.png)

ちょっと試してみたんだけど、macOSに.NET6 SDKをインストールして*.csprojファイルがあるフォルダに移動して"dotnet run"を実行するとちゃんと動く。

これも[github](https://github.com/miyamoto999/sampleSqlite3_cs)に置いておいた。