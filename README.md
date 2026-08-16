# Zemail PHP SDK

[![Latest Version on Packagist](https://img.shields.io/packagist/v/zemailme/zemail-php.svg?style=flat-square)](https://packagist.org/packages/zemailme/zemail-php)
[![GitHub Tests Action Status](https://img.shields.io/github/actions/workflow/status/zemailme/zemail-php/tests.yml?branch=main&label=tests&style=flat-square)](https://github.com/zemailme/zemail-php/actions?query=workflow%3ATests+branch%3Amain)
[![PHP Version](https://img.shields.io/packagist/php-v/zemailme/zemail-php.svg?style=flat-square)](https://packagist.org/packages/zemailme/zemail-php)
[![License](https://img.shields.io/github/license/zemailme/zemail-php?style=flat-square)](https://github.com/zemailme/zemail-php/blob/main/LICENSE)

The official PHP SDK for the [Zemail Developer API](https://zemail.me/api-docs). Create and manage temporary mailboxes, receive emails, and handle attachments programmatically.

---

## Requirements

- PHP 8.2 or higher
- Guzzle HTTP client (`guzzlehttp/guzzle: ^7.8`)
- `ext-json`

---

## Installation

Install the package via Composer:

```bash
composer require zemailme/zemail-php
```

---

## Quickstart

Initialize the `Client` with your API key:

```php
use Zemail\Client;

$client = new Client('zm_live_your_api_key_here');
```

You can optionally specify an API version or custom Guzzle client options (e.g. timeout):

```php
$client = new Client(
    apiKey: 'zm_live_your_api_key_here',
    version: '2026-04-23',
    guzzleOptions: [
        'timeout' => 10.0,
        'headers' => [
            'X-Custom-Header' => 'value',
        ],
    ]
);
```

---

## Usage

### 1. Account & Subscription

Access your account profile, active plan, and API/mailbox usage limits:

```php
// Get account profile
$account = $client->account()->get();
echo "Account ID: {$account->id}, Email: {$account->email}, Tier: {$account->tier}\n";

// Get active subscription
$subscription = $client->account()->subscription();
echo "Status: {$subscription->status}, Tier: {$subscription->tier}\n";

// Get current resource & API usage
$usage = $client->account()->usage();
print_r($usage->mailboxes);
print_r($usage->storage);
print_r($usage->developerApi);
```

---

### 2. Domains

List available domains for mailbox creation:

```php
$domains = $client->domains()->list();

foreach ($domains->data as $domain) {
    echo "Domain: {$domain->name} (Types: " . implode(', ', $domain->allowedTypes) . ")\n";
}
```

---

### 3. Mailboxes

#### List Mailboxes
```php
$mailboxes = $client->mailboxes()->list(page: 1, limit: 10);

foreach ($mailboxes->data as $mailbox) {
    echo "Mailbox: {$mailbox->address} (ID: {$mailbox->id})\n";
}

if ($mailboxes->hasMore) {
    echo "Next cursor: {$mailboxes->nextCursor}\n";
}
```

#### Create a Random Mailbox
```php
$mailbox = $client->mailboxes()->create([
    'type' => 'random',
]);

echo "Created random mailbox: {$mailbox->address}\n";
```

#### Create a Custom Mailbox
```php
$mailbox = $client->mailboxes()->create([
    'type' => 'custom',
    'domain' => 'zemail.me',
    'username' => 'my-inbox',
]);

echo "Created custom mailbox: {$mailbox->address}\n";
```

#### Get Mailbox Details
```php
$mailbox = $client->mailboxes()->get(123);
echo "Address: {$mailbox->address}, Unread emails: {$mailbox->unreadCount}\n";
```

#### Delete a Mailbox
```php
$deleted = $client->mailboxes()->delete(123);
// Returns true on success
```

---

### 4. Emails & Attachments

#### List Emails in a Mailbox
```php
// List recent emails with optional search query
$emails = $client->mailboxes()->emails()->list(
    mailboxId: $mailbox->id,
    page: 1,
    limit: 25,
    search: 'verification'
);

foreach ($emails->data as $email) {
    echo "[{$email->id}] From: {$email->sender} | Subject: {$email->subject}\n";
}
```

#### Get Full Email Details
```php
$email = $client->mailboxes()->emails()->get($mailbox->id, $emailId);

echo "Subject: {$email->subject}\n";
echo "Plain text body: {$email->bodyText}\n";
echo "HTML body: {$email->bodyHtml}\n";

// Inspect attachments
foreach ($email->attachments as $attachment) {
    echo "Attachment: {$attachment->name} ({$attachment->size} bytes)\n";
}
```

#### Mark Email as Read
```php
$isRead = $client->mailboxes()->emails()->markAsRead($mailbox->id, $emailId);
```

#### Get Temporary Attachment Download URL
```php
$download = $client->mailboxes()->emails()->getAttachmentDownloadUrl(
    mailboxId: $mailbox->id,
    emailId: $emailId,
    attachmentId: 'att_123'
);

echo "Download URL: {$download['url']}\n";
echo "Expires at: {$download['expires_at']}\n";
```

#### Delete an Email
```php
$deleted = $client->mailboxes()->emails()->delete($mailbox->id, $emailId);
// Returns true on success
```

---

## Error Handling

All SDK exceptions inherit from `Zemail\Exceptions\ZemailException`:

| Exception | HTTP Status | Description |
|---|---|---|
| `AuthenticationException` | `401`, `403` | Invalid API key or unauthorized access |
| `NotFoundException` | `404` | Resource (mailbox, email, domain) not found |
| `ValidationException` | `422` | Request validation failure (includes `$e->errors`) |
| `RateLimitException` | `429` | Daily or concurrency rate limit reached |
| `ZemailException` | Any | Generic SDK/API exception |

```php
use Zemail\Exceptions\AuthenticationException;
use Zemail\Exceptions\NotFoundException;
use Zemail\Exceptions\RateLimitException;
use Zemail\Exceptions\ValidationException;
use Zemail\Exceptions\ZemailException;

try {
    $mailbox = $client->mailboxes()->create(['type' => 'custom']);
} catch (ValidationException $e) {
    echo "Validation failed: " . $e->getMessage() . "\n";
    print_r($e->errors);
} catch (AuthenticationException $e) {
    echo "Auth error: " . $e->getMessage() . "\n";
} catch (RateLimitException $e) {
    echo "Rate limited: " . $e->getMessage() . "\n";
} catch (NotFoundException $e) {
    echo "Not found: " . $e->getMessage() . "\n";
} catch (ZemailException $e) {
    echo "General error: " . $e->getMessage() . "\n";
}
```

---

## Development & Testing

Run unit & feature tests with Pest:

```bash
composer test
```

Run static analysis with PHPStan:

```bash
composer test:types
```

Format code with Laravel Pint:

```bash
composer format
```

---

## License

The MIT License (MIT). Please see [License File](LICENSE) for more information.