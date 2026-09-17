# Contributing

Discuss focused changes in the [issue tracker](https://github.com/altophp/importmap/issues).
The library supports PHP 8.3; its current test tooling requires PHP 8.4 or later.
Install dependencies with `composer install`, then run
`vendor/bin/php-cs-fixer fix --dry-run --diff --sequential`, `composer sa`, and
`composer test`. `composer cs` applies fixes; review those changes before committing.

Include tests for exact and prefix mappings, scopes, null entries, rendering
escaping, and merge behavior when changing their contracts. Keep browser
integration claims separate from PHP resolver behavior.
