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

## DBセットアップ（マイグレーション 〜 テーブル作成）

- __bash__

  ```bash
  $ mix ecto.gen.migration create_people

  * creating priv/repo/migrations
  * creating priv/repo/migrations/20200517041113_create_people.exs
  ```

- priv/repo/migrations/20200517041113_create_people.exs

  ```elixir
    def change do
      create table(:people) do    --> add
        add :first_name, :string  --> add
        add :last_name, :string   --> add
        add :age, :integer        --> add
      end
  ```

- __bash__

  ```bash
  $ mix ecto.migrate

  13:28:21.372 [info]  == Running 20200517041113 Friends.Repo.Migrations.CreatePeople.change/0 forward

  13:28:21.373 [info]  create table people

  13:28:21.382 [info]  == Migrated 20200517041113 in 0.0s
  ```

#### 結果確認（テーブル）

```postgres
$ psql friends_repo
psql (12.2)
Type "help" for help.

friends_repo=# \dt
               List of relations
 Schema |       Name        | Type  |  Owner   
--------+-------------------+-------+----------
 public | people            | table | postgres
 public | schema_migrations | table | postgres
(2 rows)

friends_repo=# \d people
                                      Table "public.people"
   Column   |          Type          | Collation | Nullable |              Default               
------------+------------------------+-----------+----------+------------------------------------
 id         | bigint                 |           | not null | nextval('people_id_seq'::regclass)
 first_name | character varying(255) |           |          | 
 last_name  | character varying(255) |           |          | 
 age        | integer                |           |          | 
Indexes:
    "people_pkey" PRIMARY KEY, btree (id)

```

- Ectoでの __DBテーブル作成__ に成功しました。

- （メモ）

  - マイグレーションでミスがあった場合、`mix ecto.rollback` で変更を元に戻すことが可能。  
  （その後、変更修正してから、再度 `mix ecto.create`を実行する）

  - この段階で `mix ecto.rollback` すると、いま作成したばかりのテーブルを削除可能。

---

## スキーマ作成

- ファイル作成： `lib/friends/person.ex`

  ```elixir
  defmodule Friends.Person do
    use Ecto.Schema

    schema "people" do
      field :first_name, :string
      field :last_name, :string
      field :age, :integer
    end
  end
  ```

- `$ iex -S mix` で検証

  ```elixir
  iex(1)> person = %Friends.Person{}
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:built, "people">,
    age: nil,
    first_name: nil,
    id: nil,
    last_name: nil
  }

  iex(2)> person = %Friends.Person{age: 28}
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:built, "people">,
    age: 28,
    first_name: nil,
    id: nil,
    last_name: nil
  }

  iex(3)> person.age
  28

  iex(4)> Friends.Repo.insert(person)

  15:42:21.463 [debug] QUERY OK db=2.9ms decode=1.2ms queue=1.1ms idle=27.4ms
  INSERT INTO "people" ("age") VALUES ($1) RETURNING "id" [28]
  {:ok,
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
    age: 28,
    first_name: nil,
    id: 1,
    last_name: nil
  }}
  ```

### バリデーション機能実装

- `lib/friends/person.ex` の `defmodule` 内に、以下を追加

  ```elixir
  def changeset(person, params \\ %{}) do
    person
    |> Ecto.Changeset.cast(params, [:first_name, :last_name, :age])
    |> Ecto.Changeset.validate_required([:first_name, :last_name])
  end
  ```

- `$ iex -S mix` で検証 _（バリデーションエラー パターン）_

  ```elixir
  iex(1)> person = %Friends.Person{}
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:built, "people">,
    age: nil,
    first_name: nil,
    id: nil,
    last_name: nil
  }

  iex(2)> changeset = Friends.Person.changeset(person, %{})
  #Ecto.Changeset<
    action: nil,
    changes: %{},
    errors: [
      first_name: {"can't be blank", [validation: :required]},
      last_name: {"can't be blank", [validation: :required]}
    ],
    data: #Friends.Person<>,
    valid?: false
  >

  iex(3)> Friends.Repo.insert(changeset)
  {:error,
  #Ecto.Changeset<
    action: :insert,
    changes: %{},
    errors: [
      first_name: {"can't be blank", [validation: :required]},
      last_name: {"can't be blank", [validation: :required]}
    ],
    data: #Friends.Person<>,
    valid?: false
  >}

  iex(4)> changeset.valid?
  false
  ```

  - ちゃんと `:error` が返り、バリデーションが効いていることを確認できました。

