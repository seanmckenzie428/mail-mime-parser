# zbateson/mail-mime-parser

Testable and PSR-compliant mail mime parser alternative to PHP's imap* functions and Pear libraries for reading messages in _Internet Message Format_ [RFC 822](http://tools.ietf.org/html/rfc822) (and later revisions [RFC 2822](http://tools.ietf.org/html/rfc2822), [RFC 5322](http://tools.ietf.org/html/rfc5322)).

[![Build Status](https://github.com/zbateson/mail-mime-parser/actions/workflows/tests.yml/badge.svg)](https://github.com/zbateson/mail-mime-parser/actions/workflows/tests.yml)
[![Code Coverage](https://scrutinizer-ci.com/g/zbateson/mail-mime-parser/badges/coverage.png?b=master)](https://scrutinizer-ci.com/g/zbateson/mail-mime-parser/?branch=master)
[![Scrutinizer Code Quality](https://scrutinizer-ci.com/g/zbateson/mail-mime-parser/badges/quality-score.png?b=master)](https://scrutinizer-ci.com/g/zbateson/mail-mime-parser/?branch=master)
[![Total Downloads](https://poser.pugx.org/zbateson/mail-mime-parser/downloads)](//packagist.org/packages/zbateson/mail-mime-parser)
[![Latest Stable Version](https://poser.pugx.org/zbateson/mail-mime-parser/v)](//packagist.org/packages/zbateson/mail-mime-parser)

The goals of this project are to be:

* Well written
* Standards-compliant but forgiving
* Tested where possible

To include it for use in your project, please install via composer:

```
composer require zbateson/mail-mime-parser
```

## Sponsors

A huge thank you to my first sponsor. <3

[![SecuMailer](https://mail-mime-parser.org/sponsors/logo-secumailer.png)](https://secumailer.com)

## Removal Notice (since 2.0.0)

getContentResourceHandle, getTextResourceHandle, and getHtmlResourceHandle have all been deprecated in 1.2.1 and removed in 2.0.0. fread() will only return a single byte of a multibyte char, and so will cause potentially unexpected results/warnings in some cases, and psr7 streams should be used instead. Note that getBinaryContentResourceHandle and getResourceHandle are still available.

## Change in 2.0

Starting in 2.0, the passed resource handle or stream must remain open while the returned message object is still in use.

Old code:
```php
$handle = fopen('file.mime', 'r');
$message = $mailParser->parse($handle);         // returns `Message`
fclose($handle);
```

New code:
```php
// attaches the resource handle to the returned `IMessage` if the second parameter
// is true.  The resource handle is closed when the IMessage is destroyed.
$message = $mailParser->parse(fopen('file.mime', 'r'), true);
```

For a more complete list of changes, please visit the [2.0 Upgrade Guide](https://mail-mime-parser.org/upgrade-2.0).

## Requirements

MailMimeParser requires PHP 5.4 or newer.  Tested on PHP 5.4, 5.5, 5.6, 7, 7.1, 7.2, 7.3, 7.4 and 8.0.

## Usage

```php
use ZBateson\MailMimeParser\MailMimeParser;
use ZBateson\MailMimeParser\Message;
use ZBateson\MailMimeParser\Header\HeaderConsts;

// use an instance of MailMimeParser as a class dependency
$mailParser = new MailMimeParser();

$handle = fopen('file.mime', 'r');
// parse() accepts a string, resource or Psr7 StreamInterface
// pass `true` as the second argument to attach the passed $handle and close
// it when the IMessage is destroyed.
$message = $mailParser->parse($handle, false);         // returns `IMessage`

// OR: use this procedurally (Message::from also accepts a string,
// resource or Psr7 StreamInterface
$message = Message::from($string);

echo $message->getHeaderValue(HeaderConsts::FROM);     // user@example.com
echo $message
    ->getHeader(HeaderConsts::FROM)                    // AddressHeader
    ->getPersonName();                                 // Person Name
echo $message->getHeaderValue(HeaderConsts::SUBJECT);  // The email's subject
echo $message
    ->getHeader(HeaderConsts::TO)                      // also AddressHeader
    ->getAddresses()[0]                                // AddressPart
    ->getName();                                       // Person Name
echo $message
    ->getHeader(HeaderConsts::CC)                      // also AddressHeader
    ->getAddresses()[0]                                // AddressPart
    ->getEmail();                                      // user@example.com

echo $message->getTextContent();                       // or getHtmlContent()

echo $message->getHeader('X-Foo');                     // for custom or undocumented headers

$att = $message->getAttachmentPart(0);                 // first attachment
echo $att->getHeaderValue(HeaderConsts::CONTENT_TYPE); // e.g. "text/plain"
echo $att->getHeaderParameter(                         // value of "charset" part
    'content-type',
    'charset'
);
echo $att->getContent();                               // get the attached file's contents
$stream = $att->getContentStream();                    // the file is decoded automatically
$dest = \GuzzleHttp\Psr7\stream_for(
    fopen('my-file.ext')
);
\GuzzleHttp\Psr7\copy_to_stream(
    $stream, $dest
);
// OR: more simply if saving or copying to another stream
$att->saveContent('my-file.ext');               // writes to my-file.ext
$att->saveContent($stream);                     // copies to the stream

// close only when $message is no longer being used.
fclose($handle);

```

## Documentation

* [Usage Guide](https://mail-mime-parser.org/)
* [API Reference](https://mail-mime-parser.org/api/2.0)

## Upgrading to 1.x or 2.x

* [Upgrade Guide](https://mail-mime-parser.org/upgrade-1.0)
* [Upgrade Guide](https://mail-mime-parser.org/upgrade-2.0)

## License

BSD licensed - please see [license agreement](https://github.com/zbateson/mail-mime-parser/blob/master/LICENSE).
