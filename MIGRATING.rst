.. _MIGRATING:

MIGRATING
*********

This document describes the major changes occurring between versions of
Modules. It provides an overview of the new features and changed behaviors
that will be encountered when upgrading.


Migrating from v4.0 to v4.1
===========================

This new version is backward-compatible with v4.0 and primarily fixes bugs and
adds new features.

New features
------------

Version 4.1 introduces a bunch of new functionalities. These major new
features are described in this section.

Virtual modules
^^^^^^^^^^^^^^^

A virtual module stands for a module name associated to a modulefile. The
modulefile is the script interpreted when loading or unloading the virtual
module which appears or can be found with its virtual name.

The **module-virtual** modulefile command is introduced to give the ability
to define these virtual modules. This new command takes a module name as
first argument and a modulefile location as second argument::

    module-virtual app/1.2.3 /path/to/virtualmod/app

With this feature it is now possible to dynamically define modulefiles
depending on the context.

Extend module command with site-specific Tcl code
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``module`` command can now be extended with site-specific Tcl
code. ``modulecmd.tcl`` now looks at a **siteconfig.tcl** file in an
``etcdir`` defined at configure time (by default ``$prefix/etc``). If
it finds this Tcl script file, it is sourced within ``modulecmd.tcl`` at the
beginning of the main procedure code.

``siteconfig.tcl`` enables to supersede any global variable or procedure
definitions made in ``modulecmd.tcl`` with site-specific code. A module
sub-command can for instance be redefined to make it fit local needs
without having to touch the main ``modulecmd.tcl``.

Quarantine mechanism to protect module execution
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

To protect the module command run-time environment from side effect
coming from the current environment definition a quarantine mechanism
is introduced. This mechanism, sets within module function definition
and shell initialization script, modifies the ``modulecmd.tcl`` run-time
environment to sanitize it.

The mechanism is piloted by environment variables. First of all
``MODULES_RUN_QUARANTINE``, a space-separated list of environment variable
names. Every variable found in ``MODULES_RUN_QUARANTINE`` will be set in
quarantine during the ``modulecmd.tcl`` run-time. Their value will be set
empty or set to the value of the corresponding ``MODULES_RUNENV_<VAR>``
environment variable if defined. Once ``modulecmd.tcl`` is started it
restores quarantine variables to their original values.

``MODULES_RUN_QUARANTINE`` and ``MODULES_RUNENV_<VAR>`` environment variables
can be defined at build time by using the following configure option::

    --with-quarantine-vars='VARNAME[=VALUE] ...'

Quarantine mechanism is available for all supported shells except ``csh``
and ``tcsh``.

Pager support
^^^^^^^^^^^^^

The informational messages Modules sends on the *stderr* channel may
sometimes be quite long. This is especially the case for the avail
sub-command when hundreds of modulefiles are handled. To improve the
readability of those messages, *stderr* output can now be piped into a
paging command.

This new feature can be controlled at build time with the ``--with-pager``
and ``--with-pager-opts`` configure options. Default pager command is set
to ``less`` and its relative options are by default ``-eFKRX``. Default
configuration can be supersedes at run-time with ``MODULES_PAGER`` environment
variables or command-line switches (``--no-pager``, ``--paginate``).

.. warning:: On version ``4.1.0``, the ``PAGER`` environment variable was
   taken in consideration to supersede pager configuration at run-time. Since
   version ``4.1.1``, ``PAGER`` environment variable is ignored to avoid side
   effects coming from the system general pager configuration.

Module function to return value in scripting languages
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

On Tcl, Perl, Python, Ruby, CMake and R scripting shells, module function
was not returning value and until now an occurred error led to raising a
fatal exception.

To make ``module`` function more friendly to use on these scripting shells
it now returns a value. False in case of error, true if everything goes well.

As a consequence, returned value of a module sub-command can be checked. For
instance in Python::

    if module('load', 'foo'):
      # success
    else:
      # failure

New modulefile commands
^^^^^^^^^^^^^^^^^^^^^^^

4 new modulefile Tcl commands have are introduced:

* **is-saved**: returns true or false whether a collection, corresponding to
  currently set collection target, exists or not.
* **is-used**: returns true or false whether a given directory is currently
  enabled in ``MODULEPATH``.
