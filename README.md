# BBCode Extension for PHP

A high-performance PHP extension for parsing BBCode markup. Features tree-based parsing, callback support, smiley handling, and tag nesting validation.

## Requirements

- PHP 8.0 or later
- C compiler (gcc, clang)

## Installation

### From Source

```bash
# Clone the repository
git clone https://github.com/volkish/php_pecl_bbcode.git
cd php_pecl_bbcode

# Build the extension
phpize
./configure --enable-bbcode
make

# Run tests
make test

# Install (may require sudo)
make install
```

### Enable the Extension

Add to your `php.ini`:

```ini
extension=bbcode.so
```

Or load dynamically in your script:

```php
if (!extension_loaded('bbcode')) {
    dl('bbcode.so');
}
```

## Quick Start

```php
<?php
// Define BBCode rules
$bbcodes = [
    'b' => [
        'type' => BBCODE_TYPE_NOARG,
        'open_tag' => '<strong>',
        'close_tag' => '</strong>',
    ],
    'i' => [
        'type' => BBCODE_TYPE_NOARG,
        'open_tag' => '<em>',
        'close_tag' => '</em>',
    ],
    'url' => [
        'type' => BBCODE_TYPE_OPTARG,
        'open_tag' => '<a href="{PARAM}">',
        'close_tag' => '</a>',
        'default_arg' => '{CONTENT}',
    ],
];

// Create parser and parse text
$parser = bbcode_create($bbcodes);
$text = '[b]Hello[/b] [i]World[/i]! Visit [url]https://php.net[/url]';
echo bbcode_parse($parser, $text);

// Output: <strong>Hello</strong> <em>World</em>! Visit <a href="https://php.net">https://php.net</a>
```

## API Reference

### Functions

#### `bbcode_create([array $initial_tags]): resource`

Creates a new BBCode parser resource.

```php
$parser = bbcode_create($rules);
```

#### `bbcode_add_element(resource $parser, string $tag_name, array $tag_definition): bool`

Adds a new tag rule to an existing parser.

```php
bbcode_add_element($parser, 'code', [
    'type' => BBCODE_TYPE_NOARG,
    'open_tag' => '<pre><code>',
    'close_tag' => '</code></pre>',
]);
```

#### `bbcode_parse(resource $parser, string $text): string`

Parses BBCode text and returns the processed result.

```php
$html = bbcode_parse($parser, '[b]Bold text[/b]');
```

#### `bbcode_add_smiley(resource $parser, string $search, string $replace): bool`

Adds a smiley replacement rule.

```php
bbcode_add_smiley($parser, ':)', '<img src="smile.gif" alt=":)">');
bbcode_add_smiley($parser, ':(', '<img src="sad.gif" alt=":(">');
```

#### `bbcode_set_flags(resource $parser, int $flags, int $mode = BBCODE_SET_FLAGS_SET): bool`

Sets parser flags.

```php
// Set flags (replaces existing)
bbcode_set_flags($parser, BBCODE_AUTO_CORRECT | BBCODE_DEFAULT_SMILEYS_ON);

// Add flags
bbcode_set_flags($parser, BBCODE_CORRECT_REOPEN_TAGS, BBCODE_SET_FLAGS_ADD);

// Remove flags
bbcode_set_flags($parser, BBCODE_DEFAULT_SMILEYS_ON, BBCODE_SET_FLAGS_REMOVE);
```

#### `bbcode_set_arg_parser(resource $parser, resource $arg_parser): bool`

Sets a sub-parser for processing tag arguments.

```php
$arg_parser = bbcode_create($arg_rules);
bbcode_set_arg_parser($parser, $arg_parser);
```

#### `bbcode_destroy(resource $parser): bool`

Destroys a parser resource and frees memory.

```php
bbcode_destroy($parser);
```

### Tag Definition Options

| Key | Type | Description |
|-----|------|-------------|
| `type` | int | **Required.** Tag type constant (see below) |
| `open_tag` | string | HTML opening tag. Use `{CONTENT}` and `{PARAM}` placeholders |
| `close_tag` | string | HTML closing tag |
| `default_arg` | string | Default argument for `BBCODE_TYPE_OPTARG` when no argument given |
| `flags` | int | Tag-specific flags |
| `childs` | string | Comma-separated list of allowed child tags (`all`, `!tag1,tag2` for exclusion, or empty for none) |
| `parents` | string | Comma-separated list of allowed parent tags |
| `content_handling` | callable | Callback function for content transformation |
| `param_handling` | callable | Callback function for parameter transformation |
| `max` | int | Maximum number of times this tag can be parsed (-1 for unlimited) |

### Tag Type Constants

