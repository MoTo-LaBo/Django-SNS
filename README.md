# Django 社内SNS App 作成
- function based view で作成
  - sign up 作成・登録
  - html で user が情報を入力してそれを django が受けとって内部で user table に保存するという流れ
  - 内部でどのような処理が行われていくのかがハッキリとわかる
- browser 上で user 登録とログインできるようにする
- user がログインしない場合は中の情報を閲覧できない
## 1. 環境構築と初期設定
- 仮想環境とdjango install と upgrade
### 1. 仮想環境構築
    python3 -m venv venv
### 2. 仮想環境を立ち上げる
    source venv/bin/activate
### 3. django install
    pip install django
### 4. django upgrade
    pip install --upgrade pip
### 5. project 作成
    django-admin startproject boardproject .
### 6. App 作成
    python manage.py startapp boardapp
### 7. setting.py に app と DIRS に templats を記述 / url の繋ぎ込み
- INSTALLED_APPS に app を記述して認識させる
  - 'boardapp.apps.BoardappConfig' ※ 頭文字は大文字
- 'DIRS': [BASE_DIR / 'templates'], のように記述
  - templates directory 作成
- project urls.py file に path の記載と improt
  - path('', include('boardapp.urls')),
- app urls.py file 作成
  - project urls.py cp & ps
  - 今のところは、urlspatterns の記載がないと error が出てしまう為記載。後々削除する
> ここまででが初期設定でしておく事！この後は、project によって臨機応変にカスタムしていく
## 2. sign up view 作成 (簡易登録画面)
- function based view で作成
  - code は長くなってしまうが、内部でどのような事が起こっているか分かりやすい
### render method
    def signupfunc(request):
        return render(request, 'signup.html', {})
- app views.py file に上記を記述
- render(request, 'signup.html', {}) の中身は ↓
  - render (リクエスト、表示するhtml、{model の情報})
  - render(request, 'signup.html', {"some": 100})
  - ※ { key: value} で html で {{ some }} 記述する事で browser 上では１００が表示される
- render method を使うことによって HttpResponse のようなものを作れる
- render は非常によく使う method
## 3. POST と GET
- form などで data を送る場合は POST
  - django で error が出ないように、{% csrf_token %} は必ず記載する
  - < form method="POST" > {% csrf_token %}
- それ以外の web site に訪問する場合は GET
- request method とは？
> https://docs.djangoproject.com/ja/3.2/ref/request-response/
### 1. django が備えている user model
- user model を import する
> https://docs.djangoproject.com/ja/3.2/topics/auth/default/
### 2. user model import
    from django.contrib.auth.models import User
- object_list = User.objects.all()
  - function based view においても data をとってきてある変数に入れておく事により、data を html file で使用できる
  - 対象とする data を全てとってくる場合の記載
    - **User.objects.all()**
### 3. user model を作成
    python manage.py migrate
- table を作って database に書き込みをしなければ user を作成することができない
- 今回は makemigrations を入力しない
  - models.py file で新しい model を作成していないから
### 4. user を作成
    python manage.py createsuperuser
- request.POST を使う事によって form (sign up) からきた data を取得できる
- ユーザを作成する
  - ユーザを作成するための最も直接的な方法は、組み込まれている create_user() というヘルパー関数を利用することです
> https://docs.djangoproject.com/ja/3.2/topics/auth/default/
#### code 記述
    from django.contrib.auth.models import User
    user = User.objects.create_user('john', 'lennon@thebeatles.com', 'johnpassword')
    # At this point, user is a User object that has already been saved
    # to the database. You can continue to change its attributes
    # if you want to change other fields.
    user.last_name = 'Lennon'
    user.save()
### 5. django には default で重複した user の登録があると error を出してくれる
- IntegrityError at /signup/ UNIQUE constraint failed: auth_user.username
- 本番環境で user が間違えて重複して登録をしようとして、エラー画面が出ないように error をハンドリングしていく
- **try と except** を使用して error をハンドリングする
  - 何かしらしてみて、ダメだった場合(エラーが出た場合)の対応を記述してそちらが表示・動作するようにする
###  6. django integrityError を import
    from django.db import IntegrityError
## 4. user login させる
### 1. urls.py file に path(url)を作成・ views.py file に下記を記述
- path('login/', loginfunc, name='login'), import loginfunc
- 認証を有効としたいユーザーがいる場合 - login() 関数によってそれを行う事ができる
-  authenticate() (オーセンティケート)および login() を用いる
-  login()
   -  login method を使用して request object を受け取って、user 情報をもとに ログインさせる
