---
name: php-coding-style
description: Apply personal PHP coding standards and architectural preferences when writing, reviewing, or refactoring PHP code. Use this skill whenever the user asks to write PHP code, create a PHP class/service/repository, scaffold a project, or review PHP for style. Also trigger when the user says "write me a PHP...", "create a PHP class/interface/service", "scaffold a PHP project", or any PHP code generation task, even if they don't explicitly mention style. This skill ensures all output matches the user's exact personal preferences for formatting, architecture, tooling, and documentation.
---

# PHP Coding Style: Personal Preferences

This skill encodes the exact PHP coding standards and architectural preferences to apply
whenever writing, generating, or reviewing PHP code for this user.

**Always apply this skill when generating any PHP code, no exceptions.**

---

## Code Style

### Baseline Rules
- **Standard**: PSR-12 strict, always
- **PHP version**: latest stable, verified live before pinning. Use all modern features available
- **strict_types**: `declare(strict_types=1);` at the top of **every single file**, no exceptions
- **Line length**: 80 characters (PSR-2 classic)

Every file starts like this, no exceptions:

```php
<?php

declare(strict_types=1);

namespace App\Domain\Order;
```

### Type Declarations: Everywhere
- Native types on **all** properties, parameters, and return types
- Use `?Type` for nullable, never omit nullability
- Use `never` return type for methods that always throw
- Use union types `int|string` where appropriate
- Use `readonly` properties for Value Objects and immutable data
- Use constructor property promotion (PHP 8.1+) for clean DI

```php
// CORRECT: full type coverage, constructor promotion
final class OrderService
{
    public function __construct(
        private readonly OrderRepository $repository,
        private readonly LoggerInterface $logger,
    ) {}

    public function findById(string $orderId): ?Order
    {
        return $this->repository->findById($orderId);
    }

    public function cancel(string $orderId, string $reason): Result
    {
        // ...
    }
}

// WRONG: missing types, no promotion
class OrderService
{
    private $repository;

    public function findById($orderId)
    {
        // ...
    }
}
```

### PHPDoc: Full Coverage
- Every public and protected method gets a full PHPDoc block
- Always include `@param`, `@return`, `@throws`
- Use PHPDoc for complex array shapes and generics native types can't express
- Private methods get PHPDoc if non-obvious
- Never write PHPDoc that just restates the signature, add meaning

```php
/**
 * Process a pending order and transition it to confirmed state.
 *
 * Validates stock availability before confirming. If stock is
 * insufficient, returns a failure Result without throwing.
 *
 * @param string $orderId The unique order identifier.
 * @param array<string, int> $stockLevels Current stock levels by SKU.
 *
 * @return Result<Order, OrderError>
 *
 * @throws RepositoryException If the database connection fails.
 */
public function processOrder(
    string $orderId,
    array $stockLevels,
): Result {
    // ...
}
```

---

## Project Structure

This is the canonical structure for every PHP project. Always follow this layout.

```
project-root/
├── app/
│   ├── bootstrap.php               ← wires DI, .env, Twig, Monolog, Router
│   └── ApplicationName/            ← one named app dir per project
│       ├── Kernel.php              ← request lifecycle + middleware + routing
│       ├── Controllers/
│       │   └── DefaultController.php
│       ├── Models/
│       │   └── BaseModel.php
│       └── Services/
│
├── cli/
│   └── console.php                 ← Symfony Console entry point
│
├── config/
│   └── container.php               ← PHP-DI container definitions
│
├── public/
│   └── index.php                   ← true front controller, calls bootstrap.php
│
├── src/
│   ├── Controllers/                ← request handlers
│   ├── Models/                     ← domain models
│   ├── Views/                      ← Twig templates
│   │   ├── layouts/                ← base layout templates
│   │   ├── partials/               ← reusable template fragments
│   │   └── pages/                  ← full page templates
│   ├── Services/                   ← business logic
│   └── Helpers/                    ← stateless utility functions
│
├── storage/
│   └── uploads/                    ← user uploads and file storage
│
├── var/
│   ├── cache/                      ← Twig compiled cache, app cache
│   └── logs/                       ← Monolog log files
│
├── .env                            ← local environment (gitignored)
├── .env.example                    ← committed template
└── composer.json
```