- `$ iex -S mix` で検証 _（サクセス パターン）_

  データの `INSERT` (CREATE処理) まで合わせて実行します。

  ```
  iex(5)> person = %Friends.Person{}
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:built, "people">,
    age: nil,
    first_name: nil,
    id: nil,
    last_name: nil
  }

  iex(6)> changeset = Friends.Person.changeset(person, %{first_name: "im", last_name: "miolab"})
  #Ecto.Changeset<
    action: nil,
    changes: %{first_name: "im", last_name: "miolab"},
    errors: [],
    data: #Friends.Person<>,
    valid?: true
  >

  iex(7)> changeset.errors
  []

  iex(8)> changeset.valid?
  true

  iex(9)> Friends.Repo.insert(changeset)

  16:15:47.461 [debug] QUERY OK db=1.7ms decode=1.1ms queue=1.4ms idle=421.1ms
  INSERT INTO "people" ("first_name","last_name") VALUES ($1,$2) RETURNING "id" ["im", "miolab"]
  {:ok,
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
    age: nil,
    first_name: "im",
    id: 2,
    last_name: "miolab"
  }}
  ```

#### 結果確認（テーブル）

```postgres
$ psql friends_repo
psql (12.2)
Type "help" for help.

friends_repo=# select * from people;
 id | first_name | last_name | age 
----+------------+-----------+-----
  1 |            |           |  28
  2 | im         | miolab    |    
(2 rows)
```

- Ectoでの __CREATE__ 処理に成功しました。

- `id: 2` レコードで、`first_name` `last_name` のバリデーションも効いています。

---

## クエリ実行

- 事前準備として、DBを再作成します。

  - PostgreSQL

    ```postgres
    # exit
    ```

  - __bash__

    ```bash
    $ mix ecto.drop
    The database for Friends.Repo has been dropped

    $ mix ecto.create
    The database for Friends.Repo has been created

    $ mix ecto.migrate

    08:14:06.052 [info]  == Running 20200517041113 Friends.Repo.Migrations.CreatePeople.change/0 forward

    08:14:06.055 [info]  create table people

    08:14:06.078 [info]  == Migrated 20200517041113 in 0.0s
    ```

  - PostgreSQL

    ```postgres
    $ psql friends_repo

    friends_repo=# select * from people;
    id | first_name | last_name | age 
    ----+------------+-----------+-----
    (0 rows)
    ```

    DBが再作成されました。


- `$ iex -S mix` で以下を実行。

  クエリを構築 → クエリをリポジトリに渡す → DBに対してクエリを実行

  ```elixir
  iex(1)> people = [
  ...(1)> %Friends.Person{first_name: "im", last_name: "miolab", age: 28},
  ...(1)> %Friends.Person{first_name: "foo", last_name: "miolab", age: 27},
  ...(1)> %Friends.Person{first_name: "foobar", last_name: "hogehoge", age: 26},
  ...(1)> %Friends.Person{first_name: "eli", last_name: "xir", age: 26},
  ...(1)> ]
  [
    %Friends.Person{
      __meta__: #Ecto.Schema.Metadata<:built, "people">,
      age: 28,
      first_name: "im",
      id: nil,
      last_name: "miolab"
    },
    %Friends.Person{
      __meta__: #Ecto.Schema.Metadata<:built, "people">,
      age: 27,
      first_name: "foo",
      id: nil,
      last_name: "miolab"
    },
    %Friends.Person{
      __meta__: #Ecto.Schema.Metadata<:built, "people">,
      age: 26,
      first_name: "foobar",
      id: nil,
      last_name: "hogehoge"
    },
    %Friends.Person{
      __meta__: #Ecto.Schema.Metadata<:built, "people">,
      age: 26,
      first_name: "eli",
      id: nil,
      last_name: "xir"
    }
  ]

  iex(2)> Enum.each(people, fn(person) -> Friends.Repo.insert(person) end)

  10:17:19.023 [debug] QUERY OK db=2.3ms decode=1.0ms queue=2.6ms idle=1198.7ms
  INSERT INTO "people" ("age","first_name","last_name") VALUES ($1,$2,$3) RETURNING "id" [28, "im", "miolab"]

  10:17:19.027 [debug] QUERY OK db=1.1ms queue=1.1ms idle=1208.3ms
  INSERT INTO "people" ("age","first_name","last_name") VALUES ($1,$2,$3) RETURNING "id" [27, "foo", "miolab"]

  10:17:19.030 [debug] QUERY OK db=1.0ms queue=0.9ms idle=1210.7ms
  INSERT INTO "people" ("age","first_name","last_name") VALUES ($1,$2,$3) RETURNING "id" [26, "foobar", "hogehoge"]

  10:17:19.032 [debug] QUERY OK db=1.0ms queue=0.9ms idle=1212.8ms
  INSERT INTO "people" ("age","first_name","last_name") VALUES ($1,$2,$3) RETURNING "id" [26, "eli", "xir"] 
  :ok
  ```

  （※ `changeset` 不経由）