* **is-avail**: returns true or false whether a given modulefile exists in
  currently enabled module paths.
* **module-info loaded**: returns the exact name of the modulefile currently
  loaded corresponding to the name argument.

Multiple collections, paths or modulefiles can be passed respectively to
``is-saved``, ``is-used`` and ``is-avail`` in which case true is returned if
at least one argument matches condition (acts as a OR boolean operation). No
argument may be passed to ``is-loaded``, ``is-saved`` and ``is-used``
commands to return if anything is respectively loaded, saved or used.

If no loaded modulefile matches the ``module-info loaded`` query, an empty
string is returned.

New module sub-commands
^^^^^^^^^^^^^^^^^^^^^^^

Modulefile-specific commands are sometimes wished to be used outside of a
modulefile context. Especially for the commands managing path variables
or commands querying current environment context. So the following
modulefile-specific commands have been made reachable as module sub-commands
with same arguments and properties as if called from within a modulefile:

* **append-path**
* **prepend-path**
* **remove-path**
* **is-loaded**
* **info-loaded**

The ``is-loaded`` sub-command returns a boolean value. Small Python example::

    if module('is-loaded', 'app'):
      print 'app is loaded'
    else:
      print 'app not loaded'

``info-loaded`` returns a string value and is the sub-command counterpart
of the ``module-info loaded`` modulefile command::

    $ module load app/0.8
    $ module info-loaded app
    app/0.8

Further reading
---------------

To get a complete list of the changes between Modules v4.0 and v4.1,
please read the :ref:`NEWS` document.


Migrating from v3.2 to v4.0
===========================

Major evolution occurs with this v4.0 release as the traditional *module*
command implemented in C is replaced by the native Tcl version. This full
Tcl rewrite of the Modules package was started in 2002 and has now reached
maturity to take over the binary version. This flavor change enables to
refine and push forward the *module* concept.

This document provides an outlook of what is changing when migrating from
v3.2 to v4.0 by first describing the introduced new features. Both v3.2
and v4.0 are quite similar and transition to the new major version should
be smooth. Slights differences may be noticed in a few use-cases. So the
second part of the document will help to learn about them by listing the
features that have been discontinued in this new major release or the
features where a behavior change can be noticed.

New features
------------

On its overall this major release brings a lot more robustness to the
*module* command with now more than 4000 non-regression tests crafted
to ensure correct operations over the time. This version 4.0 also comes
with fair amount of improved functionalities. The major new features are
described in this section.

Additional shells supported
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modules v4 introduces support for **fish**, **lisp**, **tcl** and **R**
code output.

Non-zero exit code in case of error
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

All module sub-commands will now return a non-zero exit code in case of error
whereas Modules v3.2 always returned zero exit code even if issue occurred.

Output redirect
^^^^^^^^^^^^^^^

Traditionally the *module* command output text that should be seen by the
user on *stderr* since shell commands are output to *stdout* to change
shell's environment. Now on *sh*, *bash*, *ksh*, *zsh* and *fish* shells,
output text is redirected to *stdout* after shell command evaluation if
shell is in interactive mode.

Filtering avail output
^^^^^^^^^^^^^^^^^^^^^^

Results obtained from the **avail** sub-command can now be filtered to only
get the default version of each module name with use of the **--default**
or **-d** command line switch. Default version is either the explicitly
set default version or the highest numerically sorted modulefile or module
alias if no default version set.

It is also possible to filter results to only get the highest numerically
sorted version of each module name with use of the **--latest** or **-L**
command line switch.

Extended support for module alias and symbolic version
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Module aliases are now included in the result of the **avail**, **whatis**
and **apropos** sub-commands. They are displayed in the module path
section where they are defined or in a *global/user modulerc* section for
aliases set in user's or global ``modulerc`` file. A **@** symbol is added
in parenthesis next to their name to distinguish them from modulefiles.

Search may be performed with an alias or a symbolic version-name passed
as argument on **avail**, **whatis** and **apropos** sub-commands.

Modules v4 resolves module alias or symbolic version passed to **unload**
command to then remove the loaded modulefile pointed by the mentioned
alias or symbolic version.

