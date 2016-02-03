.. _syntax:

******
Syntax
******

Whitespace and Comments
-----------------------

RainerScript is whitespace-insensitive except for a few places in legacy
directives, which are identified when they're discussed. Whitespace is any
number of tab, space, or newline characters, and is allowed anywhere between
tokens. It's required only where two adjacent tokens would otherwise be
ambiguous.

Both C-style block comments (``/* comment */``) and shell-style line comments
(``# comment``) are allowed in most places. Unlike many languages, comments are
not fully interchangeable with whitespace. They're allowed in any context where
a statement would be allowed, and also within some compound statements. Look for
the ``cws`` production in the syntax descriptions for exact information on where
comments are permitted.

.. productionlist::
   character   : ? any ASCII character between 0x20 and 0x7E inclusive ?
   char_htab   : ? ASCII 0x09 Horizontal Tab ?
   char_newline: ? ASCII 0x0A Line Feed ?
   char_space  : ? ASCII 0x20 Space ?
   whitespace: ( char_htab | char_newline | char_space )*
   comment_block: "/*" ( character - "*/" )* "*/"
   comment_line: "#" ( character - char_newline )* char_newline
   comment: comment_block | comment_line
   cws: ( whitespace | comment )*

.. _literals:

Literals
--------

A :dfn:`literal` is a representation of a fixed value included directly in
the configuration file. It will be converted to an in-memory representation and
used directly in the operation where it appears.

Number Literals
^^^^^^^^^^^^^^^

Literal numbers can be specified in decimal (base 10), octal (base 8), or
hexadecimal (base 16). Decimal is the default; start the literal with ``0``
for octal or ``0x`` for hexadecimal. Only positive integers and zero are
supported. Negative numbers and fractional parts cannot be used.

.. tip::

   Since a leading zero is used to indicate octal literals, zero-padding
   decimal literals will cause issues. If you want to right-align number
   literals you should use space-padding instead.

.. warning::

   As can be seen in the formal syntax definition below, there's a parser bug in
   the handling of hexadecimal numbers: only one digit is supported, and the
   values ``8`` and ``9`` are missing. It is therefore recommended that decimal
   numbers be used instead.

.. productionlist::
   digit_oct  : "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7"
   digit_dec  : "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7" | "8" | "9"
   digit_hex  : "0" | "1" | "2" | "3" | "4" | "5" | "6" | "7"
              : "a" | "b" | "c" | "d" | "e" | "f"
   number_oct : "0" digit_oct+
   number_hex : "0x" digit_hex
   number_dec : "0" | ( (digit_dec - "0") digit_dec* )
   number     : number_dec | number_hex | number_octal


String Literals
^^^^^^^^^^^^^^^

RainerScript supports string literals wrapped in either single (``'``) or double
(``"``) quote characters. Both styles of string are identical except for their
quote character.

.. warning::

   For reasons that aren't clear dollar signs (``$``) must be escaped in
   double-quoted strings in expressions. This does not apply to single-quoted
   strings or strings used in objects.

.. productionlist::
   str_escape_octal : "\" `digit_oct`{3}
   str_escape_hex   : "\x" `digit_hex`{2}
   str_escape_char  : "\" `character`
   str_escape       : str_escape_octal | str_escape_hex | str_escape_char
   string_single    : "'" ( `character` - ( "'" | "\"       ) | str_escape )* "'"
   string_double    : '"' ( `character` - ( '"' | "\" | "$" ) | str_escape )* '"'
   string           : string_single | string_double


Escape sequences are supported in order to represent characters that aren't
otherwise permissible. A backslash (``\``) followed by three octal digits will
be interpreted as an octal number and replaced by the ASCII character encoded by
that number.  Likewise, ``\x`` followed by two hexadecimal digits will be
interpreted as a hexadecimal number and replaced by the corresponding ASCII
character. A backslash followed by any other character is a character escape.
The list of supported character escapes can be found in the following table. Any
character escape not currently supported is reserved for future use and has
undefined behavior.

.. warning::

   Parser support for character escape sequences is variable. This does not
   appear to be intentional. Check the notes in the following table to make sure
   the escape you want to use is available in the appropriate context.

============  ========  =====
Sequence      ASCII     Value
============  ========  =====
``\\``        ``0x5C``  Backslash
``\"``        ``0x22``  Double Quote
``\'``        ``0x27``  Single Quote
``\$`` [#o]_  ``0x24``  Dollar Sign
``\?``        ``0x3F``  Question Mark
``\a`` [#e]_  ``0x07``  Terminal Bell
``\b``        ``0x08``  Backspace
``\f`` [#e]_  ``0x0C``  Form Feed
``\n``        ``0x0A``  Line Feed
``\r``        ``0x0D``  Carriage Return
``\t``        ``0x09``  Horizontal Tab
============  ========  =====

.. [#e] not supported within :ref:`expressions`
.. [#o] not supported within :ref:`objects`
