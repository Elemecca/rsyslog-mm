.. _conditionals:

************
Conditionals
************

Rsyslog supports three kinds of conditional logic: the ``if`` statement,
classic BSD facility/priority selectors, and property filters. All three are
statements that control the execution of a block, so they can be used at any
point in the configuration --- including within another conditional --- and are
interchangeable. For example::

    if $fromhost == 'host1' then {
        mail.* action(type="omfile" file="/var/log/host1/mail.log")
        *.err /var/log/host1/errlog
    } else {
        mail.* action(type="omfile" file="/var/log/mail.log")
        *.err /var/log/errlog
    }

.. productionlist::
   stmt_conditional: `stmt_if` | `stmt_selector` | `stmt_propfilter`

Conditional Expressions
=======================

Rsyslog supports a fairly standard system of conditional expressions which are
documented in the section on :ref:`expressions`. They can be used with the
``if`` statement for conditional execution.

.. productionlist::
   stmt_if: "if" `expression` "then" `block` ( "else" `block` )?

.. tip::
   Conditional expressions are powerful, but evaluating them can be costly.
   Since the BSD-style selectors discussed below operate using bitmasks instead
   of string comparison they're `much` faster and should be used when possible,
   i.e. when operating only on the facility or priority.

A conditional statement takes either of these forms::

    if <expression> then <block>
    if <expression> then <block> else <block>

Where ``<expression>`` is a conditional expression as documented in the
:ref:`expressions` section and ``<block>`` is either a single statement or a
collection of statements wrapped in curly braces (``{}``). For example::

    if $programname == 'prog1' then {
        action(type="omfile" file="/var/log/prog1.log")
        if $msg contains 'test' then
            action(type="omfile" file="/var/log/prog1test.log")
        else
            action(type="omfile" file="/var/log/prog1notest.log")
    }


BSD Selectors
=============

Rsyslog, of course, supports traditional BSD-style selectors, which
filter on the facility and priority (together, the PRI field). It's
worth noting these are not second-class citizens in the world of
filters; they are, in fact, the most efficient way to filter on facility
or priority or both. A selector consists of two fields separated by a
period (``.``), facility on the left and priority on the right.

.. productionlist::
   facility: "auth" | "authpriv" | "cron" | "daemon" | "kern"
           : "lpr" | "mail" | "mark" | "news" | "security"
           : "syslog" | "user" | "uucp" | "ftp" | "audit"
           : "local" `digit_octal`
           : `digit`? `digit`
   priority: "alert" | "crit" | "debug" | "emerg" | "err" | "error"
           : "info" | "none" | "notice" | "panic" | "warn"
           : "warning" | "*"
   selector: facility ( "," facility )* "." "!"? "="? priority
           : selector ";" selector
   stmt_selector: selector `block`


Facility
--------

In BSD syslog, the facility field must be either a facility name or its
underlying number. Unfortunately different \*NIX implementations have
never agreed on exactly how the numbers are assigned, so their use is
discouraged. If you must use the numeric values, the correct ones for your
system can be found in ``/usr/include/syslog.h``.

Rsyslog provides a few helpful extensions to the BSD behavior.  If an
asterisk (``*``) is used in the facility field the selector will match any
facility. Also, if the same priority filter should apply to multiple
facilities (but not all of them), a list of facility names separated with
commas (``,``) may be provided in place of a single facility specification.

.. note::

   When handling the special ``*`` facility, the parser only checks that the
   first character of the given facility is ``*``. Any characters after that
   are ignored. That means these selectors are equivalent and all perfectly
   valid::

     *.emerg
     *foo.emerg
     ****.emerg

   After handling a facility value the parser skips any number of commas
   that are present. That means these selectors are equivalent and all
   perfectly valid::

     auth,authpriv.emerg
     auth,,,,authpriv.emerg
     auth,authpriv,.emerg

The valid facility names are:

=============  =======
Keyword        Purpose
=============  =======
``kern``       messages
``user``       user-level messages
``mail``       mail system
``daemon``     system daemons
``auth``       authorization messages
``security``   deprecated alias for ``auth``
``syslog``     messages generated internally by syslogd
``lpr``        printing subsystem
``news``       network news subsystem
``uucp``       UUCP subsystem
``cron``       scheduled task subsystem
``authpriv``   authorization messages (private)
``ftp``        FTP daemon
``audit``      ???
``local0``     local use
``local1``     local use
``local2``     local use
``local3``     local use
``local4``     local use
``local5``     local use
``local6``     local use
``local7``     local use
=============  =======


Priority
--------

Like the facility, in BSD syslog the priority field must be either a
priority name or number. While the priority numbers are consistent across
platforms it's still better to use the names. The selector will match
messages at any priority equal to or higher than that specified.