| Constant | Description |
|----------|-------------|
| `BBCODE_TYPE_NOARG` | Tag without argument: `[tag]content[/tag]` |
| `BBCODE_TYPE_SINGLE` | Self-closing tag: `[tag]` |
| `BBCODE_TYPE_ARG` | Tag requiring argument: `[tag=value]content[/tag]` |
| `BBCODE_TYPE_OPTARG` | Tag with optional argument: `[tag]` or `[tag=value]` |
| `BBCODE_TYPE_ROOT` | Root element (use with empty string key `''`) |

### Tag Flags

| Constant | Description |
|----------|-------------|
| `BBCODE_FLAGS_ARG_PARSING` | Parse BBCode within tag arguments |
| `BBCODE_FLAGS_CDATA_NOT_ALLOWED` | Disallow text content (only child tags) |
| `BBCODE_FLAGS_SMILEYS_ON` | Enable smileys for this tag |
| `BBCODE_FLAGS_SMILEYS_OFF` | Disable smileys for this tag |
| `BBCODE_FLAGS_ONE_OPEN_PER_LEVEL` | Only one instance per nesting level |
| `BBCODE_FLAGS_REMOVE_IF_EMPTY` | Remove tag if content is empty |
| `BBCODE_FLAGS_DENY_REOPEN_CHILD` | Prevent reopening child tags after closing |

### Parser Flags

| Constant | Description |
|----------|-------------|
| `BBCODE_ARG_DOUBLE_QUOTE` | Allow double-quoted arguments: `[tag="value"]` |
| `BBCODE_ARG_SINGLE_QUOTE` | Allow single-quoted arguments: `[tag='value']` |
| `BBCODE_ARG_HTML_QUOTE` | Allow HTML-style quotes |
| `BBCODE_ARG_QUOTE_ESCAPING` | Allow escaping quotes within arguments |
| `BBCODE_AUTO_CORRECT` | Auto-correct malformed BBCode |
| `BBCODE_CORRECT_REOPEN_TAGS` | Reopen tags closed by auto-correction |
| `BBCODE_DISABLE_TREE_BUILD` | Disable tree building (faster but less accurate) |
| `BBCODE_DEFAULT_SMILEYS_ON` | Enable smileys by default |
| `BBCODE_DEFAULT_SMILEYS_OFF` | Disable smileys by default |
| `BBCODE_FORCE_SMILEYS_OFF` | Force smileys off globally |
| `BBCODE_SMILEYS_CASE_INSENSITIVE` | Case-insensitive smiley matching |

### Flag Mode Constants

| Constant | Description |
|----------|-------------|
| `BBCODE_SET_FLAGS_SET` | Replace all flags |
| `BBCODE_SET_FLAGS_ADD` | Add to existing flags |
| `BBCODE_SET_FLAGS_REMOVE` | Remove from existing flags |

## Examples

### Basic Formatting Tags

```php
<?php
$bbcodes = [
    'b' => ['type' => BBCODE_TYPE_NOARG, 'open_tag' => '<b>', 'close_tag' => '</b>'],
    'i' => ['type' => BBCODE_TYPE_NOARG, 'open_tag' => '<i>', 'close_tag' => '</i>'],
    'u' => ['type' => BBCODE_TYPE_NOARG, 'open_tag' => '<u>', 'close_tag' => '</u>'],
    's' => ['type' => BBCODE_TYPE_NOARG, 'open_tag' => '<s>', 'close_tag' => '</s>'],
];

$parser = bbcode_create($bbcodes);
echo bbcode_parse($parser, '[b]Bold[/b] [i]Italic[/i] [u]Underline[/u]');
// Output: <b>Bold</b> <i>Italic</i> <u>Underline</u>
```

### URL and Image Tags

```php
<?php
$bbcodes = [
    'url' => [
        'type' => BBCODE_TYPE_OPTARG,
        'open_tag' => '<a href="{PARAM}">',
        'close_tag' => '</a>',
        'default_arg' => '{CONTENT}',
    ],
    'img' => [
        'type' => BBCODE_TYPE_NOARG,
        'open_tag' => '<img src="',
        'close_tag' => '" />',
        'childs' => '',  // No child tags allowed
    ],
];

$parser = bbcode_create($bbcodes);

// URL without argument - uses content as href
echo bbcode_parse($parser, '[url]https://php.net[/url]');
// Output: <a href="https://php.net">https://php.net</a>

// URL with argument
echo bbcode_parse($parser, '[url=https://php.net]PHP Website[/url]');
// Output: <a href="https://php.net">PHP Website</a>

// Image
echo bbcode_parse($parser, '[img]https://example.com/image.png[/img]');
// Output: <img src="https://example.com/image.png" />
```

### Nested Tags with Child Restrictions

