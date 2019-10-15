## 概要

Laravelを使い、シンプルなCRUDサンプルを作成する

## 手順
### lacrud作成
#### Laravelのひな型作成

まず、`composer create-project --prefer-dist laravel/laravel lacrud "5.8.*"`をターミナルで実行する

```shell
composer create-project --prefer-dist laravel/laravel lacrud "5.8.*"
```

アプリ作成後、`lacrud`ディレクトリへと移動します

```shell
cd lacrud
```

`php artisan serve`をターミナルで実行するとローカルサーバが起動します

```shell
php artisan serve
```

`localhost:8000`にブラウザからアクセスして以下の画面が表示されればOKです

#### Postモデルの作成

早速、CRUDで作成する`Post`モデルを作成します

Laravelではターミナルのコマンドからモデルやコントローラーを作成することができます
ターミナルからモデルやコントローラーを簡単に作成することができるので能率よく開発することができます

以下のコマンドで`Post`モデルを作成します

```shell
php artisan make:model -m Post
```

`make:model`コマンドでモデルを作成します
`-m`はデータベースへのマイグレーションファイルを同時に作成するオプションです

コマンド実行後、`app/Post.php`と`database/migrations/XXXX_XX_XX_XXXXXX_create_posts_table.php`が作成されいると思います

`app/Post.php`には`Post`モデルでのデータ処理のメソッドなどを記述していきます

`database/migrations/XXXX_XX_XX_XXXXXX_create_posts_table.php`はマイグレーションファイルと呼ばれるものです
データベースにテーブルを作成したり、既存のテーブルに新しい項目を追加することもできます

`database/migrations/XXXX_XX_XX_XXXXXX_create_posts_table.php`をエディタで開き、`public function up()`を以下のように変更します

```php:database/migrations/XXXX_XX_XX_XXXXXX_create_posts_table.php
    public function up()
    {
        Schema::create('posts', function (Blueprint $table) {
            $table->bigIncrements('id');
            $table->string('title');
            $table->text('content');
            $table->timestamps();
        });
    }
```

その後、ターミナルで`php artisan migrate`を実行します

```shell
php artisan migrate
```

これで`posts`テーブルが作成され、そのテーブル内に`title`と`content`という項目が追加されます

#### PostControllerの作成

先ほどはモデルを作成しましたので、次にコントローラーを作成します

コントローラーの作成はターミナルから`make:controller`コマンドを実行することで簡単に作成できます

```shell
php artisan make:controller PostController --resource --model=Post
```

これで`app/Http/Controllers/PostController.php`が作成されます

`--resource`はRESTfulなアクションを生成できるオプションです。
これを使うことによって必要なアクションをスムーズに作成できます

また、`--model=Post`ではコントローラーが使用するモデルを指定することができます

これで`Post`モデルを取り扱うコントローラーが作成されました

#### RESTfulなルーティングの設定

先ほど作成したコントローラーの各アクションを呼び出すルーティングを設定します

Laravelでのルーティング設定は`routes/web.php`にて設定することができます

```php:routes/web.php
Route::resource('posts', 'PostController');
```

`Route::resource('posts', 'PostController');`で`/posts`や`/posts/1`などのRESTfulなルーティングを設定できます

#### ビューの作成

次に、各アクションごとのビューを作成していきます

まずは`localhost:8000/posts`の`index`アクションのビューを作成します

`app/Http/Controllers/PostController.php`の`index`アクションを以下のように修正します

```php:app/Http/Controllers/PostController.php
    public function index()
    {
        return view('posts.index');
    }
```

`view()`メソッドに引数として渡している`'posts.index'`で`resources/views/posts/index.blade.php`をビューとして呼び出すように指定しています

次に、`resources/views/posts/index.blade.php`を以下のように作成します

```php:resources/views/posts/index.blade.php
<h1>Posts</h1> 
```

`php artisan serve`でローカルサーバを起動し、`localhost:8000/posts`にアクセスしてみてください

ブラウザに`Posts`と表示されていればOkです

#### 新しいPostを作成

まず、新しい`Post`を作成するリンクを`resources/views/posts/index.blade.php`に追加します

```php:resources/views/posts/index.blade.php
<h1>Posts</h1>

<a href="/posts/create">New Post</a>
```

次に、`app/Http/Controllers/PostController.php`の`create`アクションを以下のように編集します

```php:app/Http/Controllers/PostController.php
    public function create()
    {
        return view('posts.create');
    }
```