Rsyslog substantially extends this behavior. An asterisk (``*``) may be used
in place of a priority value, which means the same thing as ``debug``
(messages at any priority are selected) but more clearly expresses that
meaning. A priority value may be preceded by an equals sign (``=``), in which
case the selector will match only messages with exactly that priority. If
the priority is preceded by an exclamation point (``!``) the meaning of the
priority filter will be inverted, so any priorities it would normally match
will be excluded instead. If both modifiers are used, the exclamation point
must be before the equals sign (``!=``). The keyword ``none`` means the same
thing as ``!*`` (messages at all priorities are excluded) but more clearly
expresses that meaning.

The valid priority names, in descending order, are:

===========  ======  ========
Keyword      Number  Severity
===========  ======  ========
``emerg``    ``0``   Emergency: system is unusable
``panic``            deprecated alias for ``emerg``
``alert``    ``1``   Alert: action must be taken immediately
``crit``     ``2``   Critical: critical conditions
``err``      ``3``   Error: error conditions
``error``            deprecated alias for ``err``
``warning``  ``4``   Warning: warning conditions
``warn``             deprecated alias for ``warning``
``notice``   ``5``   Notice: normal but significant condition
``info``     ``6``   Informational: informational messages
``debug``    ``7``   Debug: debug-level messages
===========  ======  ========


Compound Selectors
------------------

Compound selectors can be created by joining together selectors with
semicolons (``;``). The sub-selectors are applied from left to right and
only the last action applied for each combination of facility and priority
takes effect. For any given message, if any sub-selector matched the message
will be considered to have matched the compound selector, unless the last
selector to match was an exclusion (priority ``none`` or starting with
``!``, but not ``!none``).

.. note::

   After encountering the semicolon which ends a sub-selector the parser
   will skip any number of commas or semicolons that are present. That means
   all these compound selectors are equivalent and perfectly valid::

     auth.emerg;authpriv.emerg
     auth.emerg;;authpriv.emerg
     auth.emerg;,,authpriv.emerg
     auth.emerg;,,,;,,,;authpriv.emerg
     auth.emerg;authpriv.emerg;




Property Filters
================

Rsyslog adds another type of simple filter which can match on any message
property, not just the facility and priority. They compare a provided static
value with the value of a selected message property using any of several
comparison operations.

.. productionlist::
   propfilter_op: "isempty" | "isequal" | "contains" | "startswith"
                : "regex" | "ereregex"
   propfilter_string: '"' ( `character` - ( '"' | '\' ) | '\' ( '"' | '\' ) )* '"'
   propfilter: ":" `property` "," `space`* "!"? propfilter_op "," `space`* propfilter_string
   stmt_propfilter: propfilter `block`

.. warning::
   Property filters were added to Rsyslog before support for full conditional
   expressions was introduced. While they're not quite deprecated they're less
   flexible and no more efficient than conditional expressions, which should
   therefore generally be used instead when writing new configurations.

A property filter consists of a colon followed by a property name, then a comma,
optional space, a comparison operation, another comma and space, and finally a
quoted string. Property names are case-sensitive, so ``msg`` works while ``MSG``
will cause an error. A full list of built-in properties can be found in the
section on properties.

The supported comparison operations are listed below. In addition, an
exclamation point (``!``) can be added to the beginning of any operation name to
negate its meaning.

+----------------+-------------------------------------------------------------+
| Keyword        | Operation                                                   |
+================+=============================================================+
| ``isempty``    | Checks if the property is empty, which means either it      |
|                | hasn't been set or has been set to the empty string.        |
+----------------+-------------------------------------------------------------+
| ``isequal``    | Checks whether the given value exactly matches the          |
|                | property's value.                                           |
+----------------+-------------------------------------------------------------+
| ``contains``   | Checks whether the given value exactly matches a substring  |
|                | of the property's value at any location.                    |
+----------------+-------------------------------------------------------------+
| ``startswith`` | Checks whether the given value exactly matches a substring  |
|                | of the property's value starting at the first character.    |
+----------------+-------------------------------------------------------------+
| ``regex``      | Interprets the given value as a POSIX Basic Regular         |
|                | Expression and checks whether it matches the property.      |
+----------------+-------------------------------------------------------------+
| ``ereregex``   | Interprets the given value as a POSIX Extended Regular      |
|                | Expression and checks whether it matches the property.      |
+----------------+-------------------------------------------------------------+

The value is a quoted string, but it follows different rules than strings in
most other parts of the configuration file. It supports only very limited
escapes: ``\\`` will produce a backslash (``\``) and ``\"`` will produce a
double quote (``"``). All other escape sequences (backslash followed by any
character) are reserved for future use and behave in an undefined manner. Any
backslash not intended as part of an escape sequence must therefore be escaped.