-  authenticate()
   -  認証バックエンド
   -  その user がいるのかどうか？その user がどこまでの権限を持っていて, data をどこまで扱うことができるのか？login 前の確認認証
> https://docs.djangoproject.com/ja/3.2/topics/auth/default/
### code 記述
    from django.contrib.auth import authenticate, login

    def my_view(request):
        username = request.POST['username']
        password = request.POST['password']
        user = authenticate(request, username=username, password=password)
        if user is not None:
            login(request, user)
            # Redirect to a success page.
            ...
        else:
            # Return an 'invalid login' error message.
### 2. render と redirect
- render (レンダー)
  - 呼び出された関数 view の中で、受けっとた data を組み合わせて respons を返していく
  - 違う view を呼び出したりしない
  - context と違うものを入れていく
- redirect(リダイレクト)
  - 違う view を呼び出す
  - login が完了したら data 一覧に page を遷移させる
  - 何らかの処理が終わって、違う所に移りたい時に使用
## 5. List View 作成
### 1. urls.py file に path 記載
- path('list/', listfunc, name='list'), import listfunc
### 2. views.py file に listfunc 関数記載
### 3. models.py file 記載
    class BoardModel(models.Model):
        title = models.CharField(max_length=100)
        content = models.TextField()
        author = models.CharField(max_length=50)
        snsimage = models.ImageField(upload_to='')
        good = models.IntegerField()
        read = models.IntegerField()
        readtext = models.TextField()
### 4. ImageField 使用するため下記を install
    pip install Pillow
### 5. makemigrations 実行
    python manage.py makemigrations
- migrations  の中に migrations/0001_initial.py 作成される
- この data を元に database に対して書き込みをしていく
### 6. migrate 実行
    python manage.py migrate
- 上記の内容 model （table）が作成完了！
### 7. data を追加する為のフィールドを見るために admin.py file に記述
    from .models import BoardModel

    admin.site.register(BoardModel)
- 1~7 で作成した model を認識させる必要があるので記述と import をする
- python manage.py runserver
  - http://127.0.0.1:8000/admin/ で 管理画面で 作成した model の確認
### 8. List View の中に model の data を入れ込んでいく
- object_list = BoardModel.objects.all()
  - とする事で任意の model の data をとてくる事ができる
  - BoardModel に入っている object 全てを object_list という変数に入れることができる
  - views.py file に models の import も忘れずに行う
## 5. 画像(image) file の扱い方
- url 周りの設定をしていかないと画像は表示されない
  - 今回の設定はあくまでも開発環境での設定
- 一般に deploy した場合は、画像は django が扱うのではなく web server が扱いにする事が一般的
- それぞれ役割分担をして効率よくする為
  - django は pythonでの リクエスト -> レスポンスの複雑な処理
    - アプリケーションなどの複雑な処理
  - 画像や css は内部で処理をするのではなく, web server が対象となる css や画像を1個1個取ってくるというやり方
    - 簡単で単純な処理
### 1. setting.py file に下記を記載
    # BASE directory の media の中に画像file を保存しますという指示
    MEDIA_ROOT = BASE_DIR / 'media'

    # url と 画像file を結びつける時に使用される変数
    MADIA_URL = 'medi/'
### 2. django media static で検索
> https://docs.djangoproject.com/ja/3.2/howto/static-files/
#### 開発時の静的ファイルの取扱い
    from django.conf import settings
    from django.conf.urls.static import static

    urlpatterns = [
        # ... the rest of your URLconf goes here ...
    ] + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
- この機能は本番環境で利用するのに適していません! 一般的なデプロイ方法に関しては 静的ファイルのデプロイ を参照
#### ユーザーによりアップロードされるファイルの開発時の取扱い
    from django.conf import settings
    from django.conf.urls.static import static

    urlpatterns = [
    # ... the rest of your URLconf goes here ...
    ] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