その後、`resources/views/posts/create.blade.php`を作成します

```php:resources/views/posts/create.blade.php
<form method="POST" action="/posts">
    {{ csrf_field() }}
    <input type="text" name="title">
    <input type="text" name="content">
    <input type="submit">
</form>
```

`<form method="POST" action="/posts">`で`/posts`というURLに`POST`リクエストでフォームに入力されたデータを送信しています

RESTfulなURLでは、`/posts`のような`/[モデル名]`のURLに`POST`リクエストでパラメータを送信することで新しいデータを作成します

また`Laravel`では`CSRF`という攻撃方法に対処するための対策がされています。
`{{ csrf_field() }}`では`CSRFトークン`を生成し、フォームで入力されたパラメータを送信できるようにしています

最後に、`app/Http/Controllers/PostController.php`の`store`アクションを以下のように変更します

```php:app/Http/Controllers/PostController.php
    public function store(Request $request)
    {
        $post = new Post();
        $post->title = $request->input('title');
        $post->content = $request->input('content');
        $post->save();

        return redirect()->route('posts.show', ['id' => $post->id])->with('message', 'Post was successfully created.');
    }
```

`store`アクションでは`/posts`へ`POST`リクエストで送信されたパラメータを受け取っています 

受け取ったパラメータは`$request`内に保存されています 
`$request->input('title')`のようにパラメータを取り出すことができます

```php
    $post = new Post();
```

では新しい`Post`モデルのデータを作成しています

```php
    $post->title = $request->input('title');
    $post->content = $request->input('content');
```

`$request`に保存されているパラメータを上記のように`Post`モデルのカラムごとに受け取ります

```php
    $post->save();
    return redirect()->route('posts.show', ['id' => $post->id])->with('message', 'Post was successfully created.');
```

最後に、`$post->save();`で作成したモデルのデータを保存して`/posts/:id`というURLへリダイレクトします

現段階では`/posts/:id`のビューは作成していないのでエラーになります
次に、`/posts/:id`のビューなどを作成します

#### 個別の詳細ページを作成

まずは、`app/Http/Controllers/PostController.php`の`show`アクションを以下のように変更します

```php:app/Http/Controllers/PostController.php
    public function show(Post $post)
    {
        return view('posts.show', compact('post'));
    }
```

`show`アクションの引数`$post`は`/posts/:id`から自動的に該当する投稿データを取得しています

ちなみに、以下のようにして取得することもできます

```php:app/Http/Controllers/PostController.php
    public function show($id)
    {
        $post = Post::findOrFail($id);

        return view('posts.show', compact('post'));
    }
```

また、`compact('post')`でビューに変数を渡すことができます


あとは、`resources/views/posts/show.blade.php`を以下のように作成します

```php:resources/views/posts/show.blade.php
    @if (session('message'))
        {{ session('message') }}
    @endif

    {{ $post->title }}
    {{ $post->content }} 
```

これで、新しい投稿を行うと`show`アクションを呼び出し、作成した内容が表示されます

#### 作成した投稿一覧を作成

新規作成ページと閲覧ページは実装できました

次に、作成した投稿の一覧を実装していきます

まず、`app/Http/Controllers/PostController.php`の`index`アクションを以下のように変更します

```php:app/Http/Controllers/PostController.php
    public function index()
    {
        $posts = Post::all();

        return view('posts.index', compact('posts'));
    }
```

`Post::all()`で作成されている投稿をすべて取得しています

そして、`resources/views/posts/index.blade.php`で受け取った`$post`を`@foreach`で展開します

```php:resources/views/posts/index.blade.php
<h1>Posts</h1>

@foreach($posts as $post)
    <a href="/posts/{{ $post->id }}">{{ $post->title }}</a>
@endforeach

<a href="/posts/create">New Post</a> 
```

これで各投稿への一覧ページが作成されます

#### 各投稿の編集ページ作成

各投稿を編集できるように実装していきます

まず、`resources/views/posts/index.blade.php`に`/posts/:id/edit`へのリンクを作成します

```php:resources/views/posts/index.blade.php
<h1>Posts</h1>

@foreach($posts as $post)
    <a href="/posts/{{ $post->id }}">{{ $post->title }}</a>
    <a href="/posts/{{ $post->id }}/edit">Edit</a>
@endforeach

<a href="/posts/create">New Post</a> 
```

各投稿の閲覧ページからも編集画面へ移動できるように`resources/views/posts/index.blade.php`にもリンクを作成します

