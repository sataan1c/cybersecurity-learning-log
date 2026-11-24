# Bandit

My notes from solving [OverTheWire Bandit](https://overthewire.org/wargames/bandit/).
For each level I wrote down how I solved it and where I got stuck.

> I don't publish passwords or keys here, because OverTheWire asks players
> not to. I keep them locally.

## Level 0
The first step is just connecting to the server over SSH:
`ssh login@domain -p port`

## Level 0 → 1
A warm-up. `ls` shows what's in the folder, and `cat readme` shows the
password.

## Level 1 → 2
The file is called `-`, and `cat -` doesn't work. I learned that many
commands treat `-` as "standard input", so `cat` just waits for me to type
something. The fix is to give a path to the file, so the command knows it's
a file name:
`cat ./-`

## Level 2 → 3
This time the file name has both dashes and spaces:
`--spaces in this filename--`. I already knew the `./` trick for the dashes.
For spaces, I put a backslash before each one, so the shell doesn't split the
name into separate words:
`cat ./--spaces\ in\ this\ filename--`

Quotes work too, but Tab completion is the easiest way: it adds the
backslashes for you.

## Level 3 → 4
Inside `inhere`, plain `ls` shows nothing. Files that start with a dot are
hidden, so I used `ls -la` and found `...Hiding-From-You`:
`cat ./...Hiding-From-You`

Since then, `ls -la` is the first thing I run in any folder.

## Level 4 → 5
There are 10 files, and only one is human-readable. At first I thought they
were directories, but the `-` at the start of each line in `ls -l` means
they are regular files. The `file` command shows what's inside a file:
`data` means binary, `ASCII text` means readable text.
`file ./*` checks all of them at once, and only `-file07` was text:
`cat ./-file07`

## Level 5 → 6
Now there are lots of folders with lots of files. Checking them one by one
would take forever, so I used `find` with the three properties from the
level description (human-readable, 1033 bytes, not executable):
`find . -type f -size 1033c -readable ! -executable`

My mistake: `find` returned `./maybehere07/.file2`, but I opened `-file2`,
which is a decoy in the same folder. The real file starts with a **dot**.

## Level 6 → 7
The file is "somewhere on the server", so I searched from the root `/` with
filters for the owner, group and size:
`find / -type f -size 33c -user bandit7 -group bandit6 2>/dev/null`

Without `2>/dev/null` the screen fills up with "Permission denied" errors.
That part sends the error stream (stderr) to `/dev/null`, where it just
disappears, and only the real result is left:
`cat /var/lib/dpkg/info/bandit7.password`

## Level 7 → 8
The password is next to the word `millionth`, so `grep` finds that line:
`grep "millionth" data.txt`

## Level 8 → 9
The password is the only line that appears once. `uniq -u` keeps unique
lines, but it only compares lines that are next to each other. So I sort
the file first:
`sort data.txt | uniq -u`

## Level 9 → 10
`data.txt` is binary, so `cat` shows garbage. `strings` pulls out the
readable parts, and `grep` finds the line with `=` signs:
`strings data.txt | grep "=="`

My first try was `grep "==" | strings data.txt`, which is backwards.
A pipe sends data from left to right, so the command that produces the data
has to come first.

## Level 10 → 11
The data is in Base64. It looks like a random mix of letters and numbers,
often with `=` at the end. Base64 is not encryption, anyone can decode it:
`base64 -d data.txt`

## Level 11 → 12
All letters are shifted by 13 positions (ROT13). `tr` swaps each character
from the first set for the character in the same position in the second
set. The second set is just the alphabet starting from `N`:
`cat data.txt | tr 'A-Za-z' 'N-ZA-Mn-za-m'`

Fun fact: the alphabet has 26 letters, so shifting by 13 twice brings you
back. The same command both encodes and decodes.

## Level 12 → 13
The longest level so far. `data.txt` is a hexdump of a file that was
compressed many times, like a set of nesting dolls.

I can't create files in the home folder, so I made a private temporary
folder in `/tmp`:
```
mktemp -d
cp data.txt /tmp/<directory>/
cd /tmp/<directory>
```

Then I turned the hexdump back into a real file:
`xxd -r data.txt > data`

My mistakes here: first I forgot `> data`, so `xxd` printed binary garbage
on the screen. Then I wrote `file data.txt > data`, which saved the output
of `file` instead of the binary data.

After that it was a loop:
1. `file data` – what kind of archive is it?
2. `mv` – rename it to the right extension (`.gz`, `.bz2`).
3. Unpack it with `gzip -d`, `bzip2 -d` or `tar -xf`.
4. `ls` – see what came out.

One thing that confused me: `gzip` and `bzip2` replace the archive with the
unpacked file, but `tar` keeps the archive and extracts files with new names
like `data5.bin`. I also skipped `file` a couple of times and got stuck when
the format changed, so now I check with `file` after every step instead of
guessing.

## Level 13 → 14
No password this time, just a private SSH key. With key-based login, the
server stores the public key, and SSH proves you have the matching private
key, so you don't need a password.

My first idea was to use the key right on the server:
`ssh -i sshkey.private bandit14@localhost -p 2220`
The server didn't allow it: SSH connections to `localhost` are blocked. So I
copied the key to my own computer. Note the uppercase `-P` in `scp`:
`scp -P 2220 bandit13@bandit.labs.overthewire.org:sshkey.private .`

The next error said "permissions are too open". SSH refuses a private key
that other users can read, so I made it readable only by me:
`chmod 600 sshkey.private`

Then the login worked:
`ssh -i sshkey.private bandit14@bandit.labs.overthewire.org -p 2220`

As bandit14, I could finally read the password for this level:
`cat /etc/bandit_pass/bandit14`

The level says "read the error messages", and each error told me exactly what
was wrong.

## Level 14 → 15
I need to send the current password to port 30000 on `localhost`. At first
I thought I had to log in there, but it's not SSH. It's a small program that
waits for text. `nc` (netcat) just opens a connection to a port and sends
what I type:
`nc localhost 30000`

I pasted the password, and it answered with the next one.

## Level 15 → 16
Same idea, but on port 30001 the service only talks over SSL/TLS, so `nc`
doesn't work. `openssl s_client` connects like `nc`, but sets up encryption
first:
`openssl s_client -connect localhost:30001 -quiet`

All that long output without `-quiet` is the TLS handshake: the certificate
and encryption details.

## Level 16 → 17
The password goes to one of the ports between 31000 and 32000, and I have to
find the right one. Step one: which ports are open? By default `nmap` scans
only common ports, so I set the range:
`nmap localhost -p 31000-32000`

Five ports were open. Step two: which ones speak SSL/TLS? `-sV` makes `nmap`
actually talk to each service:
`nmap localhost -p 31046,31518,31691,31790,31960 -sV`

How I picked the port:
- `echo` – just sends back whatever you send it;
- `ssl/echo` – the same, over TLS;
- `ssl/unknown` on 31790 – a TLS service that answers everything with
  "Wrong! Please enter the correct current password". That's the one.

When I sent the password, I got `KEYUPDATE` instead of an answer. It turns
out `s_client` treats a line starting with `k` as its own command, and my
password started with `k`. `-quiet` fixes this. I also forgot `-connect`
once and got an "Extra option" error:
`openssl s_client -connect localhost:31790 -quiet`

The answer wasn't a password but another private SSH key. I saved it and
logged in like on level 13. This time I got stuck on permissions again:
I saved the key on the Windows disk (`/mnt/c/...`) in WSL, and `chmod 600`
just didn't change anything there. Moving the key to my Linux home folder
(`~`) fixed it:
```
chmod 600 bandit17.key
ssh -i bandit17.key bandit17@bandit.labs.overthewire.org -p 2220
```

## Level 17 → 18
Two files, `passwords.old` and `passwords.new`, and the password is the line
that changed. That's what `diff` is for:
`diff passwords.old passwords.new`

`42c42` means line 42 changed. The line after `<` is from the first file,
and the line after `>` is from the second one. The password is after `>`.

## Level 18 → 19
Someone changed `.bashrc`, so the server kicks me out right after login.
The trick is that `ssh` can run one command on the server without opening
a shell. I found it in the SYNOPSIS of `man ssh`: anything after the address
is treated as a command.
`ssh bandit18@bandit.labs.overthewire.org -p 2220 cat readme`

The command runs, prints the result and disconnects, and `.bashrc` never gets
a chance to kick me out.

## Level 19 → 20
`bandit20-do` has the **setuid** permission (`s` instead of `x` in `ls -l`).
A setuid program runs with the permissions of its **owner**, not the person
who starts it. Here the owner is bandit20.

`./bandit20-do whoami` printed `bandit20`, which proved it. At first I tried
to SSH as bandit20 through it, which was overcomplicated. I just needed to
read the file:
`./bandit20-do cat /etc/bandit_pass/bandit20`

On a real system, setuid files can be found with
`find / -perm -4000 2>/dev/null`.

## Level 20 → 21
`suconnect` connects to a port I choose, reads one line and, if it's the
bandit20 password, sends back the next one.

The twist: until now **I** connected to services. Here the program connects,
so I have to be the server. I started `nc` in listening mode in the
background and gave it the password through a pipe:
`echo <bandit20 password> | nc -l -p 32864 &`

Then I checked that it was running with `jobs` and started the program:
`./suconnect 32864`

It took me a few tries:
- I wrote `localhost:3000`, but `nc` takes the port separately, not with
  a colon like `openssl`;
- I put `echo` on the right side of the pipe, but it has to be on the left;
- I wrote `echo > password`, which tried to create a file instead of sending
  the text. Since nothing was sent, `suconnect` just sat there waiting;
- I tried to do it while logged in as bandit19.

## Level 21 → 22
A program runs on a schedule with **cron**. System cron jobs live in
`/etc/cron.d/`:
```
ls /etc/cron.d/
cat /etc/cron.d/cronjob_bandit22
```

The job runs `/usr/bin/cronjob_bandit22.sh` every minute as bandit22.
Scripts are just text files, so I read it:
`cat /usr/bin/cronjob_bandit22.sh`

The script copies the bandit22 password into a file in `/tmp` and makes it
readable for everyone (`chmod 644`). So I didn't need to run anything, just
read that file:
`cat /tmp/<file name from the script>`

I also understood why running the script myself wouldn't help: it would run
as bandit21 and get "Permission denied". Cron runs it as bandit22, and
that's what makes it work.

## Level 22 → 23
Another cron job, but this time the script doesn't just copy the password to
a fixed file. I read it with `cat /usr/bin/cronjob_bandit23.sh` and worked
through it line by line:
```
myname=$(whoami)
mytarget=$(echo I am user $myname | md5sum | cut -d ' ' -f 1)
cat /etc/bandit_pass/$myname > /tmp/$mytarget
```
- `whoami` is the user running the script – **bandit23**, because that's who
  cron runs it as.
- The second line builds the file name: it takes the phrase
  `I am user bandit23`, hashes it with `md5sum`, and `cut -d ' ' -f 1` keeps
  only the hash (the first field, split on spaces).
- The last line copies bandit23's password into `/tmp/<that hash>`.

So the file name isn't a secret – it's just the hash of a phrase I can build
myself. I ran the same pipeline by hand to get the name, then read the file:
`echo I am user bandit23 | md5sum | cut -d ' ' -f 1`

My mistake: I first wrote `$bandit23` with a dollar sign. In bash `$name`
means "the value of variable name", and there's no such variable, so it
became empty and the hash was wrong. The name goes in as plain text, no `$`.

## Level 23 → 24
The hardest one so far, and the first where I had to write my own script.
The cron job runs `/usr/bin/cronjob_bandit24.sh` as bandit24. I read it and
saw it goes into `/var/spool/bandit24/foo`, and for every file **owned by
bandit23** it runs the file and then deletes it:
```
if [ "${owner}" = "bandit23" ] && [ -f "$i" ]; then
    timeout -s 9 60 "./$i"
fi
rm -rf "./$i"
```
The script itself doesn't copy any password – it just runs other people's
files as bandit24. So the useful part was mine to write: a script that reads
bandit24's password and saves it where I can read it.

What I did:
1. Made a private working folder in `/tmp` with `mktemp -d` and opened it up
   so bandit24 could write there: `chmod 777 /tmp/<dir>`.
2. Wrote a two-line script there:
   ```
   #!/bin/bash
   cat /etc/bandit_pass/bandit24 > /tmp/<dir>/pass.txt
   ```
3. Made it executable: `chmod 755 myscript.sh`.
4. Copied it into `/var/spool/bandit24/foo/`. `cp` makes me (bandit23) the
   owner, which is exactly the condition the cron script checks.
5. Waited a minute and read `/tmp/<dir>/pass.txt`.

This fought me for a while:
- I had a typo in the path: `/etc/bandit_pas24` instead of
  `/etc/bandit_pass/bandit24`. The nasty part is that `>` creates the output
  file even when the command fails, so `pass.txt` showed up **empty** instead
  of giving an error. An empty result file means the command is wrong, not
  the permissions.
- My wait-loop `until cat ...` stopped on the empty file and printed nothing.
  I switched to `until [ -s file ]`, which waits for a **non-empty** file.
- I kept checking too early. `* * * * *` means cron runs at the start of each
  minute by the server clock, not 60 seconds after my copy, so the wait is
  anywhere from a few seconds to almost a minute. `date` shows the server's
  seconds if I want to know how long is left.

## Level 24 → 25
A daemon on port 30002 gives the bandit25 password if I send the bandit24
password plus a secret 4-digit pincode. There's no clever trick for the pin:
I have to try all 10000 combinations (0000–9999). This is **brute-forcing**.

The key hint was "you do not need to create new connections each time" – the
daemon reads many attempts in one connection, so I don't open 10000 separate
connections.

I generated all 10000 lines with `seq`, piped them into `nc`, and filtered
out the "Wrong!" replies to find the one real answer:
`seq -f "<bandit24 password> %04g" 0 9999 | nc localhost 30002 | grep -v Wrong`

- `%04g` prints each number as 4 digits with leading zeros (`0000`, `0007`,
  `9999`), so each line comes out as `password 0000` … `password 9999`.
- `nc` sends all of them in one connection.
- `grep -v Wrong` drops the "Wrong!" replies and leaves the one real answer.

## Level 25 → 26
Logging in as bandit26 is easy – there's an SSH key, like on level 13. The
catch: bandit26's shell is not bash. I found it in `/etc/passwd` (the last
field of each line is the user's shell):
`bandit26:...:/usr/bin/showtext`

`showtext` is a small script:
```
#!/bin/sh
export TERM=linux
exec more ~/text.txt
exit
```
It shows a text file with `more`, and `exec` **replaces** the shell with
`more`, so when `more` finishes, the whole session closes. That's why a
normal login kicks me out instantly.

The trick is to stop `more` from finishing. `more` shows text page by page:
if the file fits on screen it exits right away, but if it **doesn't** fit it
stops at `--More--` and waits. So I made the terminal window tiny (just a few
lines) before logging in. Then the text didn't fit and `more` waited for me.
(The level warns Windows users to use cmd, not PowerShell, because PowerShell
handles the window size differently and the trick fails.)

While `more` was waiting, I escaped through its built-in commands:
1. Pressed `v` – this opens the current file in the `vi` editor. `more`, like
   `less`, can launch an editor while it waits.
2. In `vi`, which runs commands from its command mode (lines starting with
   `:`), I ran:
   ```
   :set shell=/bin/bash
   :shell
   ```
   The first line tells `vi` to use bash (otherwise it would launch the
   restricted `showtext` again). The second line launches it. I ended up in a
   normal bash shell, running as bandit26.
3. Read the password: `cat /etc/bandit_pass/bandit26`.

A few things that took me a while to see:
- A user's login shell is just a program, and here it's a script, not bash.
- `exec` is why the session dies: the script becomes `more`, so there's
  nothing to return to when `more` exits.
- `vi` isn't only an editor – from command mode it can run any system command
  (`:!cmd`) or open a shell (`:shell`). I wasn't editing the file; `:`
  commands are instructions to the editor, not text (that needs `i` first).

This is escaping a **restricted shell**: if a locked-down shell lets you reach
a pager or editor (`more`, `less`, `man`, `vi`, …), you can usually break out
into a full shell. GTFOBins lists how to do it for many programs.

## Level 26 → 27
Same `showtext`/`more` trap as the last level, so I broke out the same way:
tiny window → `more` waits → `v` to open `vi`. This time I didn't even need a
full shell. bandit27 has its own `bandit27-do`, so from `vi` I set bash as the
shell, got a prompt, and used the setuid helper like on level 19:
`./bandit27-do cat /etc/bandit_pass/bandit27`

## Level 27 → 28
The password is in a git repository on the server. The level says to clone it
**from my own machine**, not from the OverTheWire server, so I did it in WSL.

Git over SSH doesn't take `-p` for the port – the port goes **inside** the
URL, right after the host:
`git clone ssh://bandit27-git@bandit.labs.overthewire.org:2220/home/bandit27-git/repo`

My first try failed with "Permission denied (publickey)" and a message that I
connected on port 22. That's because I left the port out, so git used the
default 22 instead of 2220. With `:2220` in the URL it worked. The password
for `bandit27-git` is the same as for bandit27.

After cloning, the password was just sitting in `README`:
`cd repo && cat README`

## Level 28 → 29
Cloned the repo the same way, but this time `README` had the password blanked
out (`xxxxxxxxxx`). Git keeps every version of a file, so an old password that
was "removed" is still in the history.

`git log` showed a commit called **"fix info leak"**, which was a dead
giveaway. I opened it:
`git show <commit hash>`

The diff showed the change: the line starting with `-` (removed) was the real
password, and the `+` line was the `xxxxxxxxxx` that replaced it. So deleting a
secret from a file isn't enough – it stays in the git history, which is exactly
why I don't commit passwords to my own notes.

## Level 29 → 30
Cloned again. This time `README` on the main branch (master) had
`<no passwords in production!>` instead of a password, and its whole history
only showed that placeholder. The hint from the series: the password is hidden
in another **branch**.

Branches are parallel lines of work in one repo. `git log` only shows the
current one, so I used:
`git log -p --all`

`-p` shows the diff of each commit and `--all` covers every branch at once, so
I didn't have to check branches one by one. In the **dev** branch
(`HEAD -> dev`) a commit "add data needed for development" added the real
password (the `+` line), while master only ever had the placeholder. So a
secret can hide in any branch, not just master – `git log -p --all` (or
`git branch -a` to list branches first) shows everything at once.

## Level 30 → 31
Cloned the repo and `README.md` was empty, just a commit saying
`just an empty file... muahaha`. `git log -p --all` had only one commit and
no branches, so the password wasn't in a commit or a branch. That left tags.

A tag is a label stuck on a commit, and it doesn't show up in `git log` or in
the branch list, which is why I kept missing it. Listed them and opened the
one I found:
```
git tag
git show <tag name>
```
`git tag` gave me `secret`, and `git show secret` had the password.

## Level 31 → 32
No more reading – this time I had to push a file. The README told me exactly
what: file `key.txt`, content `May I come in?`, branch `master`.

The catch was a `.gitignore` with `*.txt` in it, so git quietly ignores every
`.txt` file and a plain `git add` adds nothing. I forced it with `-f`:
```
echo 'May I come in?' > key.txt
git add -f key.txt
git commit -m "add key"
git push
```

Two things tripped me up. First git wouldn't commit at all – "Author identity
unknown", because my WSL git had no name or email yet, so I set them once.
Then my first push came back "Wrong!" from the server, because the file
content didn't match exactly. Recreated it with `echo`, pushed again, and the
bandit32 password came back in the `remote:` lines.

## Level 32 → 33
Logged in and landed in a weird shell with a `>>` prompt that SHOUTS
everything back – it turns whatever I type into uppercase. So `ls` becomes
`LS`, nothing is found, and I'm stuck.

The way out is something with no uppercase version. `$0` is the name of the
current shell, and `0` is a digit, so uppercase doesn't touch it. Typing `$0`
just starts the shell again, this time a normal one:
```
$0
```
Normal prompt, `whoami` said bandit33, done:
`cat /etc/bandit_pass/bandit33`

## Done
That's the end – level 33's `README.txt` is just a congratulations message,
no more levels. All of Bandit, 0 through 33.
