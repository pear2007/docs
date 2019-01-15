* * *

layout: article language: 'en' version: '4.0'

* * *

<h5 class="alert alert-warning">This article reflects v3.4 and has not yet been revised</h5>

<a name='overview'></a>

# セキュリティ

このコンポーネントは、パスワードハッシュやCross-Site Request Forgery ([CSRF](https://en.wikipedia.org/wiki/Cross-site_request_forgery)) 対策などの, 一般的なセキュリティタスクで開発者を支援します。

<a name='hashing'></a>

## パスワードのハッシュ

プレーンテキストでパスワードを保存することは、セキュリティ上の習慣としては不適切です。 データベースへのアクセス権を持つ人は、すぐにすべてのユーザーアカウントにアクセスし、権限がなくても行動することができてしまいます。 To combat that, many applications use the familiar one way hashing methods '[md5](https://php.net/manual/en/function.md5.php)' and '[sha1](https://php.net/manual/en/function.sha1.php)'. しかし、ハードウェアは毎日進化し、より高速になり、これらのアルゴリズムは総当たり攻撃に対して脆弱になっています。 These attacks are also known as [rainbow tables](https://en.wikipedia.org/wiki/Rainbow_table).

The security component uses [bcrypt](https://en.wikipedia.org/wiki/Bcrypt) as the hashing algorithm. Thanks to the '[Eksblowfish](https://en.wikipedia.org/wiki/Bcrypt#Algorithm)' key setup algorithm, we can make the password encryption as `slow` as we want. Slow algorithms minimize the impact of bruce force attacks.

Bcrypt, is an adaptive hash function based on the Blowfish symmetric block cipher cryptographic algorithm. It also introduces a security or work factor, which determines how slow the hash function will be to generate the hash. This effectively negates the use of FPGA or GPU hashing techniques.

Should hardware becomes faster in the future, we can increase the work factor to mitigate this.

This component offers a simple interface to use the algorithm:

```php
<?php

use Phalcon\Mvc\Controller;

class UsersController extends Controller
{
    public function registerAction()
    {
        $user = new Users();

        $login    = $this->request->getPost('login');
        $password = $this->request->getPost('password');

        $user->login = $login;

        // パスワードのハッシュの保存
        $user->password = $this->security->hash($password);

        $user->save();
    }
}
```

デフォルトの作業係数でパスワードのハッシュを保存します。より高い作業係数により暗号化が遅くなれば、パスワードの脆弱性が低くなります。パスワードが正しいかどうかを次のように確認できます。

```php
<?php

use Phalcon\Mvc\Controller;

class SessionController extends Controller
{
    public function loginAction()
    {
        $login    = $this->request->getPost('login');
        $password = $this->request->getPost('password');

        $user = Users::findFirstByLogin($login);
        if ($user) {
            if ($this->security->checkHash($password, $user->password)) {
                // パスワード有効
            }
        } else {
            // タイミング攻撃への対応 ユーザーの有無に関わらず、
            // このスクリプトは常にハッシュ計算と
            // 同程度の時間をかける
            $this->security->hash(rand());
        }

        // バリデーション失敗時
    }
}
```

The salt is generated using pseudo-random bytes with the PHP's function [openssl_random_pseudo_bytes](https://php.net/manual/en/function.openssl-random-pseudo-bytes.php) so is required to have the [openssl](https://php.net/manual/en/book.openssl.php) extension loaded.

<a name='csrf'></a>

## Cross-Site Request Forgery (CSRF) の保護

これは、Webサイトやアプリケーションに対するもう1つの一般的な攻撃です。 ユーザーの登録やコメントの追加などのタスクを実行するように設計されたフォームは、この攻撃に対して脆弱です。

そのアイデアは、フォームの値がアプリケーションの外部に送信されないようにすることです。 To fix this, we generate a [random nonce](https://en.wikipedia.org/wiki/Cryptographic_nonce) (token) in each form, add the token in the session and then validate the token once the form posts data back to our application by comparing the stored token in the session to the one submitted by the form:

```php
<?php echo Tag::form('session/login') ?>

    <!-- ログイン と パスワードの入力 ... -->

    <input type='hidden' name='<?php echo $this->security->getTokenKey() ?>'
        value='<?php echo $this->security->getToken() ?>'/>

</form>
```

このコントローラのアクションでは、CSRFトークンが有効かどうかを確認できます:

```php
<?php

use Phalcon\Mvc\Controller;

class SessionController extends Controller
{
    public function loginAction()
    {
        if ($this->request->isPost()) {
            if ($this->security->checkToken()) {
                // トークンOK
            }
        }
    }
}
```

セッションアダプターをあなたのDIに追加するのを忘れないでください。そうしないと、このトークンチェックは動作しません。

```php
<?php

$di->setShared(
    'session',
    function () {
        $session = new \Phalcon\Session\Adapter\Files();

        $session->start();

        return $session;
    }
);
```

Adding a [captcha](https://www.google.com/recaptcha) to the form is also recommended to completely avoid the risks of this attack.

<a name='setup'></a>

## コンポーネントの設定

このコンポーネントは`security`としてサービスコンテナに自動的に登録されます。オプションを設定するために再登録することができます:

```php
<?php

use Phalcon\Security;

$di->set(
    'security',
    function () {
        $security = new Security();

        // パスワードハッシュの係数を12ラウンドに設定
        $security->setWorkFactor(12);

        return $security;
    },
    true
);
```

<a name='random'></a>

## 乱数

The [Phalcon\Security\Random](api/Phalcon_Security_Random) class makes it really easy to generate lots of types of random data.

```php
<?php

use Phalcon\Security\Random;

$random = new Random();

// ...
$bytes      = $random->bytes();

// 長さ $len の ランダム16進数文字列の生成
$hex        = $random->hex($len);

// 長さ $len のbase64文字列の生成
$base64     = $random->base64($len);

// 長さ $len の URL-safeなbase64文字列の生成
$base64Safe = $random->base64Safe($len);

// UUID (version 4) の生成
// 詳細については以下を参照してください https://en.wikipedia.org/wiki/Universally_unique_identifier
$uuid       = $random->uuid();

// 0 から $n の間の乱数の生成
$number     = $random->number($n);
```

<a name='resources'></a>

## External Resources

* [Vökuró](https://vokuro.phalconphp.com) はCSRFとパスワードハッシュ攻撃を避けるためのセキュリティコンポーネントを使うサンプルアプリケーションです。[Github](https://github.com/phalcon/vokuro)