```php:resources/views/posts/index.blade.php
    @if (session('message'))
        {{ session('message') }}
    @endif

    {{ $post->title }}
    {{ $post->content }} 

    <a href="/posts/{{ $post->id }}/edit">Edit</a>
```

次に、`app/Http/Controllers/PostController.php `の`edit`アクションを編集します

```php:
    public function edit(Post $post)
    {
        return view('posts.edit', compact('post'));
    }
```

`resources/views/posts/edit.blade.php`を作り、ビューを作成します

```php:resources/views/posts/edit.blade.php
    <form method="POST" action="/posts/{{ $post->id }}">
        {{ csrf_field() }}
        <input type="hidden" name="_method" value="PUT">
        <input type="text" name="title" value="{{ $post->title }}">
        <input type="text" name="content" value="{{ $post->content }}">
        <input type="submit">
    </form> 
```

最後に、`app/Http/Controllers/PostController.php`の`update`アクションを以下のように実装します

```php:app/Http/Controllers/PostController.php
    public function update(Request $request, Post $post)
    {
        $post->title = $request->input('title');
        $post->content = $request->input('content');
        $post->save();

        return redirect()->route('posts.show', ['id' => $post->id])->with('message', 'Post was successfully updated.');
    }
```

これで各投稿を編集することができるようになりました

#### 投稿の削除機能の実装

投稿を削除できるように実装していきます

まずは`resources/views/posts/index.blade.php`に投稿を削除するためのリンクを作成します

```php:resources/views/posts/index.blade.php
<h1>Posts</h1>

@foreach($posts as $post)
    <a href="/posts/{{ $post->id }}">{{ $post->title }}</a>
    <a href="/posts/{{ $post->id }}/edit">Edit</a>

    <form action="/posts/{{ $post->id }}" method="POST" onsubmit="if(confirm('Delete? Are you sure?')) { return true } else {return false };">
        <input type="hidden" name="_method" value="DELETE">
        <input type="hidden" name="_token" value="{{ csrf_token() }}">
        <button type="submit">Delete</button>
    </form>
@endforeach

<a href="/posts/create">New Post</a> 
```

Laravelで削除のリンクを実装する際には`<form>`タグを使い、コントローラーの`destroy`アクションへ`DELETE`リクエストを送ります

最後に、`app/Http/Controllers/PostController.php`の`destroy`アクションで投稿を削除するように修正します

```php:app/Http/Controllers/PostController.php
    public function destroy(Post $post)
    {
        $post->delete();

        return redirect()->route('posts.index');
    }
```

これで投稿の削除ができるようになります

#### Bootstrap4を使用する

最後に作成したアプリにBootstrap4を使用します

まず、各ページの基本レイアウトになる`resources/views/layouts/layouts.blade.php`を作成します

```php:resources/views/layouts/layouts.blade.php
<html>
    <head>
        <title>@yield('title')</title>
        <meta name="csrf-token" content="{{ csrf_token() }}">
        <link href="{{ asset('css/app.css') }}" rel="stylesheet">
    </head>
    <body>
        <div class="container">
            @yield('content')
        </div>

         <script src="{{ asset('js/app.js') }}"></script>
    </body>
</html> 
```

Laravelでは標準でBootstrap４が導入されており、

```php
<link href="{{ asset('css/app.css') }}" rel="stylesheet">
```

と

```php
<script src="{{ asset('js/app.js') }}"></script>
```

によってBootstrap4が呼び出されています

#### 作成したレイアウトを各ページで使用する

先ほど作成した`resources/views/layouts/layouts.blade.php`を使用して各ページのレイアウトを変更します

まずは`resources/views/posts/index.blade.php`を以下のように変更します

```php:resources/views/posts/index.blade.php
@extends('layouts.layouts')

@section('title', 'lacrud')

@section('content')
    <h1>Posts</h1>

    @foreach($posts as $post)
        <a href="/posts/{{ $post->id }}">{{ $post->title }}</a>
        <a href="/posts/{{ $post->id }}/edit">Edit</a>

        <form action="/posts/{{ $post->id }}" method="POST" onsubmit="if(confirm('Delete? Are you sure?')) { return true } else {return false };">
            <input type="hidden" name="_method" value="DELETE">
            <input type="hidden" name="_token" value="{{ csrf_token() }}">
            <button type="submit">Delete</button>
        </form>
    @endforeach

    <a href="/posts/create">New Post</a> 
@endsection
```

