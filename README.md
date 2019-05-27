# fix-codestyle-on-commit
Runs phpcs on commit, if needed.

[![Open Source Love](https://badges.frapsoft.com/os/mit/mit.svg?v=102)](https://github.com/PatricNox/fix-codestyle-on-commit)

# Requirements

* Git >= 2.x
* PHP >= 5.3
* Composer >= 1.5.x
* [PHPCS](https://github.com/squizlabs/PHP_CodeSniffer)

# Setup
## File structure

```
[ROOT]
  vendor/
             .....
             .....
  _ORGANISATION/
             pre-commit
  _scripts/
             phpcs-fix-on-commit.sh
  composer.json
  composer.lock
  readme.md

```

> **In file**: _composer.json_

```
  ],
    "post-install-cmd": [
      "bash _scripts/phpcs-fix-on-commit.sh"
  ]
```

> **In file**: _phpcs-fix-on-commit.sh_

```
  #!/bin/sh

  cp _ORGANISATION/pre-commit ../.git/hooks/pre-commit
  chmod +x ../.git/hooks/pre-commit

```

> **In file**: _pre-commit_

```
#!/bin/sh

THIS_PROJECT=`php -r "echo dirname(dirname(dirname(realpath('$0'))));"`
GIT_STAGED_FILES=`git diff --cached --name-only --diff-filter=ACMR HEAD | grep \\\\.php`

# Determine if there's staged files before commiting.
if [ "$#" -eq 1 ]
then
	oIFS=$IFS
	IFS='
	'
	SFILES="$1"
	IFS=$oIFS
fi

# Make sure we got staged files.
SFILES=${SFILES:-$GIT_STAGED_FILES}
for FILE in $SFILES
do
	FILES="$FILES $THIS_PROJECT/$FILE"
done

# Run phpcbf on all files, if we got staged files.
if [ "$FILES" != "" ]
then
    # Check if there's something to fix before fixing.
	src/vendor/bin/phpcs --standard="PSR2" --encoding=utf-8 -n -p $FILES
    if [ $? != 0 ]
    then
        # We got something to fix.
        echo "_______________"
        echo "Fixing code standard with phpcs."
        echo "_______________"
        src/vendor/bin/phpcbf --standard="PSR2" --encoding=utf-8 -n -p $FILES
        if [ $? == 0 ]
        then
            echo "Could not fix errors, please fix the error before committing."
            exit 1
        else
            # Stage fixed files.
            git add $FILES
        fi
    fi
fi

echo ""
echo "## -- Committed! -- ##"
echo "----------------------"
exit $?

```

# License
[MIT](LICENSE) @ PatricNox
