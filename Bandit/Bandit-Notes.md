# Bandit

> Passwords are stored locally and are not published here, as OverTheWire asks.

## Level 0
Connect to the server over SSH:
`ssh login@domain -p port`

## Level 0 → 1
- `ls` – lists files and directories.
- `cat readme` – reads the file with the password.

## Level 1 → 2
`ls` shows a file named `-`, but we can't open it directly, because many
commands treat `-` as standard input. We need to use a relative path to the
current directory:
`cat ./-`

## Level 2 → 3
`ls` shows a file named `--spaces in this filename--`, and there are two
problems here. The first one is the dashes in the filename, and the second
one is the spaces. We already know how to deal with dashes. For spaces, we
can put a backslash `\` before each space:
`cat ./--spaces\ in\ this\ filename--`

## Level 3 → 4
`ls` shows a directory named `inhere`. Inside it, we need to use `ls -la` to
see hidden files, and we find a hidden file named `...Hiding-From-You`:
`cat ./...Hiding-From-You`

## Level 4 → 5
Inside the `inhere` directory there are 10 files, and only one of them is
human-readable. We can use the `file` command to check each file's type.
`data` means binary data, and `ASCII text` means it is our file. We check
the files and see that `-file07` contains the password:
`cat ./-file07`

## Level 5 → 6
Inside the `inhere` directory there are many subdirectories, so we need to
find the file with the password among many other files. The level
description gives three properties:
- human-readable
- 1033 bytes in size
- not executable

We can use the `find` command with filters:
`find . -type f -size 1033c -readable ! -executable`

## Level 6 → 7
`ls` shows no files or directories. This means the file is hidden somewhere
else on the server, so we search from the root directory `/`. We can go
there with `cd /`, or just run `find /` from any directory.

The file has three properties:
- owned by user bandit7
- owned by group bandit6
- 33 bytes in size

We can use the `find` command again, now with the `-user` and `-group`
filters. We will also get many "Permission denied" errors, because we don't
have access to many directories. To hide them, we redirect the error stream
(stderr) to `/dev/null`:
`find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null`

Then we read the file that was found:
`cat /var/lib/dpkg/info/bandit7.password`

## Level 7 → 8
`ls` shows only one file, `data.txt`. It contains many lines with a word and
a password after it. We know that our password is next to the word
`millionth`, so we can easily get it with `grep`:
`grep "millionth" data.txt`

## Level 8 → 9
`data.txt` contains many lines that repeat, and our password appears only
once. `uniq` compares only neighboring lines, so we first sort the file with
`sort` to put identical lines together, and then use `uniq -u` to keep only
the unique line:
`sort data.txt | uniq -u`

## Level 9 → 10
`data.txt` is a binary file, so we can't read it directly. We know that our
password is preceded by several `=` characters. We can get it by combining
two commands: `strings` extracts all readable strings from the file, and
`grep` finds the right one:
`strings data.txt | grep "=="`

## Level 10 → 11
`data.txt` contains a Base64-encoded string. We can decode it with the
`base64` tool:
`base64 -d data.txt`