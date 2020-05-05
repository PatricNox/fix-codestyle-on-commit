# Check codestyle on commit
Runs phpcs on commit, stopping it from becoming a commit incase there's code that doesn't follow the coding standard.

[![Open Source Love](https://badges.frapsoft.com/os/mit/mit.svg?v=102)](https://github.com/PatricNox/fix-codestyle-on-commit)

# Requirements

* Git >= 2.x
* PHP >= 5.3
* Composer >= 1.5.x
* [PHPCS](https://github.com/squizlabs/PHP_CodeSniffer)

# Setup
## Git hooks

Edit `PHPCS_PATH` to the correct path, if applicable.

In this example, I have a docker setup which serves the application from _/laravel_.

> **In file**: _.git/hooks/pre-commit_

```
#!/bin/sh

# PHPCS Path (Edit if applicable)
PHPCS_PATH='laravel/vendor/bin/phpcs'

# Project path, no need to edit.
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

# Run phpcs on all files, if we got staged files.
if [ "$FILES" != "" ]
then
    # Check the staged files for code standard.
    $PHPCS_PATH --standard="PSR2" --encoding=utf-8 -n -p $FILES
    if [ $? != 0 ]
    then
        # We got something to fix.
        echo "_______________"
        echo "Found staged file(s) that doesn't follow the coding standard."
        echo "_______________"
	exit 1
    fi
    else
        # Make them committable.
        git add $FILES
    fi
fi

echo ""
echo "## -- Committed! -- ##"
echo "----------------------"
exit $?

```

# License
[MIT](LICENSE) @ PatricNox
