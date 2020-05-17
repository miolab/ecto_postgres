# Ecto & PostgreSQL

`Ecto` と `PostgreSQL` でいろいろお試しします。

- __Ecto__

  `Elixir`の
  - Databaseラッパー

  - クエリジェネレーター

- 参考

  - [Hex](https://hex.pm/packages/ecto_sql)

  - [Hexdocs](https://hexdocs.pm/ecto/getting-started.html)

---

## 実行環境

- macOS

  ```bash
  $ psql --version
  psql (PostgreSQL) 12.2

  $ elixir --version
  Erlang/OTP 22 [erts-10.5.3] [source] [64-bit] [smp:8:8] [ds:8:8:10] [async-threads:1] [hipe]

  Elixir 1.9.2 (compiled with Erlang/OTP 22)
  ```

---

## EctoでDB作成

Hexdocsを元に実行します。

- __bash__

  ```bash
  $ mix new friends --sup

  * creating README.md
  * creating .formatter.exs
  * creating .gitignore
  * creating mix.exs
  * creating lib
  * creating lib/friends.ex
  * creating lib/friends/application.ex
  * creating test
  * creating test/test_helper.exs
  * creating test/friends_test.exs

  Your Mix project was created successfully.
  You can use "mix" to compile it, test it, and more:

      cd friends
      mix test

  Run "mix help" for more commands.
  ```

  ```bash
  cd friends
  ```

- mix.exs

  ```elixir
  defp deps do
    [
      {:ecto_sql, "~> 3.0"},    -> add
      {:postgrex, ">= 0.0.0"}   -> add
  ```

- __bash__

  ```bash
  $ mix deps.get

  Resolving Hex dependencies...
  Dependency resolution completed:
  New:
    connection 1.0.4
    db_connection 2.2.2
    decimal 1.8.1
    ecto 3.4.4
    ecto_sql 3.4.3
    postgrex 0.15.4
    telemetry 0.4.1
  * Getting ecto_sql (Hex package)
  * Getting postgrex (Hex package)
  * Getting connection (Hex package)
  * Getting db_connection (Hex package)
  * Getting decimal (Hex package)
  * Getting ecto (Hex package)
  * Getting telemetry (Hex package)
  ```

  ```bash
  $ mix ecto.gen.repo -r Friends.Repo

  ==> connection
  Compiling 1 file (.ex)
  Generated connection app
  ===> Compiling telemetry
  ==> decimal
  Compiling 1 file (.ex)
  Generated decimal app
  ==> db_connection
  Compiling 14 files (.ex)
  Generated db_connection app
  ==> ecto
  Compiling 55 files (.ex)
  Generated ecto app
  ==> postgrex
  Compiling 61 files (.ex)
  Generated postgrex app
  ==> ecto_sql
  Compiling 26 files (.ex)
  Generated ecto_sql app
  ==> friends
  * creating lib/friends
  * creating lib/friends/repo.ex
  * creating config/config.exs
  Don't forget to add your new repo to your supervision tree
  (typically in lib/friends/application.ex):

      {Friends.Repo, []}

  And to add it to the list of ecto repositories in your
  configuration files (so Ecto tasks work as expected):

      config :friends,
        ecto_repos: [Friends.Repo]
  ```

- config/config.exs

  ```elixir
  config :friends, Friends.Repo,
    database: "friends_repo",
    username: "postgres",
    password: "postgres",
    hostname: "localhost"

  config :friends, ecto_repos: [Friends.Repo]  --> add
  ```

- lib/friends/application

  ```elixir
  def start(_type, _args) do
    children = [
      Friends.Repo,    --> add
  ```

- __bash__

  ```bash
  $ mix ecto.create

  Compiling 3 files (.ex)
  Generated friends app
  The database for Friends.Repo has been created
  ```

#### 結果確認（DB）

```postgres
$ psql -l
                                     List of databases
         Name         |  Owner   | Encoding |   Collate   |    Ctype    | Access privileges 
----------------------+----------+----------+-------------+-------------+-------------------
 friends_repo         | postgres | UTF8     | ja_JP.UTF-8 | ja_JP.UTF-8 | 
 .
 .
 .
```

- Ectoでの __DBの作成__ に成功しました。

---

## DBセットアップ（マイグレーション〜）

```bash
$ mix ecto.gen.migration create_people

* creating priv/repo/migrations
* creating priv/repo/migrations/20200517041113_create_people.exs
```




---


- （メモ）

  - マイグレーションでミスがあった場合、`mix ecto.rollback` で変更を元に戻すことが可能。  
  （その後、変更修正してから、再度 `mix ecto.create`を実行）

  - 最初の `mix ecto.create` 直後の段階で `mix ecto.rollback` すると、作成したばかりのテーブルを削除可能。

---

(on going)