A symbolic version sets on a module alias is now propagated toward the
resolution path to also apply to the relative modulefile if it still
correspond to the same module name.

Hiding modulefiles
^^^^^^^^^^^^^^^^^^

Visibility of modulefiles can be adapted by use of file mode bits or file
ownership. If a modulefile should only be used by a given subset of persons,
its mode an ownership can be tailored to provide read rights to this group of
people only. In this situation, module only reports the modulefile, during an
**avail** command for instance, if this modulefile can be read by the current
user.

These hidden modulefiles are simply ignored when walking through the
modulepath content. Access issues (permission denied) occur only when trying
to access directly a hidden modulefile or when accessing a symbol or an alias
targeting a hidden modulefile.

Improved modulefiles location
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

When looking for an implicit default in a modulefile directory, aliases
are now taken into account in addition to modulefiles and directories to
determine the highest numerically sorted element.

Modules v4 resolves module alias or symbolic version when it points to a
modulefile located in another modulepath.

Access issues (permission denied) are now distinguished from find issues
(cannot locate) when trying to access directly a directory or a modulefile
as done on **load**, **display** or **whatis** commands. In addition,
on this kind of access not readable ``.modulerc`` or ``.version`` files are
ignored rather producing a missing magic cookie error.

Module collection
^^^^^^^^^^^^^^^^^

Modules v4 introduces support for module *collections*. Collections
describe a sequence of **module use** then **module load** commands that
are interpreted by Modules to set the user environment as described by this
sequence. When a collection is activated, with the **restore** sub-command,
modulepaths and loaded modules are unused or unloaded if they are not part
or if they are not ordered the same way as in the collection.

Collections are generated by the **save** sub-command that dumps the current
user environment state in terms of modulepaths and loaded modules. By default
collections are saved under the ``$HOME/.module`` directory. Collections
can be listed with **savelist** sub-command, displayed with **saveshow**
and removed with **saverm**.

Collections may be valid for a given target if they are suffixed. In this
case these collections can only be restored if their suffix correspond
to the current value of the ``MODULES_COLLECTION_TARGET`` environment
variable. Saving collection registers the target footprint by suffixing
the collection filename with ``.$MODULES_COLLECTION_TARGET``.

Path variable element counter
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modules 4 provides path element counting feature which increases a
reference counter each time a given path entry is added to a given
path-like environment variable. As consequence a path entry element is
removed from a path-like variable only if the related element counter is
equal to 1. If this counter is greater than 1, path element is kept in
variable and reference counter is decreased by 1.

This feature allows shared usage of particular path elements. For instance,
modulefiles can append ``/usr/local/bin`` to ``PATH``, which is not unloaded
until all the modulefiles that loaded it unload too.

Optimized I/O operations
^^^^^^^^^^^^^^^^^^^^^^^^

Substantial work has been done to reduce the number of I/O operations
done during global modulefile analysis commands like **avail** or
**whatis**. ``stat``, ``open``, ``read`` and ``close`` I/O operations have
been cut down to the minimum required when walking through the modulepath
directories to check if files are modulefiles or to resolve module aliases.

Interpretation of modulefiles and modulerc are handled by the minimum
required Tcl interpreters. Which means a configured Tcl interpreter is
reused as much as possible between each modulefile interpretation or
between each modulerc interpretation.

Sourcing modulefiles
^^^^^^^^^^^^^^^^^^^^

Modules 4 introduces the possibility to **source** a modulefile rather
loading it. When it is sourced, a modulefile is interpreted into the shell
environment but then it is not marked loaded in shell environment which
differ from **load** sub-command.

This functionality is used in shell initialization scripts once **module**
function is defined. There the ``etc/modulerc`` modulefile is sourced to
setup the initial state of the environment, composed of *module use*
and *module load* commands.


Removed features and substantial behavior changes
-------------------------------------------------

Following sections provide list of Modules v3.2 features that are
discontinued on Modules v4 or features with a substantial behavior change
that should be taken in consideration when migrating to v4.

Package initialization
^^^^^^^^^^^^^^^^^^^^^^

``MODULESBEGINENV`` environment snapshot functionality is not supported
anymore on Modules v4. Modules collection mechanism should be used instead to
**save** and **restore** sets of enabled modulepaths and loaded modulefiles.

