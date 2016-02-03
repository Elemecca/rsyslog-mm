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

Literals
--------

Number Literals
^^^^^^^^^^^^^^^


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
quote character. In order to represent characters that aren't otherwise
permissible in the configuration file, escape sequences starting with a
backslash (``\``) are supported. 

.. productionlist::
   str_escape_octal : "\" 3 * `digit_oct`
   str_escape_hex   : "\x", 2 * `digit_hex`
   str_escape_char  : "\" ( '"' | "'" | "\" | "$" | "b" | "n" | "t" | "r" )
   str_escape       : str_escape_char | str_escape_hex | str_escape_octal
   string_single    : "'" ( `character` - ( "'" | "\"       ) | str_escape )* "'"
   string_double    : '"' ( `character` - ( '"' | "\" | "$" ) | str_escape )* '"'
   string           : string_single | string_double