`@extends('layouts.layouts')`で使用するレイアウトのファイルを宣言しています 
今回は`resources/views/layouts/layouts.blade.php`を使用しています

また`@section('title', 'lacrud')`は`resources/views/layouts/layouts.blade.php`内で定義している`@yield('title')`に埋め込むというコードになります

同様に`@section('content')`から`@endsection`は`@yield('content')`に埋め込まれます

Laravelではこういった形で共通のレイアウトなどを使用していきます

残りのページも同様に修正していきます

まずは`resources/views/posts/create.blade.php`を以下のように変更します

```php:resources/views/posts/create.blade.php
@extends('layouts.layouts')

@section('title', 'lacrud')

@section('content')
    <form method="POST" action="/posts">
        {{ csrf_field() }}
        <input type="text" name="title">
        <input type="text" name="content">
        <input type="submit">
    </form>
@endsection
```

次に`resources/views/posts/show.blade.php`を以下のように編集します

```php:resources/views/posts/show.blade.php
@extends('layouts.layouts')

@section('title', 'lacrud')

@section('content')
    @if (session('message'))
        {{ session('message') }}
    @endif

    {{ $post->title }}
    {{ $post->content }}
@endsection 
```

最後に、`resources/views/posts/edit.blade.php`を修正します

```php:resources/views/posts/edit.blade.php
@extends('layouts.layouts')

@section('title', 'lacrud')

@section('content')
    <form method="POST" action="/posts/{{ $post->id }}">
        {{ csrf_field() }}
        <input type="hidden" name="_method" value="PUT">
        <input type="text" name="title" value="{{ $post->title }}">
        <input type="text" name="content" value="{{ $post->content }}">
        <input type="submit">
    </form> 
@endsection
```

#### ヘッダーを追加する

Bootstrap4を使ってヘッダーを作成します

Laravelではコンポーネントといってヘッダーやフッターなどの共通部品を作成し使用することができます

先ほどの`resources/views/layouts/layouts.blade.php`にコンポーネント使用するコードを追加します

```php:resources/views/layouts/layouts.blade.php
<html>
    <head>
        <title>@yield('title')</title>
        <meta name="csrf-token" content="{{ csrf_token() }}">
        <link href="{{ asset('css/app.css') }}" rel="stylesheet">
    </head>
    <body>
        @component('components.header')
        @endcomponent
        <div class="container">
            @yield('content')
        </div>

        <script src="{{ asset('js/app.js') }}"></script>
    </body>
</html>
```

コンポーネントの呼び出しを行っているのは下記の部分になります

```php:
    @component('components.header')
    @endcomponent
```

`components.header`は`resources/views/components`ディレクトリ内の`header.blade.php`を呼び出すというコードになります


そして、`resources/views/components/header.blade.php`を以下のように作成します

```php:resources/views/components/header.blade.php
<nav class="navbar navbar-expand-lg navbar-light bg-light">
  <a class="navbar-brand" href="/">lacrud</a>
  <button class="navbar-toggler" type="button" data-toggle="collapse" data-target="#navbarSupportedContent" aria-controls="navbarSupportedContent" aria-expanded="false" aria-label="Toggle navigation">
    <span class="navbar-toggler-icon"></span>
  </button>

  <div class="collapse navbar-collapse" id="navbarSupportedContent">
    <ul class="navbar-nav mr-auto">
      <li class="nav-item active">
        <a class="nav-link" href="/">Home <span class="sr-only">(current)</span></a>
      </li>
      <li class="nav-item">
        <a class="nav-link" href="#">Link</a>
      </li>
      <li class="nav-item dropdown">
        <a class="nav-link dropdown-toggle" href="#" id="navbarDropdown" role="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
          Dropdown
        </a>
        <div class="dropdown-menu" aria-labelledby="navbarDropdown">
          <a class="dropdown-item" href="#">Action</a>
          <a class="dropdown-item" href="#">Another action</a>
          <div class="dropdown-divider"></div>
          <a class="dropdown-item" href="#">Something else here</a>
        </div>
      </li>
      <li class="nav-item">
        <a class="nav-link disabled" href="#" tabindex="-1" aria-disabled="true">Disabled</a>
      </li>
    </ul>
    <form class="form-inline my-2 my-lg-0">
      <input class="form-control mr-sm-2" type="search" placeholder="Search" aria-label="Search">
      <button class="btn btn-outline-success my-2 my-sm-0" type="submit">Search</button>
    </form>
  </div>
</nav>
```

