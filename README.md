# PHP exchange1c - обмін 1С:Підприємство з сайтом на php
[![Packagist](https://img.shields.io/packagist/l/alexnsk83/exchange1c.svg?style=flat-square)](LICENSE)
[![Packagist](https://img.shields.io/packagist/dt/alexnsk83/exchange1c.svg?style=flat-square)](https://packagist.org/packages/bigperson/exchange1c)
[![Packagist](https://img.shields.io/packagist/v/alexnsk83/exchange1c.svg?style=flat-square)](https://packagist.org/packages/bigperson/exchange1c)

> [!NOTE]
> Цей репозиторій є форком оригінального проєкту [alex8bits/exchange1c](https://github.com/alex8bits/exchange1c).

Встановлення цієї бібліотеки має спростити інтеграцію 1С у ваш сайт.

Бібліотека містить набір інтерфейсів, які необхідно реалізувати, щоб отримати можливість обмінюватися товарами та документами з 1С. Передбачається, що у Вас є "1С:Підприємство 8, Управління торгівлею", редакція 11.3, версія 11.3.2 на платформі 8.3.9.2033.

Якщо у вас версія конфігурації нижча, то скоріш за все бібліотека все одно буде працювати, оскільки здебільшого обмін з сайтами суттєво не змінюється в 1С від версії до версії.

Ця бібліотека була написана на основі модуля https://github.com/carono/yii2-1c-exchange - всі основні інтерфейси взяті саме з цього модуля.

# Залежності
* php ^8.0
* imrev-agency/commerceml
* illuminate/contracts ^10|^11|^12
* symfony/http-foundation ^7.2

# Встановлення
`composer require imrev-agency/exchange1c`

# Використання з Laravel

Пакет підтримує Laravel Package Discovery. Після встановлення через Composer `Exchange1CServiceProvider` підключиться автоматично — біндінги `AuthServiceInterface`, `ModelBuilderInterface` та `EventDispatcherInterface` будуть зареєстровані без будь-ких додаткових дій.

У конфігу вкажіть дані для авторизації та класи моделей:

```php
$configValues = [
    'import_dir' => storage_path('1c_exchange'),
    'auth' => [
        'login'    => 'admin',
        'password' => 'secret',
        'custom'   => false,
    ],
    'use_zip'    => false,
    'file_part'  => 0,
    'models'     => [
        \Bigperson\Exchange1C\Interfaces\GroupInterface::class   => \App\Models\Category::class,
        \Bigperson\Exchange1C\Interfaces\ProductInterface::class => \App\Models\Product::class,
        \Bigperson\Exchange1C\Interfaces\OfferInterface::class   => \App\Models\Offer::class,
    ],
];
$config = new \Bigperson\Exchange1C\Config($configValues);
```

Отримайте `CatalogService` з контейнера (Laravel вирішить всі залежності автоматично):

```php
app()->bind(\Bigperson\Exchange1C\Config::class, fn() => $config);

$catalogService = app(\Bigperson\Exchange1C\Services\CatalogService::class);
```

Якщо вам потрібно перевизначити реалізацію будь-якого інтерфейсу (наприклад, використовувати власний `AuthService`), додайте біндінг в `AppServiceProvider`:

```php
$this->app->bind(
    \Bigperson\Exchange1C\Interfaces\AuthServiceInterface::class,
    \App\Services\MyCustomAuthService::class
);
```

# Використання без Laravel (ручне збирання)

```php
require_once './../vendor/autoload.php';

$configValues = [
    'import_dir' => '1c_exchange',
    'auth' => [
        'login'    => 'admin',
        'password' => 'admin',
        'custom'   => false,
    ],
    'use_zip'    => false,
    'file_part'  => 0,
    'models'     => [
        \Bigperson\Exchange1C\Interfaces\GroupInterface::class   => \App\Models\Category::class,
        \Bigperson\Exchange1C\Interfaces\ProductInterface::class => \App\Models\Product::class,
        \Bigperson\Exchange1C\Interfaces\OfferInterface::class   => \App\Models\Offer::class,
    ],
];
$config      = new \Bigperson\Exchange1C\Config($configValues);
$request     = \Symfony\Component\HttpFoundation\Request::createFromGlobals();
$dispatcher  = new \Your\EventDispatcher\Implementation(); // реалізує EventDispatcherInterface
$modelBuilder = new \Bigperson\Exchange1C\ModelBuilder();

$authService     = new \Bigperson\Exchange1C\Services\AuthService($config);
$loaderService   = new \Bigperson\Exchange1C\Services\FileLoaderService($config);
$categoryService = new \Bigperson\Exchange1C\Services\CategoryService($config, $dispatcher, $modelBuilder);
$offerService    = new \Bigperson\Exchange1C\Services\OfferService($config, $dispatcher, $modelBuilder);
$catalogService  = new \Bigperson\Exchange1C\Services\CatalogService($config, $authService, $loaderService, $categoryService, $offerService);

$mode = $request->get('mode');
$type = $request->get('type');

try {
    if ($type === 'catalog') {
        if (!method_exists($catalogService, $mode)) {
            throw new \Exception('not correct request, mode=' . $mode);
        }
        $body     = $catalogService->$mode($request);
        $response = new \Symfony\Component\HttpFoundation\Response($body, 200, ['Content-Type', 'text/plain']);
        $response->send();
    } else {
        throw new \LogicException(sprintf('Logic for type "%s" not implemented', $type));
    }
} catch (\Exception $e) {
    $body  = "failure\n";
    $body .= $e->getMessage() . "\n";
    $body .= $e->getFile() . "\n";
    $body .= $e->getLine() . "\n";

    $response = new \Symfony\Component\HttpFoundation\Response($body, 500, ['Content-Type', 'text/plain']);
    $response->send();
}
```

Більш детальну інформацію про інтерфейси та їх реалізацію можна знайти в документації https://github.com/carono/yii2-1c-exchange

# Ліцензія
Цей пакет є відкритим кодом під ліцензією [MIT license](LICENSE).




