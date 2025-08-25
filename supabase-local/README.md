# Dockerを使ってローカル環境でSupabase CLIを使ってみる

しりとりゲームを作ってみました。

## 環境
+ ubuntu 22.04

## 手順
+ (未インストールなら) Dockerをインストールする
```
$ sudo apt install -y docker.io
$ docker version
```
+ インストール後、docker deamonが起動しない (permission denied と表示される) 場合
```
$ sudo groupadd docker 2>/dev/null || true 
$ sudo usermod -aG docker $USER
$ newgrp docker
```
と打ったあと、deamonが起動していることを確認する。
```
$ docker run hello-world
$ docker version
```
+ 次に Supabase-CLI をインストールする
```
$ sudo apt install -y curl
$ curl -fsSL https://github.com/supabase/cli/releases/latest/download/supabase_linux_amd64.tar.gz | tar xvz
$ sudo mv supabase /usr/local/bin
$ supabase --version
```
+ 適当な場所にフォルダ作成し、そこでsupabaseプロジェクトのDockerコンテナを立てる
```
$ mkdir foo && cd foo
$ supabase init
$ supabase start
```
+ プロジェクトのコンテナが正しく起動していることを確認する <br>

```
$ docker ps
```
+ (ちなみに、supabaseプロジェクトはPostgreSQLに54322のポート番号を割り当てる。複数の supabaseプロジェクトに対して同時にコンテナを起動することはできない(ポート競合)。すでに他のプロジェクトで supabase start していた場合は、先にそちらを stop する必要がある)
+ 準備ができたので、スキーマを作成する。以下のコマンドで表示される Studio URL のリンクへ飛び、サイドバーから SQL Editor を開く。
```
$ supabase status
```
+ SQL Editor に SQL文 を入力し、Run する (以下は適当に作ったサンプル)。
```sql
create extension if not exists pgcrypto;

create table if not exists moves (
  id bigserial primary key,
  game_id uuid not null,
  player text not null,
  word text not null,
  created_at timestamptz default now()
);

alter table moves enable row level security;

create policy "local_anyone_can_read_moves"
on moves for select
to anon
using (true);

create policy "local_anyone_can_insert_moves"
on moves for insert
to anon
with check (true);

begin;
drop publication if exists supabase_realtime;
create publication supabase_realtime;
commit;
alter publication supabase_realtime add table moves;
```
+ 最後に、フロントエンドを動作させる(添付のhtmlファイル(これも適当に作ったサンプル)を開く)

### 使い方
+ HTMLを2個ブラウザ上で開く
+ 2枚のページでGameIDを揃える(片方のGameIDをもう一方にコピペする)
+ 両方のページで `your name` を適当に入力する
+ 両方のページで `Join/Resubscribe` をクリックする
+ 適当にワードを入力して送信すると両方のページにログが出力される

### 参考リンク
https://qiita.com/oka_yu_f/items/d5044ecd571315c25a93 <br>
https://qiita.com/SatoshiSobue/items/a612ebbb3a9242c09db5 <br>
https://zenn.dev/bani24884/articles/e3f56462b9f409


