# Rails×React×TypeScript×Vite×PostgreSql×Pgadmin 環境構築

参照元：<br>
<a>https://zenn.dev/dsakai07/articles/84ceb379cffbcb</a><br>

docker-compose run --no-deps api rails new . --force --database=postgresql --api<br>
上記をする前に<br>
docker compose build<br>

docker-compose run --no-deps api rails new . --force --database=postgresql --api<br>
をすると本番環境用のDockerfileに上書きされる。<br>
<a>https://zenn.dev/yoiyoicho/articles/90e1ea64772ea8</a><br>
を元に開発環境用と本番環境用に分ける<br>

docker-compose.ymlをdocker-compose-dev.ymlに変更した場合以下コマンドとなる<br>
<pre>docker compose -f docker-compose-dev.yml build
docker compose -f docker-compose-dev.yml up -d
docker compose -f docker-compose-dev.yml down
docker compose -f docker-compose-dev.yml exec app bash
docker compose -f docker-compose-dev.yml run app rake db:create</pre>

ポスグレのホスト名、ユーザー名など変更時はvolumeを削除し<br>
docker-compose build --no-cache<br>
を行う必要あり。<br>
今回の場合、docker-compose.ymlにて<br>
<pre>
db:
    image: postgres
    # ホスト名、ユーザー名など変更時はvolumeを削除しdocker-compose build --no-cache
    volumes:
      - ./tmp/db:/var/lib/postgresql/data
      - ./docker/postgres/init.d:/docker-entrypoint-initdb.d
      - ./docker/postgres/pgdata:/var/lib/postgresql/data
    environment:
      - POSTGRES_HOST_AUTH_METHOD=trust
      - POSTGRES_USER=${DB_USER}
      - POSTGRES_PASSWORD=${DB_PASSWORD}
    ports:
      - "${DB_PORT}:5432"
</pre>
よりvolumeはdockerフォルダ内に値するため、dockerフォルダを削除する必要がある。<br>
<a>https://halkyo.com/post/web/docker-rails-postgresql-active-record-database-connection-error</a><br><br>

## フロントをReact×TypeScript×Vite対応方法

参照元：<br>
<a>https://zenn.dev/dsakai07/articles/84ceb379cffbcb</a><br>
<a>https://qiita.com/tomofu74/items/a8750738a1e18c471bbb</a>

docker-compose run front sh -c "npm create vite@latest"<br>
Vite作成時のオプションは以下URL参照<br>
https://osu-log.com/archives/379<br>

Vite作成時にできたpackage.jsonをfrontendフォルダ直下にコピー</br>
package.json内で以下変更<br>
<pre><s>"dev": "vite"</s> ➞ "dev": "vite --host"</pre>



docker compose up -d</br>
docker compose exec front bash</br>
cd app<br>
npm run dev<br>

docker-compose.ymlで設定したポート番号にて表示できると完了<br><br>

## Pgadminを表示する方法

参照元：<br>
<a>https://pote-chil.com/blog/docker-postgres-pgadmin</a><br>

docker-compose.ymlのvolumeで以下のように設定している場合、
<pre>
    volumes:
    # var/lib/pgadminのフォルダを作成しておく必要あり
      - ./docker/pgadmin:/var/lib/pgadmin
</pre>
docker/pgadminフォルダの権限をWritableに変更する必要がある。<br>

docker compose up -d<br>
localhost:${PGADMIN_PORT}<br>
PGADMIN_DEFAULT_EMAIL、PGADMIN_DEFAULT_PASSWORDの値でログイン<br>

ログインできたら完了

## RailsとReactを紐づける（rack-corsを有効化）
Gemfileの以下コードのコメントアウトを外す。<br>
<pre>gem "rack-cors"</pre>

backend/config/initializers/cors.rbに以下を追記する。
<pre>
    Rails.application.config.middleware.insert_before 0, Rack::Cors do
        allow do
            origins 'localhost:3001' # フロントエンドのURLを指定
            resource '*',
            headers: :any,
            methods: [:get, :post, :put, :patch, :delete, :options, :head]
        end
    end
</pre>

docker compose exec front bash<br>
bundle install<br>

## 必要なパッケージをインストール
参照元：<br>
<a>https://pote-chil.com/blog/docker-postgres-pgadmin](https://musclecoding.com/rails-gem-recommended/)https://musclecoding.com/rails-gem-recommended</a><br>

上記参照URLにて必要なパッケージをbackend/Gemfileに記載する。<br>

<p style="color:red; border-bottom: 1px, solid, red;">※devise_token_authはrails7に対応していないため以下のように記載する</p>
<pre>
    gem 'devise_token_auth', '>= 1.2.0', git: "https://github.com/lynndylanhurley/devise_token_auth"
</pre>
<a>https://stackoverflow.com/questions/71062789/error-using-devise-token-auth-gem-rails-7</a><br>

必要なパッケージを記載完了後以下コマンドをたたく
<pre>
    docker compose exec front bash
    bundle install
</pre>
