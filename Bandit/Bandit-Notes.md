#Bandit 

0 Level
ssh connection to the server login@domain -p port

0-1 Level
ls command - to see all directories
cat readme - to read the file with password

1-2 Level
ls - we see the file with the name "-" but we cant just open it, we need to open this with using this current directory
cat ./(this directory)- - we get the password

2-3 Level
ls - we see the filename --spaces in this filename-- and we have 2 problems here. The first one is dashes in the filename and second one is spaces in the filename. We already know how to work with dashes. For spaces we can use back slash "\" before space
cat ./--spaces\ in\ this\ filename--

3-4 Level
ls - we see the directory with name "inhere", after opening this directory we have to use ls -la command to see all hidden files and we see hidden file with name "...Hiding-From-You"
cat ./...Hiding-From-You