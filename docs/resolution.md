# Resolution

`NativeResolver` resolves a module specifier against an `ImportMap` and returns its address with any integrity metadata.

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Alto\ImportMap\ImportMap;
use Alto\ImportMap\Resolver\NativeResolver;

$map = new ImportMap();
$map
    ->add('app', '/assets/app.js', integrity: 'sha384-example')
    ->add('vendor/', '/assets/vendor/')
    ->add('app', '/assets/admin.js', scope: '/admin/');

$resolver = new NativeResolver();

$global = $resolver->resolve('app', $map);
$dependency = $resolver->resolve('vendor/router.js', $map);
$scoped = $resolver->resolve('app', $map, '/admin/dashboard.js');

echo $global['url'].PHP_EOL;
echo $global['integrity'].PHP_EOL;
echo $dependency['url'].PHP_EOL;
echo $scoped['url'].PHP_EOL;
```

Output:

```text
/assets/app.js
sha384-example
/assets/vendor/router.js
/assets/admin.js
```

`resolve(string $specifier, ImportMap $map, ?string $referencingUrl = null): array` returns an array with `url` and nullable `integrity` keys.

Resolution proceeds in this order:

1. Matching scopes, from the longest scope prefix to the shortest.
2. Top-level imports.
3. Passthrough for absolute URLs, root-relative paths, `./` paths, and `../` paths.

Within a scope or the top-level map, an exact match takes priority. Prefix entries ending with `/` use the longest matching prefix and append the remainder of the specifier to the mapped address. When a matching scope does not contain the specifier, resolution continues to shorter scopes and then the top-level imports.

## Failures and blocked entries

```php
<?php

require __DIR__.'/vendor/autoload.php';

use Alto\ImportMap\Exception\ResolutionFailedException;
use Alto\ImportMap\ImportMap;
use Alto\ImportMap\Resolver\NativeResolver;

$map = new ImportMap(imports: ['legacy' => null]);

try {
    (new NativeResolver())->resolve('legacy', $map);
} catch (ResolutionFailedException $exception) {
    echo $exception->getMessage();
}
```

Output:

```text
Resolution blocked by null entry for 'legacy'.
```

An exact null mapping throws `ResolutionFailedException`. An unmatched bare specifier throws its `SpecifierNotFoundException` subclass. Catch `ResolutionFailedException` when both outcomes require the same handling.

For a missing bare specifier, check its spelling and add the intended mapping.
For a blocked mapping, remove or replace the deliberate `null` entry only when
the application should permit that dependency. Check the referencing URL when
a scope is not selected; the longest matching scope takes precedence.

The PHP resolver returns configured addresses or URL-like input. It does not
fetch modules, resolve relative URLs against a browser document, or prove that a
URL exists. The browser resolves module URLs in its own document context.

Custom resolvers implement the same `ResolverInterface::resolve()` contract and may throw package exceptions implementing `ExceptionInterface`.