```php
<?php
$bbcodes = [
    '' => [
        'type' => BBCODE_TYPE_ROOT,
        'childs' => '!code',  // 'code' not allowed at root level
    ],
    'quote' => [
        'type' => BBCODE_TYPE_OPTARG,
        'open_tag' => '<blockquote><cite>{PARAM}</cite>',
        'close_tag' => '</blockquote>',
        'default_arg' => 'Anonymous',
        'childs' => 'b,i,url',  // Only these tags allowed inside
    ],
    'code' => [
        'type' => BBCODE_TYPE_NOARG,
        'open_tag' => '<pre><code>',
        'close_tag' => '</code></pre>',
        'childs' => '',  // No BBCode inside code blocks
    ],
    'b' => ['type' => BBCODE_TYPE_NOARG, 'open_tag' => '<b>', 'close_tag' => '</b>'],
    'i' => ['type' => BBCODE_TYPE_NOARG, 'open_tag' => '<i>', 'close_tag' => '</i>'],
    'url' => [
        'type' => BBCODE_TYPE_OPTARG,
        'open_tag' => '<a href="{PARAM}">',
        'close_tag' => '</a>',
        'default_arg' => '{CONTENT}',
    ],
];

$parser = bbcode_create($bbcodes);
$text = '[quote=John]This is [b]bold[/b] text[/quote]';
echo bbcode_parse($parser, $text);
// Output: <blockquote><cite>John</cite>This is <b>bold</b> text</blockquote>
```

### Content Callbacks

```php
<?php
// Callback to escape HTML in content
function escape_content($content, $param) {
    return htmlspecialchars($content, ENT_QUOTES, 'UTF-8');
}

// Callback to validate URLs
function validate_url($content, $param) {
    if (filter_var($param, FILTER_VALIDATE_URL)) {
        return $param;
    }
    return '#invalid-url';
}

$bbcodes = [
    'code' => [
        'type' => BBCODE_TYPE_NOARG,
        'open_tag' => '<pre><code>',
        'close_tag' => '</code></pre>',
        'content_handling' => 'escape_content',
    ],
    'url' => [
        'type' => BBCODE_TYPE_OPTARG,
        'open_tag' => '<a href="{PARAM}">',
        'close_tag' => '</a>',
        'default_arg' => '{CONTENT}',
        'param_handling' => 'validate_url',
    ],
];

$parser = bbcode_create($bbcodes);
echo bbcode_parse($parser, '[code]<script>alert("XSS")</script>[/code]');
// Output: <pre><code>&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;</code></pre>
```

### Class Method Callbacks

```php
<?php
class BBCodeHandler {
    public static function staticHandler($content, $param) {
        return strtoupper($content);
    }

    public function instanceHandler($content, $param) {
        return str_replace(' ', '&nbsp;', $content);
    }
}

$handler = new BBCodeHandler();

$bbcodes = [
    'upper' => [
        'type' => BBCODE_TYPE_NOARG,
        'open_tag' => '',
        'close_tag' => '',
        'content_handling' => ['BBCodeHandler', 'staticHandler'],
    ],
    'nbsp' => [
        'type' => BBCODE_TYPE_NOARG,
        'open_tag' => '',
        'close_tag' => '',
        'content_handling' => [$handler, 'instanceHandler'],
    ],
];

$parser = bbcode_create($bbcodes);
echo bbcode_parse($parser, '[upper]hello world[/upper]');  // HELLO WORLD
echo bbcode_parse($parser, '[nbsp]hello world[/nbsp]');    // hello&nbsp;world
```

### Smileys

```php
<?php
$bbcodes = [
    'b' => ['type' => BBCODE_TYPE_NOARG, 'open_tag' => '<b>', 'close_tag' => '</b>'],
    'code' => [
        'type' => BBCODE_TYPE_NOARG,
        'open_tag' => '<code>',
        'close_tag' => '</code>',
        'flags' => BBCODE_FLAGS_SMILEYS_OFF,  // Disable smileys in code
    ],
];

$parser = bbcode_create($bbcodes);

// Add smileys
bbcode_add_smiley($parser, ':)', '<img src="smile.gif" alt=":)">');
bbcode_add_smiley($parser, ':(', '<img src="sad.gif" alt=":(">');
bbcode_add_smiley($parser, ':D', '<img src="grin.gif" alt=":D">');

$text = 'Hello :) [b]Bold :D[/b] [code]:)[/code]';
echo bbcode_parse($parser, $text);
// Output: Hello <img src="smile.gif" alt=":)"> <b>Bold <img src="grin.gif" alt=":D"></b> <code>:)</code>
```

### Tag Count Limits

```php
<?php
$bbcodes = [
    'b' => [
        'type' => BBCODE_TYPE_NOARG,
        'open_tag' => '<b>',
        'close_tag' => '</b>',
        'max' => 2,  // Only parse first 2 occurrences
    ],
];

$parser = bbcode_create($bbcodes);
$text = '[b]One[/b] [b]Two[/b] [b]Three[/b]';
echo bbcode_parse($parser, $text);
// Output: <b>One</b> <b>Two</b> [b]Three[/b]
```

## Version

- Extension Version: 2.0.1
- Library Version: 2.0

## License

This extension is released under the PHP License 3.01.

## Author

Xavier De Cock <void@php.net>
