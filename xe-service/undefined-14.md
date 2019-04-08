# 세션

## 세션\(Session\)

### 설정

HTTP 기반의 어플리케이션은 상태를 저장할수 없기 때문에, HTTP 요청들에 관계없이 사용자의 정보를 저장하기위해서 세션이 사용됩니다. XE는 세션을 사용함에 있어 간결하고 통일된 API를 제공합니다. 세션 시스템으로는 [Memcached](http://memcached.org), [Redis](http://redis.io) 그리고 데이터베이스를 별다른 설정 없이, 동일한 API로 사용할 수 있습니다.

세션의 설정은 `config/session.php` 파일에 있습니다. 이 파일에는 각각의 옵션에 대한 정리된 문서가 포함되어 있으므로 잘 확인하시기 바랍니다. 기본으로는 `file` 세션 드라이버가 사용됩니다.

> **참고:** 세션을 암호화하여 저장하고자 한다면 `encrypt` 설정 옵션을 `true`로 지정하십시오.

#### 예약어

XE는 내부적으로 `flash`라는 세션키를 사용하고 있습니다. 이 이름으로 세션을 추가하지 마시기 바랍니다.

### 세션 사용법

여러 가지 방법으로 세션을 엑세스할 수 있습니다. HTTP 요청-request의 `session` 메소드를 사용하는 방법, `Session` 파사드를 사용하는 방법, 그리고 `session` 헬퍼함수를 사용할 수 있습니다. 아무런 전달 인자 없이 `session` 헬퍼함수를 호출할 때에는 전체 세션 객체가 반환됩니다

#### 아이템 세션에 저장하기

```php
Session::put('key', 'value');

session(['key' => 'value']);
```

#### 배열 세션값으로 저장하기

```php
Session::push('user.teams', 'developers');
```

#### 세션에서 특정 아이템 가져오기

```php
$value = Session::get('key');

$value = session('key');
```

#### 특정 아이템을 찾거나 기본값 반환받기

```php
$value = Session::get('key', 'default');

$value = Session::get('key', function() { return 'default'; });
```

#### 아이템 값 가져온 후 삭제하기

```php
$value = Session::pull('key', 'default');
```

#### 세션의 모든 데이터 가져오기

```php
$data = Session::all();
```

#### 세션에 아이템이 존재하는지 확인하기

```php
if (Session::has('users'))
{
    //
}
```

#### 세션에서 특정 아이템 삭제하기

```php
Session::forget('key');
```

#### 세션의 모든 아이템 삭제하기

```php
Session::flush();
```

#### 세션 ID 재생성하기

```php
Session::regenerate();
```

### 임시 데이터

때로는 바로 다음번의 요청에서만 사용하기 위해서 세션에 아이템을 저장할 수 있습니다. 바로 `Session::flash` 메소드를 사용하는 것입니다:

```php
Session::flash('key', 'value');
```

#### 현재의 임시 데이터를 다른 요청에서 사용하기 위해서 다시 임시 저장하기

```php
Session::reflash();
```

#### 임시 데이터의 일부 값만 다시 임시 저장하기

```php
Session::keep(['username', 'email']);
```

### 데이터베이스 세션

`database` 세션 드라이버를 사용하는 경우 세션 아이템들이 저장될 테이블을 생성해야 합니다. 다음의 `Schema` 예제를 통해서 테이블을 생성할 수 있습니다.

```php
Schema::create('sessions', function($table)
{
    $table->string('id')->unique();
    $table->text('payload');
    $table->integer('last_activity');
});
```

`session:table` 아티즌 명령어를 사용하면 이 마이그레이션을 생성할 수 있습니다!

```text
php artisan session:table

composer dump-autoload

php artisan migrate
```

### 세션 드라이버

세션 “드라이버”는 각각의 요청에 따라 세션이 어디에 저장될지를 결정합니다. XE는 별다른 설정 없이 다양한 드라이버를 사용할 수 있습니다:

* `file` - 세션이 `storage/framework/sessions` 폴더에 저장됩니다.
* `cookie` - 암호화된 쿠키를 사용하여 안전하게 세션을 저장할 것입니다.
* `database` - 세션은 어플리케이션에 의해서 데이터베이스에 저장됩니다.
* `memcached` / `redis` - 세션은 캐시 기반의 드라이버들에 의해 빠르게 저장됩니다.
* `array` - 세션은 간단한 PHP 배열에 저장되지만 현재 요청에 한해 일시적으로 저장됩니다.

Redis 세션을 사용하기 위해서는 컴포저를 통해서 `predis/predis (~1.0)` 패키지를 설치해야만 합니다.

> **주의:** 배열 드라이버의 경우에는 실제로 세션은 유지되지 않기 때문에 [unit tests](https://github.com/xpressengine/xpressengine-manual/tree/c7478cb51aab4433d992bac673751500bc61d523/docs/5.0/testing/README.md)를 수행하는 데에만 사용하시기 바랍니다.