### Key Rules
- `app/` is framework bootstrap only, no domain logic here
- `src/` is where all domain code lives, Controllers, Models, Views, Services, Helpers
- `app/ApplicationName/` directory name changes per project, always one application
- `public/` is the only web-accessible directory, nothing else exposed
- `var/` is runtime-generated only, always gitignored
- `storage/` for persistent user files, gitignore contents, not the directory

---

## Bootstrap Flow

The request lifecycle always follows this exact chain:

```
public/index.php
    ↓ requires app/bootstrap.php
    ↓ returns configured $app (Kernel instance)
app/bootstrap.php
    ↓ loads .env via phpdotenv
    ↓ builds PHP-DI container from config/container.php
    ↓ configures Monolog
    ↓ configures Twig
    ↓ instantiates Kernel with container
    ↓ returns Kernel
public/index.php
    ↓ calls $app->handle()
app/ApplicationName/Kernel.php
    ↓ runs middleware pipeline
    ↓ matches route via AltoRouter
    ↓ resolves controller from DI container
    ↓ dispatches to controller action
    ↓ sends Response
```

### public/index.php
```php
<?php

declare(strict_types=1);

require_once __DIR__ . '/../vendor/autoload.php';

$app = require_once __DIR__ . '/../app/bootstrap.php';
$app->handle();
```

### app/bootstrap.php
```php
<?php

declare(strict_types=1);

use DI\ContainerBuilder;
use Dotenv\Dotenv;

$dotenv = Dotenv::createImmutable(__DIR__ . '/..');
$dotenv->load();
$dotenv->required([
    'APP_ENV',
    'APP_NAME',
    'LOG_LEVEL',
]);

$builder = new ContainerBuilder();
$builder->addDefinitions(__DIR__ . '/../config/container.php');

if ($_ENV['APP_ENV'] === 'production') {
    $builder->enableCompilation(__DIR__ . '/../var/cache');
}

$container = $builder->build();

return $container->get(\App\ApplicationName\Kernel::class);
```

### app/ApplicationName/Kernel.php
```php
<?php

declare(strict_types=1);

namespace App\ApplicationName;

use AltoRouter;
use Psr\Log\LoggerInterface;

final class Kernel
{
    private AltoRouter $router;

    /**
     * @throws \Exception If router configuration fails.
     */
    public function __construct(
        private readonly \DI\Container $container,
        private readonly LoggerInterface $logger,
    ) {
        $this->router = new AltoRouter();
        $this->registerRoutes();
    }

    /**
     * Handle the incoming HTTP request and dispatch to controller.
     */
    public function handle(): void
    {
        $match = $this->router->match();

        // AltoRouter returns false on no match: not null
        if ($match === false) {
            http_response_code(404);
            return;
        }

        [$controllerClass, $method] = $match['target'];
        $controller = $this->container->get($controllerClass);
        $controller->$method($match['params']);
    }

    /**
     * Register all application routes.
     */
    private function registerRoutes(): void
    {
        $this->router->map(
            'GET',
            '/',
            [\App\Controllers\DefaultController::class, 'index'],
        );
    }
}
```

---

## Standard Dependencies

Every project ships with these in `composer.json`:

> Version constraints below are placeholders. Before pinning, verify the latest stable release of PHP on php.net and of each package on packagist.org.

```json
{
    "require": {
        "php": "<latest-stable>",
        "php-di/php-di": "<latest-stable>",
        "monolog/monolog": "<latest-stable>",
        "altorouter/altorouter": "<latest-stable>",
        "vlucas/phpdotenv": "<latest-stable>",
        "guzzlehttp/guzzle": "<latest-stable>",
        "symfony/console": "<latest-stable>",
        "twig/twig": "<latest-stable>"
    },
    "require-dev": {
        "phpunit/phpunit": "<latest-stable>",
        "phpstan/phpstan": "<latest-stable>",
        "squizlabs/php_codesniffer": "<latest-stable>"
    },
    "autoload": {
        "psr-4": {
            "App\\": "src/",
            "App\\ApplicationName\\": "app/ApplicationName/"
        }
    }
}
```

### config/container.php: PHP-DI Definitions
```php
<?php

declare(strict_types=1);

use Monolog\Handler\StreamHandler;
use Monolog\Logger;
use Psr\Log\LoggerInterface;
use Twig\Environment;
use Twig\Loader\FilesystemLoader;

return [
    LoggerInterface::class => function (): LoggerInterface {
        $logger = new Logger($_ENV['APP_NAME']);
        $logger->pushHandler(
            new StreamHandler(
                __DIR__ . '/../var/logs/app.log',
                $_ENV['LOG_LEVEL'] ?? 'debug',
            )
        );
        return $logger;
    },

    Environment::class => function (): Environment {
        $loader = new FilesystemLoader(__DIR__ . '/../src/Views');
        return new Environment($loader, [
            'cache' => $_ENV['APP_ENV'] === 'production'
                ? __DIR__ . '/../var/cache/twig'
                : false,
            'debug' => $_ENV['APP_ENV'] !== 'production',
        ]);
    },
];
```

