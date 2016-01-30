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

In BSD syslog, the facility field must be either a facility name or its
underlying number, but specifying facilities numerically is strongly
discouraged. Rsyslog provides several extensions on top of the BSD behavior.
If an asterisk (``*``) is used in the facility field the selector will match
any facility. If the same priority filter should apply to multiple
facilities a list of facility names separated with commas (``,``) may be
provided in place of a single facility specification. The valid facility
names are:



* ``auth``
* ``authpriv``
* ``cron``
* ``daemon``
* ``kern``
* ``lpr``
* ``mail``
* ``mark`` (internal use only)
* ``news``
* ``security`` (deprecated alias for ``auth``)
* ``syslog``
* ``user``
* ``uucp``
* ``local0`` through ``local7``

Like facilities, in BSD syslog the priority field must be either a priority
name or number, and using numbers is discouraged. The selector will match
messages at any priority equal to or higher than that specified. Rsyslog
extends this behavior to allow an asterisk (``*``) to be used, which means
the same thing as ``debug`` (messages at any priority are selected) but more
clearly expresses that meaning. A priority name may be preceded by an equals
sign (``=``), in which case the selector will match only messages with
exactly that priority. If the priority is preceded by an exclamation point
(``!``) the meaning of the priority filter will be inverted, so any
priorities it would normally match will be excluded instead. If both
modifiers are used, the exclamation point must be before the equals sign
(``!=``). The keyword ``none`` means the same thing as ``!*`` (messages at
all priorities are excluded) but more clearly expresses that meaning. The
valid priority names, in ascending order, are:

* ``debug``
* ``info``
* ``notice``
* ``warning``
* ``warn`` (deprecated alias for ``warning``)
* ``err``
* ``error`` (deprecated alias for ``err``)
* ``crit``
* ``alert``
* ``emerg``
* ``panic`` (deprecated alias for ``emerg``)

Compound selectors can be created by joining together selectors with
semicolons (``;``). The selectors are applied from left to right and the
last result for a given message applies. If any selector matched, the message
will be considered to have matched the compound selector, unless the last
selector to match was an exclusion (priority ``none`` or starting with ``!``).
