# Contributing to Zemail PHP SDK

Thank you for considering contributing to the Zemail PHP SDK! We welcome contributions from the community to help make this project better.

## How to Contribute

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin feature/my-new-feature`)
5. Create a new Pull Request

## Development Setup

1. Clone your fork locally.
2. Run `composer install` to install dependencies.
3. Make your changes in the `src/` directory.
4. Add or update tests in the `tests/` directory as appropriate.

## Testing

Please make sure that all tests pass before submitting a pull request:

```bash
composer test
```

## Static Analysis

We use PHPStan for static analysis. Please ensure there are no errors:

```bash
composer test:types
```

## Code Style

We follow the PSR-12 coding standard. You can format your code automatically using Laravel Pint:

```bash
composer format
```

## Need Help?

If you have any questions or need help with your contribution, please feel free to open an issue or reach out to us at support@zemail.me.
