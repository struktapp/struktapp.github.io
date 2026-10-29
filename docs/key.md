Strukt Key
===

Cryptography, signing, JWT, CSRF, and SSL utilities for PHP 8.2+, built on Strukt
Math, Console, and Fs.

No external crypto services, HTTP calls, or bundled private keys are required.
Keys are generated in memory and written only where requested.

## Install

```sh
composer require strukt/key
```

The package installs `vendor/bin/kcli`.

## CLI

```text
hash            Compute a message digest
bcrypt          Hash a password with bcrypt
bcrypt:check    Check a password against a bcrypt digest
keys:new        Generate an RSA key pair on disk
enc/dec         AES encrypt / decrypt
sign/verify     Sign / verify data with an RSA key
cert:selfsign   Generate a key pair and self-signed certificate
cert:verify     Verify a certificate
cert:xpub       Extract the public key from a certificate
cert:info       Read certificate fields
jwt:encode      Mint a JSON web token
jwt:decode      Decode and validate a JSON web token
envelope:close  Seal a message for a set of recipients
envelope:open   Open a sealed message with a private key
csrf            Mint a CSRF token
csrf:check      Validate a CSRF token
```

Every applicable command supports `--json`.

```bash
./vendor/bin/kcli hash "hello world"
./vendor/bin/kcli bcrypt "s3cret!" --cost=12
./vendor/bin/kcli bcrypt:check "s3cret!" '$2y$12$...'

./vendor/bin/kcli keys:new --dir keys --name alice
./vendor/bin/kcli sign "the document" --key keys/alice.pem
./vendor/bin/kcli verify "the document" "$SIG" --key keys/alice.pub.pem

./vendor/bin/kcli enc "top secret" "a-strong-key" --json
./vendor/bin/kcli dec "$CIPHERTEXT" "a-strong-key"

./vendor/bin/kcli cert:selfsign --dir keys --name web --common strukt.example --days 365
./vendor/bin/kcli cert:info keys/cert.pem
./vendor/bin/kcli cert:xpub keys/cert.pem --json

./vendor/bin/kcli jwt:encode '{"role":"admin"}' "jwt-secret"
./vendor/bin/kcli jwt:decode "$TOKEN" "jwt-secret"
./vendor/bin/kcli csrf '{"user":42}' "csrf-key"
./vendor/bin/kcli csrf:check "$TOKEN" "csrf-key"

./vendor/bin/kcli envelope:close "letter" "ca.pem,another.pem" --json
./vendor/bin/kcli envelope:open "$ENV_KEY" "$SEALED" "$IV" --key alice.pem
```

Input can be piped or read from a file with `--file`:

```bash
cat message.txt | vendor/bin/kcli hash
./vendor/bin/kcli enc --file document.txt "a-strong-key"
```

## Hashing

```php
use Strukt\Hash\Bcrypt;
use Strukt\Hash\Sha;

$hash = (new Bcrypt())->makeHash('p@55w0rd');
(new Bcrypt())->verify('p@55w0rd', $hash); // true

Sha::once256('abc');
Sha::dbl256('abc');
Sha::hmac('key', 'data', 'sha256'); // raw binary
```

## Symmetric encryption

```php
use Strukt\Codec;

$codec = Codec::make('a-strong-key'); // fresh random IV per message
$wrapped = $codec->encode($plain);    // iv-hex:base64
$codec->decode($wrapped);
```

## JWTs

```php
use Strukt\Jwt;

$signer = Jwt::make([
    'secret' => 'jwt-secret',
    'algo' => 'HS256',
    'expire' => 3600,
]);

$token = $signer->encode(['role' => 'admin']);
$claims = $signer->decode($token); // stdClass|null
$signer->valid($token);            // bool
```

## CSRF

```php
use Strukt\Csrf;

$token = Csrf::make(['user' => 42], 'csrf-key', 300)->encode();
Csrf::valid($token, 'csrf-key'); // true
```

## Key pairs, signing, and ciphering

```php
use Strukt\Ssl\KeyPairBuilder;
use Strukt\Ssl\Signer;
use Strukt\Ssl\Cipher;

$pair = KeyPairBuilder::make('keys/', 'alice')->makeKeyPair();
$pair->makePrivateKeyFile();
$pair->makePublicKeyFile();

$signer = new Signer($pair);
$signature = $signer->create('the document');
$signer->check('the document', $signature); // true

$cipher = new Cipher($pair);
$cipher->decrypt($cipher->encrypt('classified'));
```

## Envelopes and recipients

```php
use Strukt\Ssl\Envelope;

[$envKeys, $sealed, $iv] = Envelope::withAlgo()
    ->useCerts([$certPath])
    ->closeWithIv($message);

Envelope::withAlgo($iv)
    ->usePrivKey($privateKey)
    ->open($envKeys[0], $sealed);
```

## Certificates and CSRs

```php
use Strukt\Ssl\Request;
use Strukt\Ssl\Certificate\Parser;
use Strukt\Ssl\Csr\UniqueName;

$request = new Request($pair, new UniqueName(['common' => 'strukt.example']));
$request->selfSign(365);
$request->verify();

$parser = new Parser($pem); // or Cert::withPath()->parser()
$parser->getDomain();
$parser->getIssuer();
$parser->isExpired(); // bool
```

The legacy facade remains:

```php
use Strukt\Ssl\All;

$tool = All::makeKeys();
$tool = All::keyPath('keys/alice.pem');
$tool->toSend($message);
$tool->useCsr(['common' => 'x'])->selfSign();
```

## Global helpers

```text
codec($key, $cipher, $vector?)       Codec
bcry($password, $cost?)              encode()/verify()
sha256($input)                       hex digest
sha256dbl($input)                    double hex digest
csrf($data, $key, $ttl?)             encode()/decode()/valid()
jwt($data, $secret, $algo?, $ttl?)   token or valid()/yield()
ssl($arg?)                           All facade
keypair($privateKeyPath?)            KeyPair
pubkey($path)                        PublicKey
```

Secrets are explicit parameters, never pulled from a global configuration file.

## Notes

- `Bcrypt` uses native `password_hash()` and `password_verify()`.
- `Jwt` is self-contained and uses `hash_hmac` with constant-time comparison.
- `Codec` carries a fresh IV in each encoded message.
- Certificate parsing returns booleans and `DateTime` values.
- OpenSSL errors surface as exceptions.
- Default key generation uses 2048-bit RSA and the bundled OpenSSL config.
