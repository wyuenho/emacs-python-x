python-exec-find.el
===================

Greatings fellow Pythonistas and Emacs users!

Have you ever worked on a project that uses one of the many Python package
managers and/or virtual environments, where all the linters, formatters and
commit hooks are setup meticulously, and then when you fire up Emacs, packages
like `flycheck <https://www.flycheck.org/en/latest/>`_ or `lsp-mode
<https://emacs-lsp.github.io/lsp-mode/>`_ are either unable to find the binary
in your virtualenv, or are using a wrong one?

Have you ever tried one of the 11+ Emacs virtualenv packages to help you fix
this problem, but are still at a lost at why your other favorite Emacs packages
still can't find the right binaries, or they stop working when you switch to a
different project using a different flavor of virtualenv?

If you answer "yes" for any of these questions, you've come to the right place.


How does ``python-exec-find`` work?
-----------------------------------

The first key insight is to recognize the executables that many of these linting
and formatting Emacs packages rely on are configurable.

The second key insight is Emacs allows you to setup a different value for the
exectuable path on a per buffer basis.

The hardest problem is finding the correct binary, this is what ``python-exec-find``
tries to solve.

As long as you use one of the supported Python virtualenv tools, ``python-exec-find``
will be able to find the virtualenv root and binary you ask for, with **zero
Emacs configuration** necessary.

``python-exec-find`` works well with popular source code project management packages
such as `Projectile <https://docs.projectile.mx/projectile/index.html>`_ and the
built-in ``project.el``. The first time you call one the few ``python-exec-find`` helper
functions, it will use Projectile or project.el to detect the root of your
project, search for the configuration files for the many supported Python
virtualenv tools, and then lookup the location of the virtualenv based on the
content of the configuration files. Once a virtualenv is found, all executables
are found by looking into its ``bin`` directory.


Python Virtual Environment Tooling Support
------------------------------------------

- `pre-commit <https://pre-commit.com/>`_
- `poetry <https://python-poetry.org/>`_
- `pyenv <https://github.com/pyenv/pyenv>`_
- `direnv <https://direnv.net/>`_
- `pipx <https://pypa.github.io/pipx/>`_
- Whatever is on your ``exec-path``


Supported Emacs Packages
------------------------

- Built-in `project.el <https://www.gnu.org/software/emacs/manual/html_node/emacs/Projects.html>`_
- `projectile <https://docs.projectile.mx/projectile/index.html>`_
- `flycheck <https://www.flycheck.org/en/latest/>`_
- `lsp-jedi <https://github.com/fredcamps/lsp-jedi>`_
- `lsp-pyright <https://github.com/emacs-lsp/lsp-pyright>`_
- `dap-python <https://emacs-lsp.github.io/dap-mode/page/configuration/#python>`_
- `python-black <https://github.com/wbolster/emacs-python-black>`_
- `python-isort <https://github.com/wyuenho/emacs-python-isort>`_
- `python-pytest <https://github.com/wbolster/emacs-python-pytest>`_


System Requirements
-------------------

Currently ``python-exec-find`` requires a program to convert TOML to JSON, a
program to convert YAML to JSON, and ``sqlite3`` to be installed on your system.

By default, both the TOML to JSON and YAML to JSON converters are configured to
use `dasel <https://github.com/TomWright/dasel>`_.  If you are on Linux, it may
be more convenient to use `tomljson
<https://github.com/pelletier/go-toml#tools>`_ and `yq
<https://github.com/mikefarah/yq>`_ since both of which are likely to be
available on the system package manager.

When a suitable Emacs Lisp YAML and TOML parser becomes available, ``dasel``
will be made optional. Likewise, when Emacs 29 is released, the ``sqlite3``
system requirement will be made optional.


Usage
-----

If you are using Emacs on macOS, to get the most out of ``python-exec-find``, it is best
paired with `exec-path-from-shell
<https://github.com/purcell/exec-path-from-shell>`_. Once you have your
``exec-path`` synced up to your shell's ``$PATH`` environment variable, you can
use the following ways to help you setup the rest of your Emacs packages
**properly**.


Basic Setup
+++++++++++

Generally, the following snippet is all you'll need:

.. code-block:: elisp

   (global-python-exec-find-minor-mode 1)


Or, if you use `use-package <https://github.com/jwiegley/use-package>`_:

.. code-block:: elisp

   (use-package python-exec-find
     :config
     (global-python-exec-find-minor-mode 1))


This will setup the buffer local variables for all of the `Supported Emacs
Packages`_.


Advanced Usage
++++++++++++++

If you need to configure a package that ``python-exec-find`` doesn't support, or only
want to configure a couple of packages instead of all the supported one,
``python-exec-find`` offers 2 autoloaded functions to help you find the correct path to
the executable and virtualenv directory:

- ``(python-exec-find-executable-find EXECUTABLE)``
- ``(python-exec-find-virtualenv-root)``

For example, to set up ``python-mode`` to use the correct interpreter when you
execute ``M-x run-python``:

.. code-block:: elisp

   (add-hook 'python-mode-hook
             (lambda ()
               (setq-local python-shell-interpreter (python-exec-find-executable-find "python")
                           python-shell-virtualenv-root (python-exec-find-virtualenv-root))))


For ``flycheck``, due to its complexity, ``python-exec-find`` also comes with another
autoloaded function to help you setup the ``flake8``, ``pylint`` and ``mypy``
checkers:

.. code-block:: elisp

   (add-hook 'python-mode-hook 'python-exec-find-flycheck-setup)


Complete Example
++++++++++++++++

.. code-block:: elisp

   (require 'quelpa-use-package)

   (use-package exec-path-from-shell
     :if (memq (window-system) '(mac ns))
     :config (exec-path-from-shell-initialize))

   (use-package flycheck)

   (use-package lsp-jedi)

   (use-package lsp-pyright
     :after lsp)

   (use-package dap-python)

   (use-package python-pytest)

   (use-package python-black)

   (use-package python-isort)

   (use-package python-exec-find
     :quelpa (python-exec-find :fetcher github :repo "wyuenho/emacs-python-exec-find")
     :ensure-system-package (dasel sqlite3)
     :config
     (add-hook 'python-mode-hook
               (lambda ()
                 (setq-local python-shell-interpreter (python-exec-find-executable-find "python")
                             python-shell-virtualenv-root (python-exec-find-virtualenv-root))

                 (python-exec-find-flycheck-setup)

                 (setq-local lsp-jedi-executable-command
                             (python-exec-find-executable-find "jedi-language-server"))

                 (setq-local lsp-pyright-python-executable-cmd python-shell-interpreter
                             lsp-pyright-venv-path python-shell-virtualenv-root)

                 (setq-local dap-python-executable python-shell-interpreter)

                 (setq-local python-pytest-executable (python-exec-find-executable-find "pytest"))

                 (when-let ((black-executable (python-exec-find-executable-find "black")))
                   (setq-local python-black-command black-executable)
                   (python-black-on-save-mode 1))

                 (when-let ((isort-executable (python-exec-find-executable-find "isort")))
                   (setq-local python-isort-command isort-executable)
                   (python-isort-on-save-mode 1)))))


License
-------

`GPLv3 <./LICENSE>`_
