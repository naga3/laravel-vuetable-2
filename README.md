# LaravelとVuetable-2でページネーション付きテーブルを使ってみる

[vuetable-2](https://github.com/ratiw/vuetable-2)というVue用の高機能なテーブルコンポーネントを見つけたので、Laravelをバックエンドにして使ってみたサンプルです。

# プロジェクト作成

まず適当な名前でプロジェクトを作成します。

laravelコマンドを使うなら以下のような感じで。

```
laravel new laravel-vuetable-2
```

# テーブル作成

元々存在している`xxxx_create_users_table`を改変し、名前・メール・住所フィールドの入った`users`テーブルを作成します。

```php
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name');
            $table->string('email');
            $table->string('address');
            $table->timestamps();
        });
    }
```

```
php artisan migrate
```

`User`モデルは最初からあるものをそのまま使います。

# ダミーデータ作成

Laravelにバンドルされている`Faker`を使ってダミーデータを1000件ほど作成しておきます。

`database/seeds/DatabaseSeeder.php`の`run`メソッドに追加します。

```php
    public function run()
    {
        DB::table('users')->delete();
        $faker = Faker\Factory::create('ja_JP');
        for ($i = 0; $i < 1000; $i++) {
            User::create([
                'name' => $faker->name,
                'email' => $faker->email,
                'address' => $faker->address,
            ]);
        }
    }
```

```
php artisan db:seed
```

# vuetable-2の導入

[vuetable-2](https://github.com/ratiw/vuetable-2)をnpmで導入します。

```
npm i vuetable-2 -D
```

# 一覧テーブルの表示

まずはとにかく一覧を表示してみます。

## API

一覧表示のAPI部分を作成します。

`routes/api.php`に元から書いてあるルートを上書きします。

```php
<?php
use Illuminate\Http\Request;

Route::middleware('api')->get('/user', function(Request $request) {
    return App\User::paginate((int)$request->per_page);
});
```

`api/user`にアクセスしてみて一覧が取得されることを確認してください。

- `per_page`は、Vuetableから渡される1ページあたりの行数です。

## Vueコンポーネント

今回は元々ある`resources/assets/js/components/Example.vue`に上書きします。
コンポーネントを新規作成しても、もちろん構いません。

```html
<template>
  <vuetable
    api-url="api/user"
    :fields="['name', 'email', 'address']"
  ></vuetable>
</template>

<script>
import Vuetable from 'vuetable-2'

export default {
  components: {
    Vuetable
  }
}
</script>
```

- `api-url`属性は、データを取得するAPIのURLです。
- `fields`属性は、表示するフィールドです。

## bladeテンプレート

表示するガワを作成します。`resources/views/welcome.blade.php`を上書きして使います。

```html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>Vuetable-2サンプル</title>
    <link href="{{ asset('css/app.css') }}" rel="stylesheet">
</head>
<body>
    <div id="app">
        <example></example>
    </div>
    <script src="{{ asset('js/app.js') }}"></script>
</body>
</html>
```

これでブラウザでアクセスすると、一覧が10件表示されます。

# ページネーション

次にページネーション用のナビゲーションを設置してみます。幸いにもVuetableのページネーションは、Laravelのページネーションの機構をほぼそのまま使うことができます。

## Vueコンポーネント

ページネーション用のコンポーネントを追加します。

`resources/assets/js/components/Example.vue`

```html
<template>
  <div>
    <vuetable
      ref="vuetable"
      api-url="api/user"
      :fields="['name', 'email', 'address']"
      pagination-path=""
      @vuetable:pagination-data="onPaginationData"
    ></vuetable>
    <vuetable-pagination
      ref="pagination"
      @vuetable-pagination:change-page="onChangePage"
    ></vuetable-pagination>
  </div>
</template>

<script>
import {Vuetable, VuetablePagination} from 'vuetable-2'

export default {
  components: {
    Vuetable, VuetablePagination
  },
  methods: {
    onPaginationData(data) {
      this.$refs.pagination.setPaginationData(data)
    },
    onChangePage(page) {
      this.$refs.vuetable.changePage(page)
    }
  }
}
</script>
```

ブラウザで確認すると、スタイルは当たっていませんが、ページネーションが動くはずです。

- `VuetablePagination`コンポーネントでページネーションを表示します。
- Laravelのページネーションデータと合わせるために、Vuetableコンポーネントの`pagination-path`属性をセットしています。
- API側から正常にデータを取得できたとき、Vuetableコンポーネントの`pagination-data`イベントが発火するので、そのときにVuetablePaginationコンポーネントの`setPaginationData`メソッドへページネーションデータを渡してページネーションを構築します。
- ページネーションを操作したとき、VuetablePaginationコンポーネントの`change-page`イベントが発火するので、そのときVuetableコンポーネントの`changePage`メソッドへページ情報を渡して当該ページを取得します。

## スタイリング

Vuetableは[Semantic UI](https://semantic-ui.com/)の使用を前提にしています。Bootstrapの場合はページネーション用のスタイリングを変更する必要があります。

`resources/assets/js/components/Example.vue`

```html
<template>
  <div>
    <vuetable
      ref="vuetable"
      api-url="api/user"
      :fields="['name', 'email', 'address']"
      pagination-path=""
      @vuetable:pagination-data="onPaginationData"
    ></vuetable>
    <vuetable-pagination
      ref="pagination"
      @vuetable-pagination:change-page="onChangePage"
      :css="css"
    ></vuetable-pagination>
  </div>
</template>

<script>
import {Vuetable, VuetablePagination} from 'vuetable-2'

export default {
  components: {
    Vuetable, VuetablePagination
  },
  data() {
    return {
      css: {
        wrapperClass: 'pagination',
        activeClass: 'active',
        disabledClass: 'disabled',
        icons: {
          first: 'glyphicon glyphicon-backward',
          prev: 'glyphicon glyphicon-triangle-left',
          next: 'glyphicon glyphicon-triangle-right',
          last: 'glyphicon glyphicon-forward'
        }
      }
    }
  },
  methods: {
    onPaginationData(data) {
      this.$refs.pagination.setPaginationData(data)
    },
    onChangePage(page) {
      this.$refs.vuetable.changePage(page)
    }
  }
}
</script>

<style>
.pagination a {
  cursor: pointer;
  border: 1px solid lightgray;
  padding: 5px 10px;
}
.pagination a.active {
  color: white;
  background-color: #337ab7;
}
.pagination a.btn-nav.disabled {
  color: lightgray;
  cursor: not-allowed;
}
</style>
```

- VuetablePaginationコンポーネントの`css`属性にクラスやアイコンを指定します。

# まとめ

LaravelとVuetable-2の相性はとても良いことが分かりました。
