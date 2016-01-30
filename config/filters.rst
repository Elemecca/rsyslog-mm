*****************
Filter Conditions
*****************

BSD Selectors
=============

Rsyslog, of course, supports traditional BSD-style selectors, which
filter on the facility and priority (together, the PRI field). It's
worth noting these are not second-class citizens in the world of
filters; they are, in fact, the most efficient way to filter on facility
or priority or both. A selector consists of two fields separated by a
period (``.``), facility on the left and priority on the right.


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
semicolons (``;``). The selectors are applied from left to right and the
last result for a given message applies. If any selector matched, the message
will be considered to have matched the compound selector, unless the last
selector to match was an exclusion (priority ``none`` or starting with ``!``).