---

## Views: Twig

- All templates live in `src/Views/`
- Organized into `layouts/`, `partials/`, `pages/`
- Always extend a base layout, never standalone full-page templates
- Controllers never echo raw HTML, always render through Twig

```
src/Views/
├── layouts/
│   └── base.html.twig          ← master layout, defines blocks
├── partials/
│   ├── nav.html.twig
│   └── footer.html.twig
└── pages/
    ├── home.html.twig
    └── order/
        ├── index.html.twig
        └── detail.html.twig
```

```twig
{# layouts/base.html.twig #}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}App{% endblock %}</title>
</head>
<body>
    {% include 'partials/nav.html.twig' %}
    {% block content %}{% endblock %}
    {% include 'partials/footer.html.twig' %}
</body>
</html>

{# pages/home.html.twig #}
{% extends 'layouts/base.html.twig' %}

{% block title %}Home{% endblock %}

{% block content %}
    <h1>{{ title }}</h1>
{% endblock %}
```

```php
// Controller renders via Twig: never echo directly
final class DefaultController
{
    public function __construct(
        private readonly Environment $twig,
    ) {}

    /**
     * @param array<string, string> $params Route parameters.
     */
    public function index(array $params): void
    {
        echo $this->twig->render('pages/home.html.twig', [
            'title' => 'Welcome',
        ]);
    }
}
```

---

## CLI: Symfony Console

- All CLI commands live in `cli/`
- Entry point is `cli/console.php`
- Command classes live in `src/Commands/`
- Wired through the DI container, constructor injected like everything else

```php
// cli/console.php
<?php

declare(strict_types=1);

require_once __DIR__ . '/../vendor/autoload.php';

$app = require_once __DIR__ . '/../app/bootstrap.php';

$console = new \Symfony\Component\Console\Application(
    $_ENV['APP_NAME'],
    '1.0.0',
);
$console->add($app->getContainer()->get(\App\Commands\ProcessOrdersCommand::class));
$console->run();
```

```php
// src/Commands/ProcessOrdersCommand.php
<?php

declare(strict_types=1);

namespace App\Commands;

use Psr\Log\LoggerInterface;
use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

final class ProcessOrdersCommand extends Command
{
    protected static string $defaultName = 'orders:process';

    public function __construct(
        private readonly OrderService $orders,
        private readonly LoggerInterface $logger,
    ) {
        parent::__construct();
    }

    protected function configure(): void
    {
        $this->setDescription('Process all pending orders.');
    }

    /**
     * @return int Command::SUCCESS or Command::FAILURE
     */
    protected function execute(
        InputInterface $input,
        OutputInterface $output,
    ): int {
        $this->logger->info('Starting order processing batch');

        try {
            $this->orders->processPending();
            return Command::SUCCESS;
        } catch (\Throwable $e) {
            $this->logger->error(
                'Order processing failed',
                ['exception' => $e]
            );
            return Command::FAILURE;
        }
    }
}
```

---

## Architecture

### Design Philosophy: Full OOP
- Everything is a class, no standalone functions in domain or application layers
- `final` by default on concrete classes, opt out explicitly when inheritance is intentional
- Single Responsibility strictly followed, small, focused classes
- Constructor injection always, no setter injection, no property injection
- Never use `new` inside domain or application classes, only in factories and infrastructure

### Error Handling: Mixed Strategy

```php
// Custom exception hierarchy
class DomainException extends \RuntimeException {}
class InfrastructureException extends \RuntimeException {}

final class OrderNotFoundException extends DomainException
{
    public function __construct(string $orderId)
    {
        parent::__construct(
            sprintf('Order not found: %s', $orderId)
        );
    }
}

// Result for recoverable failures: never throw for expected business outcomes
final readonly class Result
{
    private function __construct(
        private readonly mixed $value,
        private readonly mixed $error,
        private readonly bool $success,
    ) {}

    public static function ok(mixed $value): self
    {
        return new self(value: $value, error: null, success: true);
    }

    public static function fail(mixed $error): self
    {
        return new self(value: null, error: $error, success: false);
    }

    public function isOk(): bool
    {
        return $this->success;
    }

    /**
     * @throws \LogicException If called on a failed Result.
     */
    public function unwrap(): mixed
    {
        if (!$this->success) {
            throw new \LogicException('Cannot unwrap a failed Result');
        }
        return $this->value;
    }

    public function error(): mixed
    {
        return $this->error;
    }
}
```