- この機能は本番環境で利用するのに適していません! 一般的なデプロイ方法に関しては 静的ファイルのデプロイ を参照
- **project の urls.py file に貼り付けていく**
  - project 全体として画像をどう扱うかという事だから
  - 　+ static(settings.MEDIA_URL, document_root=settings.MEDIA_RO
  - 左から code を読む。static は path と同じ感じ
  - setting.py file の MEDIA_URL を取得してきたときに、settings.MEDIA_ROOT で指定した場所から対象となる画像を取ってくる
  - 文字列情報と対象となる画像を関連付ける
## 6. static file (css) の扱い方
### 1.  project urls.py file に下記を記載
    + static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
  - 内部の静的 file を django に取り扱わせる時の url の パターンを認識させる
### 2. setting file に下記を記載 / static directory 作成
    STATIC_ROOT = BASE_DIR / '/staticfiles/'
    STATICFILES_DIRS = [BASE_DIR / 'static']
- django に限らず、開発をしていく場合は複数の app を使ってそれぞれの app 事に使用する画像や css があるのが一般的
  - それぞれの page (支払い、お問い合わせ、商品一覧 etc...)で UI が異なるで用いていく画像や css も変わっていく
  - その中で１つの file で管理すると、変更があった場合どこかで整合性が取れなくなりカオスな状態になってしまう
- なので App の中で static という directory を作成してその中で管理していく。その方が効率が良くなる
  - 開発に対応する形で、django が STATICFILES_DIRS = [ ] を備えている
  - list 型 で複数の directory を指定していくことができる
#### 2-1. STATIC_ROOT と STATICFILES_DIRS の関係
    STATIC_ROOT = BASE_DIR / '/staticfiles/'

    STATICFILES_DIRS = [str(BASE_DIR / 'static')]
- STATIC_ROOT は deploy （本番環境）に適用されるもの
  - STATICFILES_DIRS で作成された静的 file を全て staticfiles にコピーをしていく
  - 本番環境は１つの場所にまとめて扱っていく方が効率が良い
#### 2-2. 上記を簡単に出来るコマンド
    python manage.py collectstatic
- これで静的ファイルが、プロジェクト直下の staticfiles に集約される
- 本番環境ではとても重要なところ！！！
> https://qiita.com/saira/items/a1c565c4a2eace268a07
### 3. html file に記述
- base.html に {% block customcss %}{% endblock customcss %} 記述
- signin.html に、{% load static %} 記述
  - おそらく django に STATICFILES_DIRS に入っている static directory の場所を教える為の code
### 3-1. signup.html に下記を記述
    {% block customcss%}
    <link href="{% static 'style.css' %}" rel="stylesheet">
    {% endblock customcss%}
### 3-2. static dir に style.css 作成 css の記述
> https://getbootstrap.jp/docs/5.0/examples/sign-in/signin.css
- 今回は Bootstrap からそのまま cp  & ps
- runserver で signup 画面を確認
## 7. login 状態の判定
- 2つのやり方を実証
> https://docs.djangoproject.com/ja/3.2/topics/auth/default/
### 1. login requierd での実装
- login_required デコレータ
  - もしユーザがログインしていなければ、settings.LOGIN_URL にリダイレクトし、クエリ文字列に現在の絶対パスを渡します。リダイレクト先の例: /accounts/login/?next=/polls/3/
  - login したら同じ page に戻れるようにする設定
  - django はコマンド実行で defaulet で実装できるようになっている
### 2. App views.py に下記の import して、デコレーターを記述
    from django.contrib.auth.decorators import login_required
- listfunc 関数の上に @login_required を記述
  - 指定した関数が呼び出さられる前に、呼び出さられる（処理される）内容

### 1. テンプレートの中に記述していく
- list.html の中に 条件文を使用して記述していく
  - {% if user.is_authenticated %}
  - is_authenticated は、Ture か False が返ってくる
    - 端的にいうとログインしている場合に(Ture の場合) if 文の中の code が実行される
- if 文が適さなかった場合
  - {% else %} 記述して、処理内容を記述
- 最後は {% endif %} を忘れずに記述する
## 8. logout 機能を実装
- ユーザーをログアウトさせるには
> https://docs.djangoproject.com/ja/3.2/topics/auth/default/
### code
    from django.contrib.auth import logout

    def logout_view(request):
        logout(request)
        # Redirect to a success page.
- Redirect to a success page.(リダイレクト先を指定すること)
### 1. app views.py file 新規関数を記述と import
- 上記の def を記述
### 2. app urls.py file に path を通す
    path('logout/', logoutfunc, name='logout'),
- import  logoutfunc も忘れずに記述
1. view を呼びだす為の link を list.html に貼り付けていく
2. a tag で link を作成。link を踏む事によって指定された名前の view を呼び出す
3. 今回だと logoutfunc という view を呼び出す
4. views の中で logoutfunc が実行されて logout method が実行されて、対象とする user が logout され
5. login の画面に redirect される
- 上記のような流れになっている。reverse のようなイメージ
## 9. DetailView
- function based view で実装するので対象となる object をどう取ってくるのかという違いに着目する
### 1. path を登録
- path('detail/< int:pk >', detailfunc, name='detail'),
- detailfunc の import も忘れずに
### 2. views.py file に関数を記述
    def detailfunc(request, pk):
        object = get_object_or_404(BoardModel, pk=pk)
        return render(request, 'detail.html', {'object': object})
- django に備わっている便利機能
  - get_object_or_404
  - 対象の model に入っている番号の object があれば object を返す
  - ない場合は、404 page Not found error を返す(が表示される)
> https://docs.djangoproject.com/ja/3.2/topics/http/shortcuts/#django.shortcuts.get_object_or_404
#### 例
    from django.shortcuts import get_object_or_404

    def my_view(request):
        obj = get_object_or_404(MyModel, pk=1)
### 10. いいね機能実装
- いいねが押された時に呼び出さられる link を作成
  - link を押すと、いいね中身でいいねの数を足す関数を実行
  - その後に redirect をして 何らかの page に返していく
### 1. App urls.py file に path を追加
- path('good/<int:pk>', goodfunc, name='good'),
]
- goodfunc の import も忘れずに
### 2. App views.py file に goodfunc 記述
    def goodfunc(request, pk):
        object = BoardModel.objects.get(pk=pk)
        object.good += object.good
        object.save()
        return redirect('list')

