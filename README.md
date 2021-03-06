About Plurr
===========

Handling plurals is not a particulalrly easy task for programmers, especially if
they are not familiar with anything except their mother tongue. Existing
approaches are either platform-specific (like gettext/po), or have limitations
(for example you can't have multiple plurals in one string), or not particularly
translator-friendly, like Java's MessageFormat/ChoiceFormat.

**Plurr is a universal format specification** designed to be easy to support for
developers and understandale for translators yet robust to support different
language requirements.

[Try live demo &rarr;](http://a-i-studio.com/plurr-demo/)
===============

Currently Plurr is implemented in:

  * JavaScript
  * Perl
  * PHP

Also there's a Plurr [syntax highlighter](https://github.com/iafan/Plurr/tree/master/demo/js/codemirror/mode/plurr) avilable for [CodeMirror](http://codemirror.net/),
which is also a part of the live demo.

Feel free to contribute to this project and provide implementations for your
favorite languages.

### Syntax Example

    Do you want to delete {N_PLURAL:this {N} file|these {N} files} permanently?

Which, depending on the provided `N` value, will render as (for English):

  * N = 1, N_PLURAL = 0: `Do you want to delete this 1 file permanently?`
  * N = 5, N_PLURAL = 1: `Do you want to delete these 5 files permanently?`

The value of N_PLURAL is determined by calling a plural() function with the
value of N, and is language-dependent. By default, plurals are calculated
automatically: when `FOO_PLURAL` placeholder is found and its value is not
provided to the formatting function, Plurr will try to do
`FOO_PLURAL = plurals(FOO)` internally, taking into consideration the current
locale defined for the Plurr object. Below is a sample JavaScript code:

    var p = new Plurr();  // create a Plurr object (once). Default locale is 'en' (English)
    ...
    alert(p.format(
      "Do you want to delete {N_PLURAL:this {N} file|these {N} files} permanently?",  // message
      {'N': 5}  // parameters
    ));

The syntax itself is not something new, it is similar to the one used in the
already mentioned Java's MessageFormat, but it is minimalistic and contains no
sensitive words that translators can inadvertently change and thus break the
whole thing.

Plurr is not about handling plurals: you can use it with your otherwise
English-only application to provide better messaging in difference scenarios:
one vs many, handling genders, using variable parts of the message depending
on the context, etc.

Advantages
----------

 1. You can use Plurr with any L10N library of your choice: you can store and
    read Plurr-formatted messages the same way as you do for any other strings.
 2. Same string format across multiple languages means that strings can be
    reused more effectively. This all reduces the time and cost of translation.
 3. Named placeholders provide a great context for translators, and this means
    better translation quality.
 4. Named placeholders allow to change their order in the final string if this
    is more appropriate for a particular target language.
 5. Less programmatic string concatenation also helps understand the message as
    a whole.

Format specification
--------------------

### 1. Simple named placeholders

#### Format

    {NAME}

#### Example

    {FOO} and {BAR}

Here `{FOO}` will be substituted with the provided FOO value, and `{BAR}` —
with the value of BAR.


Recommended is the use of capital `A..Z` and `_` symbol — this makes
placeholders stand out in the text, which gives some additional clue to
translators that these sequences are something special and should not be
translated.

There are no restrictions on placeholder names, except that they must not
contain `}` or `:` symbol.

When you need to represent symbols `{` or `}` themselves in the final string,
replace them with named placeholders with corresponding values, for example:

    alert(p.format(
      "I love {<}curly{>} braces.",
      {
        '<': '{',
        '>': '}'
      }
    ));

### 2. Placeholders with alternatives

    {CHOICE:FORM0[|FORM1][|FORM2][|FORM3][|...]}

where:

  * `CHOICE` is a zero or a positive integer.
  * `FORM0`, `FORM1` and so on are the alternative versions of the string for
    each value of CHOICE (starting from 0).

If less forms are provided than the value of `CHOICE`, the last form is used. 

As `|` is used to delimit different alternatives, in order to display such
symbol in the final string, replace it with named placeholder with the
corresponding value (same as for `{` and `}` in the section above).

#### Example in English:

    {N_PLURAL:{N} file|{N} files}

will render as:

  * N = 0, N_PLURAL = 1: `0 files`
  * N = 1, N_PLURAL = 0: `1 file`
  * N = 2, N_PLURAL = 1: `2 files`
  * N = 5, N_PLURAL = 1: `5 files`
 
#### Example of the same string translated into Russian:

    {N_PLURAL:{N} файл|{N} файла|{N} файлов}

will render as:

  * N = 0, N_PLURAL = 2: `0 файлов`
  * N = 1, N_PLURAL = 0: `1 файл`
  * N = 2, N_PLURAL = 1: `2 файла`
  * N = 5, N_PLURAL = 2: `5 файлов`

### 3. Multiple placeholders in the same string

Inside a string, there can be multiple placeholders of any kind.

#### Example:

    {X_PLURAL:{X} file|{X} files} found in {Y_PLURAL:{Y} folder|{Y} folders}.
    Do you want to {COMMAND:copy|move|delete} {X:them|it|them}?

Here, in addition to handling plurals, we use a value of `COMMAND` placeholder
to display different verbs.

#### Same example in Russian:

    В {Y_PLURAL:{Y} папке|{Y} папках|{Y} папках} {X_PLURAL:найден {X} файл|найдены {X} файла|найдено {X} файлов}.
    Хотите {X:его|их} {COMMAND:скопировать|переместить|удалить}?

### 4. Handling genders

Handling genders is the same as handling any other type of placeholders. You
just need to provide a parameter (let's name it `GENDER`), which evaluates to,
say, `0` for male, `1` for feemale, and `2` in case the gender is unknown (the
way you number genders is purely arbitary, but we recommend sticking to some
scheme that you use consistently across your application). Then all you need to
do is to construct a message like this:

    Do you want to leave {GENDER:him|her|them} a message?
    {GENDER:He|She|They} will see it when {GENDER:logs|logs|log} in.

### 5. Nested placeholders

Sometimes one would like to say "One file" instead of "1 file", and "No files"
instead of "0 files". That's (and other scenarios) are possible with nested
placeholders:

    {X_PLURAL:{X:|One|{X}} file|{X:No|{X}} files} found.

Here the `{X} file` part of the outer macro is replaced so that
when X is equal to 1, the word "one" is displayed instead.
In other cases, the number (X) is displayed.

#### Same example in Russian:

    {X_PLURAL:Найден {X:|один|{X}} файл|найдены {X} файла|{X:Не найдено|найдено {X}} файлов}.

Another common scenario is to display a completely different message for a
certain condition (the number of files is zero):
   
    {X:Sorry, but no files were found. Please try again.|{X_PLURAL:{X} file|{X} files} found.}

Such an approach allows translators to provide most natural translation
possible, and gives some peace of mind to developers helping them reduce the
amount of supporting if/else code in their applications.