作成後、ターミナルで`php artisan serve`を実行してローカルサーバを起動します

`localhost:3000/posts`にアクセスし、ヘッダーが追加されていればOKです

#### 各ページのデザインにBootstrap4を適用させる

各ページのデザインにBootstrap4を使用していきます

まず、`resources/views/posts/index.blade.php`を以下のように編集していきます

```php:resources/views/posts/index.blade.php
@extends('layouts.layouts')

@section('title', 'lacrud')

@section('content')
<h1>Posts</h1>

@foreach($posts as $post)

    <div class="card">
        <div class="card-body">
            <h5 class="card-title">{{ $post->title }}</h5>
            <p class="card-text">{{ $post->content }}</p>

            <div class="d-flex" style="height: 36.4px;">
                <a href="/posts/{{ $post->id }}" class="btn btn-outline-primary">Show</a>
                <a href="/posts/{{ $post->id }}/edit" class="btn btn-outline-primary">Edit</a>
                <form action="/posts/{{ $post->id }}" method="POST" onsubmit="if(confirm('Delete? Are you sure?')) { return true } else {return false };">
                    <input type="hidden" name="_method" value="DELETE">
                    <input type="hidden" name="_token" value="{{ csrf_token() }}">
                    <button type="submit" class="btn btn-outline-danger">Delete</button>
                </form>
            </div>
        </div>
    </div>
@endforeach

<a href="/posts/create">New Post</a>
@endsection
```

各投稿をBootstrap4のCardを使用して表示させています。
詳細ページなどへのリンクは`btn btn-outline-primary`を使うことでマウスが重なった時にボタンの色が変わるようになっています

ボタンのレイアウトは`Flex`という`Bootstrap`のユーティリティを使用して調整しています

`resources/views/posts/show.blade.php`も同様の修正を行います

```php:resources/views/posts/show.blade.php
@extends('layouts.layouts')

@section('title', 'lacrud')

@section('content')

    @if (session('message'))
        {{ session('message') }}
    @endif

    <div class="card">
        <div class="card-body">
            <h5 class="card-title">{{ $post->title }}</h5>
            <p class="card-text">{{ $post->content }}</p>

            <div class="d-flex" style="height: 36.4px;">
                <button class="btn btn-outline-primary">Show</button>
                <a href="/posts/{{ $post->id }}/edit" class="btn btn-outline-primary">Edit</a>
                <form action="/posts/{{ $post->id }}" method="POST" onsubmit="if(confirm('Delete? Are you sure?')) { return true } else {return false };">
                    <input type="hidden" name="_method" value="DELETE">
                    <input type="hidden" name="_token" value="{{ csrf_token() }}">
                    <button type="submit" class="btn btn-outline-danger">Delete</button>
                </form>
            </div>
        </div>
    </div>

    <a href="/posts/{{ $post->id }}/edit">Edit</a> | 
    <a href="/posts">Back</a>

@endsection
```

`resources/views/posts/edit.blade.php`も以下のように変更します

```php:resources/views/posts/edit.blade.php
@extends('layouts.layouts')

@section('title', 'lacrud')

@section('content')

<h1>Editing Post</h1>

<form method="POST" action="/posts">
    {{ csrf_field() }}
    <div class="form-group">
        <label for="exampleInputEmail1">Title</label>
        <input type="text" class="form-control" aria-describedby="emailHelp" name="title" value="{{ $post->title }}">
    </div>
    <div class="form-group">
        <label for="exampleInputPassword1">Content</label>
        <textarea type="password" class="form-control" name="content">{{ $post->content }}</textarea>
    </div>
    <button type="submit" class="btn btn-outline-primary">Submit</button>
</form>

<a href="/posts/{{ $post->id }}">Show</a> | 
<a href="/posts">Back</a>

@endsection
```

`resources/views/posts/create.blade.php`も同様に変更します

```php:
@extends('layouts.layouts')

@section('title', 'lacrud')

@section('content')

<h1>New Post</h1>

<form method="POST" action="/posts">
    {{ csrf_field() }}
    <div class="form-group">
        <label for="exampleInputEmail1">Title</label>
        <input type="text" class="form-control" aria-describedby="emailHelp" name="title">
    </div>
    <div class="form-group">
        <label for="exampleInputPassword1">Content</label>
        <textarea type="password" class="form-control" name="content"></textarea>
    </div>
    <button type="submit" class="btn btn-outline-primary">Submit</button>
</form>

<a href="/posts">Back</a>

@endsection
```
