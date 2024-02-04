## docker & rails & next.jsの環境構築

### ディレクトリ構成
```
- backend
  - Dockerfile
  - Gemfile
  - Gemfile.lock
  - entorypoint.sh
- frontend
  - Dockerfile
- docker-compose.yml
```

### 環境構築ステップ
##### 1. 任意の名前でディレクトリを作成し、このリポジトリのファイルをコピーする
##### 2. docker-compose.ymlのportを変更する(既存アプリとかぶっていたら)
##### 3. railsアプリを作成する(apiモードで)
```
$ docker compose run api rails new . --force --database=mysql --api
```
##### 4. backend/Docker fileを元の状態に修正する(rails newの際に上書きされるので)
##### 5. dockerを一度buildする(Gemfileが変わったので)
```
$ docker compose build
```
##### 6. config/database.ymlを書き換える(〇〇は任意の値)
```
default: &default
  adapter: mysql2
  encoding: utf8mb4
  pool: <%= ENV.fetch("RAILS_MAX_THREADS") { 5 } %>
  username: root
  password: password
  host: db

development:
  <<: *default
  database: 〇〇_development

test:
  <<: *default
  database: 〇〇_test

# production:
#   <<: *default
#   database: api_production
#   username: api
#   password: <%= ENV["API_DATABASE_PASSWORD"] %>
```
##### 7. dockerを起動
```
$ docker compose up -d
```
##### 8. DBを作成
```
$ docker compose exec api rails db:create
```
##### 9. バックエンドにアクセスして画面が表示されるか確認
```
http://localhost:〇〇〇〇/
```
##### 10. rack-corsをインストール
```
gem "rack-cors"のコメントアウトを外してbundle install
# docker compose exec api bundle install
```
##### 11-a. config/initializers/cors.rbを書き換える
```
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins "*"

    resource "*",
      headers: :any,
      methods: [:get, :post, :put, :patch, :delete, :options, :head]
  end
end
```

##### 11-b. backend/config/environments/development.rbに追加
```
  # NOTE: [ActionDispatch::HostAuthorization::DefaultResponseApp] Blocked hosts: api:3000
  config.hosts << "api"
```
-----------------
fornの設定
##### 12. nextアプリを作成する
```
$ docker compose run --rm front sh -c 'npx create-next-app . --typescript'
```

##### 13. 質問に答えてアプリを作成
```
Need to install the following packages:
  create-next-app@13.4.19
Ok to proceed? (y) y
✔ Would you like to use ESLint? … No / Yes → Yes
✔ Would you like to use Tailwind CSS? … No / Yes → Yes
✔ Would you like to use `src/` directory? … No / Yes → Yes
✔ Would you like to use App Router? (recommended) … No / Yes → Yes
? Would you like to customize the default import alias? › No / Yes → No
```

##### 14. フロントにアクセスして画面が表示されたら完了
```
http://localhost:〇〇〇〇/
```


## その他
##### git addしたらエラーが発生した場合
```
maekawahideki@maekawahidekinoMacBook-Pro blog_app % git add -A
error: 'backend/' does not have a commit checked out
fatal: adding files failed
```
↓ backend側でgit initされているため発生
.gitディレクトリを削除する
```
$ cd backend
$ rm -r .git
```

