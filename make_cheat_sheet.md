## Make cheatsheet

### Makefile Structure

Each block of code in a Makefile is called a rule, it looks something like this:

~~~
file_to_create.pdf : data_it_depends_on.dat script_it_depends_on.py
	python script_it_depends_on.py data_it_depends_on.dat file_to_create.pdf
~~~

* `file_to_create.pdf` is a target, a file to be created, or built.
* `data_it_depends_on.dat` and `script_it_depends_on.py` are dependencies, files which are needed to build or update the target. Targets can have zero or more dependencies.
* `:` separates targets from dependencies.
* `python script_it_depends_on.py data_it_depends_on.dat file_to_create.pdf` is an action, a command to run to build or update the target using the dependencies. Targets can have zero or more actions. Actions are indented using the TAB character, not 8 spaces. 
* Together, the target, dependencies, and actions form a rule.

### Makefile variables

* Use `$@` to refer to the target of the current rule

* Use `$<` to refer to the first dependency of the current rule

* Use `%` as a wild-card in targets and dependencies

* Use `$*` to match the stem with which the rule matched