### Logging: Monolog via PSR-3
- Always inject `LoggerInterface`, never depend on Monolog directly
- Use context arrays, never interpolate variables into message strings
- Logs go to `var/logs/app.log`

```php
// CORRECT: structured context array
$this->logger->error(
    'Failed to process order',
    ['order_id' => $orderId, 'exception' => $e]
);

// WRONG: interpolated, unstructured
$this->logger->error(
    "Failed to process order $orderId: " . $e->getMessage()
);
```

### Constants & Configuration
- No hardcoded constants anywhere in domain or application code
- All config from `.env`, loaded by phpdotenv in `bootstrap.php`
- Never access `$_ENV` directly in domain code, always through a typed config class

```php
final readonly class AppConfig
{
    public function __construct(
        public readonly string $appName,
        public readonly string $appEnv,
        public readonly string $logLevel,
    ) {}

    /**
     * @throws \RuntimeException If a required variable is missing.
     */
    public static function fromEnvironment(): self
    {
        return new self(
            appName: self::require('APP_NAME'),
            appEnv: self::require('APP_ENV'),
            logLevel: $_ENV['LOG_LEVEL'] ?? 'debug',
        );
    }

    private static function require(string $key): string
    {
        $value = $_ENV[$key] ?? null;
        if ($value === null) {
            throw new \RuntimeException(
                sprintf('Missing required env variable: %s', $key)
            );
        }
        return $value;
    }
}
```

---

## Comments

- **Minimal**: clean, well-named code is self-documenting
- Inline comments only for non-obvious logic, external quirks, or business rule rationale
- Never comment what the code does, comment **why** when it's not obvious

```php
// CORRECT: explains the why
// AltoRouter returns false on no match, not null, explicit check required
if ($match === false) {
    http_response_code(404);
    return;
}

// WRONG: restates the code
// Check if match is false
if ($match === false) {
```

---

## Testing: PHPUnit

- Every class has a corresponding test in `tests/`
- Test behavior, not implementation
- Mock at the boundaries, repositories, HTTP clients, external APIs
- Never mock domain logic or Value Objects, instantiate directly
- Use data providers for data-driven cases

```php
final class OrderServiceTest extends TestCase
{
    private MockObject&OrderRepository $repository;
    private OrderService $service;

    protected function setUp(): void
    {
        $this->repository = $this->createMock(OrderRepository::class);
        $this->service = new OrderService(
            $this->repository,
            new NullLogger(),
        );
    }

    public function testReturnsNullWhenOrderNotFound(): void
    {
        $this->repository->method('findById')->willReturn(null);

        $result = $this->service->findById('non-existent');

        self::assertNull($result);
    }
}
```

---

## Toolchain

| Tool | Purpose | Config |
|---|---|---|
| `phpcs` | PSR-12 enforcement | `phpcs.xml` |
| `phpstan` | Static analysis, level 9 | `phpstan.neon` |
| `pre-commit` | Enforce on every commit | `.pre-commit-config.yaml` |
| `phpunit` | Testing | `phpunit.xml` |

### phpcs.xml
```xml
<?xml version="1.0"?>
<ruleset name="PSR12">
    <rule ref="PSR12"/>
    <arg name="colors"/>
    <arg value="sp"/>
    <file>src</file>
    <file>app</file>
    <file>tests</file>
</ruleset>
```

### phpstan.neon
```neon
parameters:
    level: 9
    paths:
        - src
        - app
        - tests
```

### .pre-commit-config.yaml
```yaml
repos:
  - repo: local
    hooks:
      - id: phpcs
        name: PHP CodeSniffer
        entry: vendor/bin/phpcs
        language: script
        files: \.php$
      - id: phpstan
        name: PHPStan
        entry: vendor/bin/phpstan analyse
        language: script
        files: \.php$
      - id: phpunit
        name: PHPUnit
        entry: vendor/bin/phpunit --stop-on-failure
        language: script
        pass_filenames: false
```