## 11. 既読機能実装
- 今回実装するものは本番環境に耐える事ができないので注意！！
- なぜそのようにしたのか？
  - function Based View は Django なのだが、あくまで python の文法にしたがって1個1個 class や def(関数) を記述しているだけという事を実感してもらう為
### 1. App urls.py file に path を追加
- path('read/<int:pk>', readfunc, name='read'),
- readfunc の import も忘れずに
### 2. App views.py file に readfunc 記述
- 既読ボタンを押すと内部で処理が走る
  - 既読するというボタンを押した人が、すでに既読ボタン押している場合は何もしない
  - 既読ボタンを押していない場合は、改めて既読に＋１増やす。既読した人の情報に既読ボタンを押した user の名前を追加していく
  - それで保存をしていく。その後にどこかの page に redirect させる
### 3. login している user の情報を取得してくる
    username = request.user.get_username()
- django が備えている属性や method を使用する事で取得できる
  - request.user.get_username()
  - 上記のリクエスト method で login user と username も取得することができる
## 12. class based view (ファイルのup load)
1. App urls.py file に path を追加
- path('create/', BoardCreate.as_view(), name='create'),
2. App views.py file に class 記述
3. create.html 作成
   - シンプルな form だけ作成していく
     - < form method="POST" enctype="multipart/form-data" >{% csrf_token %}
     - {% csrf_token %} を忘れずに。エラー防止
     - 複数の data を扱う場合には、"multipart/form-data" の記述が必要。text data と file (img) data ２種を扱う為
   - {{ form.as_p }} でも作成できるが,今回は author の情報も見えてしまうので使用できない
   - １つ１つ＜p＞ tag で form を作成していく
     - < input type="hidden" name="author" value="{{ user.username }}" >
     - hidden で表側には表示されない
     - {{ user.username }} : 今ログインしている user name を取得してくる
4. Not null constraint
- database では、default で何かしら data が入ってないと error になる
### それを避けるために models.py file に下記を記載
    good = models.IntegerField(null=True, blank=True, default=1)
    read = models.IntegerField(null=True, blank=True, default=1)
    readtext = models.TextField(null=True, blank=True, default='a')
- 引数に記載する事で回避する事ができる(null=True, blank=True, default=1)
  - null　->　DB 関係
  - blanc　->　 入力された form 関係
  - default　->　最初か値や文字を指定して入れておく事ができる
#### 5. create.html に login requierd をつける
    {% if user.is_authenticated %}
    {% else %}
    please login
    {% endif %}
## 13. まとめ
- render (レンダーメソッド)
  - render method  Httprespons object を返す method
  - 引数として request, templat, context この３つの情報を使用して respons を作成していく
- POST と GET
  - django において主要な method
  - form に data が送られた場合などに処理を分ける際に使用する
  - 挙動を変えていく
- User.object
  - function based view においてどのようにして data を持ってくるのか
  - User が model , object が 対象とする data (modelにあるモノ)
  - user.object とする事で model の data をすべて取得する事ができる
  - さまざまな method をつける事によって data を効率よく使用できる。意図する data を取得する事ができる
- Create_user, login, authenticate, logout
  - authenticate
  - browser にどのように情報を表示させていくか判断させる
- css, image の扱い方
  - とりわけ開発時でも扱えるようにする
  - deploy の手法はまた別の方法
- ログイン判定
  - class based view は,　デコレータをつける事によって login しているかどうかの判定をして処理をする(redirect) させる
