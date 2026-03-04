# sec-scan Framework Configs

Framework-specific configuration for the [sec-scan](https://sec-scan.ai) PHP security scanner.

## How it works

The sec-scan client auto-detects the PHP framework from `composer.lock` / `composer.json` and fetches default configuration from the sec-scan API server. The server uses this repository as its source of truth.

Default excludes prevent the scanner from analyzing auto-generated files (compiled templates, framework caches, proxy classes) that would produce false positives.

## File format

`frameworks.json` maps framework identifiers to their configuration:

```json
{
  "Shopware 6": {
    "default_excludes": [
      "var/cache",
      "public/theme",
      "public/bundles"
    ]
  }
}
```

### Fields

- `default_excludes` - Directories excluded from scanning (relative to scan root). These contain auto-generated PHP files that are not user code.

## Framework identifiers

The client detects these frameworks and uses the exact string as the lookup key:

| Identifier | Detection source |
|---|---|
| `Shopware 5` | `shopware/*` packages without `shopware/core` or `shopware/platform` |
| `Shopware 6` | `shopware/core` or `shopware/platform` |
| `OXID eShop 6.x` | `oxid-esales/*` packages, version < 7 |
| `OXID eShop 7.x` | `oxid-esales/oxideshop-ce` version 7+ |
| `WordPress/WooCommerce` | `woocommerce/*` or `wordpress*` packages |
| `Magento` | `magento/*` packages |
| `JTL-Shop 5` | `jtl-shop/*` or `jtl/*` packages |
| `PrestaShop` | `prestashop/prestashop` |
| `Sylius` | `sylius/sylius` |
| `Laravel` | `laravel/framework` |
| `Symfony` | `symfony/framework-bundle` or `symfony/symfony` |

## What gets excluded and why

These directories contain **auto-generated PHP files** that look suspicious to a security scanner but are harmless framework output:

- **Compiled templates** - Twig, Smarty, and Blade compile templates to PHP. These generated files often contain `eval`-like patterns.
- **DI container cache** - Symfony-based frameworks compile their dependency injection container to PHP classes.
- **Generated proxies** - Magento generates factory, interceptor, and proxy classes.
- **Route/config cache** - Cached PHP arrays for routes, config, and service definitions.

`vendor/` is intentionally **not** excluded - plugins and extensions installed via Composer contain project-specific code that should be scanned.

## Contributing

To add or update framework configs:

1. Fork this repository
2. Edit `frameworks.json`
3. Submit a pull request with details on which directories are auto-generated and why they should be excluded

When adding a new framework, also open an issue on the [client repository](https://github.com/sec-scan-ai/client) to add detection support.
