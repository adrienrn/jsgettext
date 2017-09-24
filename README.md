Javascript Gettext - Javascript implemenation of GNU Gettext API.

## Install

You can install this using Bower since 0.8.1.

```
$ bower install jsgettext --save
```

## Getting started

```html
<script language="javascript" src="/path/LC_MESSAGES/myDomain.json"></script>
<script language="javascript" src="/path/Gettext.js"></script>
```

```javascript
assuming myDomain.json defines variable json_locale_data
var params = {
    "domain" : "myDomain",
    "locale_data" : json_locale_data
};

var gt = new Gettext(params);
```

Create a shortcut if you'd like:

```javascript
function _(msgid) { return gt.gettext(msgid); }

alert(_("some string"));
```

Or use fully named method:

```javascript
alert(gt.gettext("some string"));
```

Change to use a different "domain":

```javascript
gt.textdomain("anotherDomain");
alert(gt.gettext("some string"));
```

The other way to load the language lookup is a "link" tag. Downside is that not all browsers cache XMLHttpRequests the same way, so caching of the language data isn't guarenteed across page loads.

Upside is that it's easy to specify multiple files.

```html
<link rel="gettext" href="/path/LC_MESSAGES/myDomain.json" />
<script language="javascript" src="/path/Gettext.js"></script>
```

Rest is the same:

```javascript
var gt = new Gettext({ "domain" : "myDomain" });
```

The reson the shortcuts aren't exported by default is because they'd be glued to the single domain you created. So, if you're adding i18n support to some js library, you should use it as so:

```javascript
if (typeof(MyNamespace) == 'undefined') MyNamespace = {};
MyNamespace.MyClass = function () {
    var gtParms = { "domain" : 'MyNamespace_MyClass' };
    this.gt = new Gettext(gtParams);
    return this;
};
MyNamespace.MyClass.prototype._ = function (msgid) {
    return this.gt.gettext(msgid);
};
MyNamespace.MyClass.prototype.something = function () {
    var myString = this._("this will get translated");
};
```

Adding the shortcuts to a global scope is easier. If that is ok in your app, this is certainly easier.

```javascript
var myGettext = new Gettext({ 'domain' : 'myDomain' });
function _ (msgid) {
    return myGettext.gettext(msgid);
}
alert(_("text"));
```

Note: if you are loading via the script tag, you can only load one file, but it can contain multiple domains.

```javascript
var json_locale_data = {
    "MyDomain" : {
        "" : {
            "header_key" : "header value",
            "header_key" : "header value",
        "msgid" : [ "msgid_plural", "msgstr", "msgstr_plural", "msgstr_pluralN" ],
        "msgctxt\004msgid" : [ null, "msgstr" ],
        },
    "AnotherDomain" : {
        },
    }
```

## Description

This is a javascript implementation of GNU Gettext, providing internationalization support for javascript.

It differs from existing javascript implementations in that it will support all current Gettext features (ex. plural and context support), and will also support loading language catalogs from .mo, .po, or preprocessed json files (converter included).

The locale initialization differs from that of GNU Gettext / POSIX. Rather than setting the category, domain, and paths, and letting the libs find the right file, you must explicitly load the file at some point. The "domain" will still be honored. Future versions may be expanded to include support for set\_locale like features.

## Configuration

Configure in one of two ways:

1. Optimal. Load language definition from statically defined json data.

        <script language="javascript" src="/path/locale/domain.json"></script>

        in domain.json
        json_locale_data = {
            "mydomain" : {
                po header fields
                "" : {
                    "plural-forms" : "...",
                    "lang" : "en",
                    },
                all the msgid strings and translations
                "msgid" : [ "msgid_plural", "translation", "plural_translation" ],
            },
        };
        please see the included bin/po2json script for the details on this format

    This method also allows you to use unsupported file formats, so long as you can parse them into the above format.

2. Use AJAX to load language file.

    Use XMLHttpRequest (actually, SJAX - syncronous) to load an external resource.

    Supported external formats are:

    - Javascript Object Notation (.json)

        (see bin/po2json)

            type=application/json

    - Uniforum Portable Object (.po)

        (see GNU Gettext's xgettext)

            type=application/x-po

    - Machine Object (compiled .po) (.mo)

        NOTE: .mo format isn't actually supported just yet, but support is planned.

        (see GNU Gettext's msgfmt)

            type=application/x-mo

## Methods

See [[Methods]] wiki page.

## Notes

These are some notes on the internals.

  - Locale caching

    Loaded locale data is currently cached class-wide. This means that if two scripts are both using Gettext.js, and both share the same gettext domain, that domain will only be loaded once. This will allow you to grab a new object many times from different places, utilize the same domain, and share a single translation file. The downside is that a domain won't be RE-loaded if a new object is instantiated on a domain that had already been instantiated.

## Bugs & To-dos

  - Error handling

    Currently, there are several places that throw errors. In GNU Gettext, there are no fatal errors, which allows text to still be displayed regardless of how broken the environment becomes. We should evaluate and determine where we want to stand on that issue.

  - Syncronous only support (no ajax support)

    Currently, fetching language data is done purely syncronous, which means the page will halt while those files are fetched/loaded.

    This is often what you want, as then following translation requests will actually be translated. However, if all your calls are done dynamically (ie. error handling only or something), loading in the background may be more adventagous.

    It's still recommended to use the statically defined <script ...> method, which should have the same delay, but it will cache the result.

  - Domain support

    domain support while using shortcut methods like `_('string')` or `i18n('string')`.

    Under normal apps, the domain is usually set globally to the app, and a single language file is used. Under javascript, you may have multiple libraries or applications needing translation support, but the namespace is essentially global.

    It's recommended that your app initialize it's own shortcut with it's own domain.  (See examples/wrapper/i18n.js for an example.)

    Basically, you'll want to accomplish something like this:

        in some other .js file that needs i18n
        this.i18nObj = new i18n;
        this.i18n = this.i18nObj.init('domain');
        do translation
        alert( this.i18n("string") );

    If you use this raw Gettext object, then this is all handled for you, as you have your own object then, and will be calling `myGettextObject.gettext('string')` and such.

  - Encoding

    May want to add encoding/reencoding stuff. See GNU iconv, or the perl module Locale::Recode from libintl-perl.

## Compatibility

This has been tested on the following browsers. It may work on others, but these are all those to which I have access.

```
    FF1.5, FF2, FF3, IE6, IE7, Opera9, Opera10, Safari3.1, Chrome

    *FF = Firefox
    *IE = Internet Explorer
```

## Requires

bin/po2json requires perl, and the perl modules Locale::PO and JSON.

## See also

bin/po2json (included),
examples/normal/index.html,
examples/wrapper/i18n.html, examples/wrapper/i18n.js,
Locale::gettext\_pp(3pm), POSIX(3pm), gettext(1), gettext(3)

## Author

Copyright (C) 2008, Joshua I. Miller <unrtst@cpan.org>, all rights reserved. See the source code for details.