#### 結果確認（テーブル）

  ```postgres
  friends_repo=# select * from people;

   id | first_name | last_name | age
  ----+------------+-----------+-----
    1 | im         | miolab    |  28
    2 | foo        | miolab    |  27
    3 | foobar     | hogehoge  |  26
    4 | eli        | xir       |  26
  (4 rows)
  ```

---

## レコード取得（Read）

`$ iex -S mix` で確認。

- 最初のレコード 取得

  ```elixir
  iex(1)> Friends.Person |> Ecto.Query.first |> Friends.Repo.one

  11:17:20.928 [debug] QUERY OK source="people" db=0.8ms decode=1.6ms queue=1.5ms idle=1791.5ms
  SELECT p0."id", p0."first_name", p0."last_name", p0."age" FROM "people" AS p0 ORDER BY p0."id" LIMIT 1 []
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
    age: 28,
    first_name: "im",
    id: 1,
    last_name: "miolab"
  }
  ```

- 最後のレコード 取得

  ```elixir
  iex(2)> Friends.Person |> Ecto.Query.last |> Friends.Repo.one

  11:17:46.505 [debug] QUERY OK source="people" db=1.5ms queue=2.5ms idle=1377.0ms
  SELECT p0."id", p0."first_name", p0."last_name", p0."age" FROM "people" AS p0 ORDER BY p0."id" DESC LIMIT 1 []
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
    age: 26,
    first_name: "eli",
    id: 4,
    last_name: "xir"
  }
  ```

- __全レコード__ 取得

  ```elixir
  iex(3)> Friends.Person |> Friends.Repo.all

  11:19:58.105 [debug] QUERY OK source="people" db=0.3ms queue=0.4ms idle=1981.1ms
  SELECT p0."id", p0."first_name", p0."last_name", p0."age" FROM "people" AS p0 []
  [
    %Friends.Person{
      __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
      age: 28,
      first_name: "im",
      id: 1,
      last_name: "miolab"
    },
    %Friends.Person{
      __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
      age: 27,
      first_name: "foo",
      id: 2,
      last_name: "miolab"
    },
    %Friends.Person{
      __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
      age: 26,
      first_name: "foobar",
      id: 3,
      last_name: "hogehoge"
    },
    %Friends.Person{
      __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
      age: 26,
      first_name: "eli",
      id: 4,
      last_name: "xir"
    }
  ]
  ```

- __`id` を指定__ してレコード取得

  ```elixir
  iex(4)> Friends.Person |> Friends.Repo.get(1)

  11:21:08.927 [debug] QUERY OK source="people" db=1.2ms queue=1.6ms idle=1801.0ms
  SELECT p0."id", p0."first_name", p0."last_name", p0."age" FROM "people" AS p0 WHERE (p0."id" = $1) [1]
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
    age: 28,
    first_name: "im",
    id: 1,
    last_name: "miolab"
  }
  iex(5)> Friends.Person |> Friends.Repo.get(2)

  11:21:14.877 [debug] QUERY OK source="people" db=3.9ms idle=1749.7ms
  SELECT p0."id", p0."first_name", p0."last_name", p0."age" FROM "people" AS p0 WHERE (p0."id" = $1) [2]
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
    age: 27,
    first_name: "foo",
    id: 2,
    last_name: "miolab"
  }
  ```

- __カラム属性__ に基づいてレコードを取得

  ```elixir
  iex(6)> Friends.Person |> Friends.Repo.get_by(first_name: "im")

  11:23:40.635 [debug] QUERY OK source="people" db=1.4ms queue=5.4ms idle=1504.6ms
  SELECT p0."id", p0."first_name", p0."last_name", p0."age" FROM "people" AS p0 WHERE (p0."first_name" = $1) ["im"]
  %Friends.Person{
    __meta__: #Ecto.Schema.Metadata<:loaded, "people">,
    age: 28,
    first_name: "im",
    id: 1,
    last_name: "miolab"
  }
  ```

---
---

(on going)