Command line switches
^^^^^^^^^^^^^^^^^^^^^

Some command line switches are not supported anymore on v4.0. When still
using them, a warning message is displayed and the command is ran with these
unsupported switches ignored. Following command line switches are concerned:

* ``--force``, ``-f``
* ``--human``
* ``--verbose``, ``-v``
* ``--silent``, ``-s``
* ``--create``, ``-c``
* ``--icase``, ``-i``
* ``--userlvl`` lvl, ``-u`` lvl

Module sub-commands
^^^^^^^^^^^^^^^^^^^

During an **help** sub-command, Modules v4 does not redirect output made
on stdout in *ModulesHelp* Tcl procedure to stderr. Moreover when running
**help**, version 4 interprets all the content of the modulefile, then call
the *ModulesHelp* procedure if it exists, whereas Modules 3.2 only interprets
the *ModulesHelp* procedure and not the rest of the modulefile content.

When **load** is asked on an already loaded modulefiles, Modules v4 ignores
this new load order whereas v3.2 refreshed shell alias definitions found
in this modulefile.

When **switching** on version 4 an *old* modulefile by a *new* one,
no error is raised if *old* modulefile is not currently loaded. In this
situation v3.2 threw an error and abort switch action. Additionally on
**switch** sub-command, *new* modulefile does not keep the position held
by *old* modulefile in loaded modules list on Modules v4 as it was the
case on v3.2. Same goes for path-like environment variables: replaced
path component is appended to the end or prepended to the beginning of
the relative path-like variable, not appended or prepended relatively to
the position hold by the swapped path component.

During a **switch** command, version 4 interprets the swapped-out modulefile
in *unload* mode, so the sub-modulefiles loaded, with ``module load``
order in the swapped-out modulefile are also unloaded during the switch.

Modules 4 provides path element counting feature which increases a reference
counter each time a given path entry is added to a given environment
variable. This feature also applies to the ``MODULEPATH`` environment
variable. As consequence a modulepath entry element is removed from the
modulepath enabled list only if the related element counter is equal to 1.
When **unusing** a modulepath if its reference counter is greater than 1,
modulepath is kept enabled and reference counter is decreased by 1.

On Modules 3.2 paths composing the ``MODULEPATH`` environment variable
may contain reference to environment variable. These variable references
are resolved dynamically when ``MODULEPATH`` is looked at during module
sub-command action. This feature has been discontinued on Modules v4.

Following Modules sub-commands are not supported anymore on v4.0:

* ``clear``
* ``update``


Modules specific Tcl commands
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Modules v4 provides path element counting feature which increases a reference
counter each time a given path entry is added to a given environment
variable. As a consequence a path entry element is not always removed
from a path-like variable when calling to ``remove-path`` or calling to
``append-path`` or ``append-path`` at unloading time. The path element is
removed only if its related element counter is equal to 1. If this counter
is greater than 1, path element is kept in variable and reference counter
is decreased by 1.

On Modules v4, **module-info mode** returns during an **unload** sub-command
the ``unload`` value instead of ``remove`` on Modules v3.2.  However if
*mode* is tested against ``remove`` value, true will be returned. During a
**switch** sub-command on Modules v4, ``unload`` then ``load`` is returned
instead of ``switch1`` then ``switch2`` then ``switch3`` on Modules
v3.2. However if *mode* is tested against ``switch`` value, true will
be returned.

When using **set-alias**, Modules v3.2 defines a shell function when
variables are in use in alias value on Bourne shell derivatives, Modules
4 always defines a shell alias never a shell function.

Some Modules specific Tcl commands are not supported anymore on v4.0. When
still using them, a warning message is displayed and these unsupported Tcl
commands are ignored. Following Modules specific Tcl commands are concerned:

* ``module-info flags``
* ``module-info trace``
* ``module-info tracepat``
* ``module-info user``
* ``module-log``
* ``module-trace``
* ``module-user``
* ``module-verbosity``


Further reading
---------------

To get a complete list of the differences between Modules v3.2 and v4,
please read the :ref:`diff_v3_v4` document.

A significant number of issues reported for v3.2 have been closed on v4.
List of these closed issues can be found at:

https://github.com/cea-hpc/modules/milestone/1?closed=1
