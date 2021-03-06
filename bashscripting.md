# Bash Scripting

## Pipes: send the result of one process to another
```bash
ls | wc -l
```

## Redirections: send streams to or from files
```bash
ls > list.txt
ls /notreal 1>output.txt 2>error.txt  # if gives error redirect to error.txt
echo "some text"
printf "some text"  # without \n at the end, same as echo
command -V echo  # shows the command is builtin or not; builtins take precedence over commands!
enable -n echo  # specific builtins can be enabled
enable -n  # shows builtins disabled
enable echo  # enables the builtin command again
```

# Parantheses (), Braces {}, Brackets []

## Brace expansion
```bash
echo /tmp/{one,two,three}/file.txt
# /tmp/one/file.txt /tmp/two/file.txt /tmp/three/file.txt

echo c{a,o,u}t
# cat cot cut

echo /tmp/{1..3}/file.txt
# /tmp/1/file.txt /tmp/2/file.txt /tmp/3/file.txt

echo {00..10}
# 00 01 02 03 04 05 06 07 08 09 10

echo {a..z}
# a b c d e f g h i j k l m n o p q r s t u v w x y z

echo {a..z..2}  # can be set with intervals
# a c e g i k m o q s u w y

touch file_{01..10}{a..d}
```

## Parameter expansion: ${...} retrieves and transforms stored values
```bash
greeting="hello there"
echo ${greeting}
# hello there
echo ${greeting:6}
# there
echo ${greeting:6:3}
# the
echo ${greeting/there/everybody}
# hello everybody
echo ${greeting//e/_}
# h_llo th_r_
echo ${greeting/e/_}
# h_llo there
```

## Command Substitution: $(...) puts the output of one command inside another
```bash
uname -r
# 4.4.0-19041-Microsoft
echo "The kernel is $(uname -r)."
# The kernel is 4.4.0-19041-Microsoft.
echo $(python3 -c 'print("Hello from Python")')
# Hello from Python
echo "Result: $(python3 -c 'print("Hello from Python")')" | tr [a-z] [A-Z]
# RESULT: HELLO FROM PYTHON
```

## Arithmetic expansion: $((...)) does math
```bash
echo $(( 2 + 3 ))

echo $(( 1 / 3 ))
# 0  # to deal with that:

declare -i c=1
declare -i d=3
e=$(echo "scale=3; $c/$d" | bc)  
# The bc command, short for basic calculator, is a language that supports arbitrary precision numbers with interactive execution of statements.
# scale is number of digits after decimal point
# .333
```

## Arithmetic evaluation: ((...)) performs calculations and changes the value of variables
```bash
a=3
(( a++ ))
echo $a
```

## Compare things / test values: [ ... ]  # 0 for success, 1 for failure
```bash
[ -d ~ ] 
echo $?
# 0

[ -d /bin/bash ]; echo $?  # check if /bin/bash is a directory or not; but is a file
# 1

help test  # to find exotic ones
[ ! 4 -lt 3 ]; echo $?  
# 0  # ! gives the opposite result
```

## Extended test: [[ ... ]]  # can be used more than one expression
```bash
[[ -d ~ && -a /bin/bash ]]; echo $?
# 0  # ~ is a directory and bash binary exists at the same time
  - && and
  - || or
[[ "cat" =~ c.* ]]; echo $?
# 0

nano script.sh
#!/usr/bin/env bash	# alt: #!/bin/bash using env command, it could be in a different location and still can work 
echo "hello"
chmod +x script.sh
./script.sh
```

## Debugger mode:
bash -x ./script.sh

```bash
#!/usr/bin/env bash
set -x			# Only first part of the code will be debugged!
for i in {1..10}
do
	echo $i
done
set +x
for i in {a..z}
do
	echo $i
done
```

## echo -n "No newline"  -n option deletes unvisible new line

# declare -r myname="Rafe"  # declare read only variable
  declare -l  # uppercase the var
  decclare -u  # lovercase the var
  declare -p  # show all variables declared during session
  declare -i  # declare an integer, instead of string value

# env  # all environment variables, look for them and echo them

# echo $RANDOM  # gives a random number between 0 to 32767
  echo $(( 1 + $RANDOM % 10 ))  # gives a random number between 1 to 10, make sure not 0 added 1 +

# echo -e ...  # Interprets escaped characters like \t, \n, \a, and other control characters
  echo -e "Name\t\tNumber"; echo -e "Scott\t\t123"
  Name            Number
  Scott           123
  echo -e "This text\nbreaks over\nthree lines"
  This text
  breaks over
  three lines
  echo -e "\a"  # Makes sound
  echo -e "\033[33;44mColor Text\033[0m"  # 033 is foreground, 33 yellow, 44 blue background, [0m color finishes with the line

# Arrays:
  - declare -a snacks=("apple" "banana" "orange")
    echo ${snacks[2]}
    orange
    snacks[5]="grapes"
    snacks+=("mango")
    echo ${snacks[@]}
    apple banana orange mango grapes
  - for i in {0..6}; do echo "$i: ${snacks[$i]}"; done  # want to see every index, including empty ones
    0: apple
    1: banana
    2: orange
    3: mango
    4:
    5: grapes
    6: strawberry
  - declare -A offices  # -A to make NAMEs associative arrays (if supported)
    offices[city]="San Fransisco"
    offices["building name"]="HQ West"
    echo ${offices["building name"]} is in ${offices[city]}
    HQ West is in San Fransisco

# Challenge:
  - Compose a script to show some system information
  - Use some standard tools, like df, free, or others
  - Use awk or sed to extract text from output, if you know them
  - Use formatted text

```bash
#!/usr/bin/env bash
# A script to output a brief summary of system information
#  - df -h
#  Filesystem      Size  Used Avail Use% Mounted on
#  rootfs          223G  167G   57G  75% /
#  none            223G  167G   57G  75% /dev
#  none            223G  167G   57G  75% /run
#  none            223G  167G   57G  75% /run/lock
#  none            223G  167G   57G  75% /run/shm
#  none            223G  167G   57G  75% /run/user
#  tmpfs           223G  167G   57G  75% /sys/fs/cgroup
#  - df -h /
#  Filesystem      Size  Used Avail Use% Mounted on
#  rootfs          223G  167G   57G  75% /
#  - df -h / | awk '{print $4}'
#  Avail
#  57G
#  - df -h / | awk 'NR==2 {print $4}'
#  57G

freespace=$(df -h / | awk 'NR==2 {print $4}')

#  - free -h
#                total        used        free      shared  buff/cache   available
#  Mem:            15G        5.0G         10G         17M        223M         10G
#  Swap:           27G         38M         27G
#  - free -h | awk 'NR==2 {print $4}'
#  10G

freemem=$(free -h | awk 'NR==2 {print $4}')

greentext="\033[32m"
bold="\033[1m"
normal="\033[0m"

printf -v logdate "%(%Y-%m-%d)T"  # print out the time and assign it to logdate variable

#  - echo $HOSTNAME
#  DESKTOP-NIM1BCV
#  - uname -r
#  4.4.0-19041-Microsoft
#  - echo $BASH_VERSION
#  4.4.19(1)-release
#  - ls | wc -l
#  19

echo -e $bold"Quick system report for "$greentext"$HOSTNAME"$normal
printf "\tKernel Release:\t%s\n" $(uname -r)
printf "\tBash Version:\t%s\n" $BASH_VERSION
printf "\tFree Storage:\t%s\n" $freespace
printf "\tFree Memory:\t%s\n" $freemem
printf "\tFiles in pwd:\t%s\n" $(ls | wc -l)
printf "\tGenerated on:\t%s\n" $logdate
```

## Conditionals:
```bash
if ...condition...[[...]]...true
then
   ...
elif
   ...
else
   ...
fi
```

## Loops: while, until, for

-------------------------------
while/until ...
do
   ...
done
-------------------------------
#!/usr/bin/env bash
echo "While loop"
declare -i n=0
while (( n < 10 ))
do
	echo "n: $n"
	(( n++ ))
	sleep 1
done


#!/usr/bin/env bash
echo "Until loop"
declare -i m=0
until (( m == 10 ))
do
	echo "m: $m"
	(( m++ ))
	sleep 1
done

--------------------------------
for i in ...
do
	...
done
--------------------------------
#!/usr/bin/env bash
echo "For loop"
for i in 1 2 3                      # Direct integers can be used to range
do
        echo "$i"
done

#!/usr/bin/env bash
echo "For loop"
for i in {1..100}                    # Brace expansion can be used to define the range
do
        echo "$i"
        sleep 1
done


#!/usr/bin/env bash
echo "For loop"
for (( i=1; i<=100; i++ ))	# Arithmetic evaluation: ((...)) performs calculations and changes the value of variables
do
        echo "$i"
        sleep 1
done


#!/usr/bin/env bash
echo "For loop"
declare -a fruits=("apple" "banana" "cherry")	# Arrays: declare -a snacks=("apple" "banana" "orange")
for i in ${fruits[@]}				# Parameter expansion: ${...} retrieves and transforms stored values
do
        echo "Today's fruit is: $i"
        sleep 1
done


#!/usr/bin/env bash
echo "For loop"
for i in $(ls)	# Command Substitution: $(...) puts the output of one command inside another
do
        echo "Found a file: $i"
        sleep 1
done



# Case statement: Cheks an input againist a set of predefined values
                  Runs code when input matches the condition
------------------------------------------
case ... in
	...) statement;;
	...|...) statement;;
	*) statement	# * is like else
esac
------------------------------------------

animal="dog"
case $animal in
	cat) echo "Feline";;
	dog|puppy) echo "Canine";;
	*) echo "No match!"	# * is like else
esac



#!/usr/bin/env bash
while read f
do
        echo "I read a line and it saysn: $f"
done < ~/textfile.txt
---

#!/usr/bin/env bash
while IFS= read -r LINE
do
	echo "$LINE"
done < "$1"





## Functions: Repeatedly call a piece of code
```bash
fn(){
	....
}
```
```bash
#!/usr/bin/env bash
greet(){
        echo "Hi there"
}
echo "And now, a greeting..."
greet
```

```bash
#!/usr/bin/env bash
greet(){
        echo "Hi there, $1"  # $1 will take the first argument
}
echo "And now, a greeting..."
greet Scott  # Scott is the first argument in this example
```

## Function variables: $@ represents the list of arguments given to a function
		      $* also represents the list of arguments, but it brings them all in one item
                      $FUNCNAME represents the name of the function

```bash
#!/usr/bin/env bash
numberthing(){
        declare -i i=1
        for f in $@;		# $@ represents the list of arguments given to a function
        do
                echo "$i: $f"	# function will give a number to every argument 
                (( i += 1 ))
        done
        echo "This counting was brought to you by $FUNCNAME."
}
numberthing $(ls /)		# function will get the result of ls  and number them
echo
numberthing pine birch maple spruce	# function will get arguments one by one and number them
```

### Every variable in bash is global, to make it local put "local" keyword before it!

### Write files: echo "abc" > out.txt  # overwrites the content of out.txt
               echo "abc" >> out.txt  # appends the content at the end of the out.txt
```bash
#!/usr/bin/env bash
for i in 1 2 3 4 5
do
        echo "This is line $i" > ./out.txt  # overwrites in every iteration, only can see last line
done
```
```bash
#!/usr/bin/env bash
for i in 1 2 3 4 5
do
        echo "This is line $i" >> ./out.txt  # useful to create your own log files
done
```

Read files:  extract matching text(search for a term in log files), transform etc.
             do echo $line; done < in.txt  # loop will read in.txt line by line

#!/usr/bin/env bash
while read f
	do echo "I read a line and it says: $f"
done < ./out.txt


## Challenge:
- Compose a script that uses control structures to show random replies
  * echo $RANDOM  # gives a random number between 0 to 32767
  * echo $(( 1 + $RANDOM % 10 ))  # gives a random number between 1 to 10, make sure not 0 added 1 +
- Examples include a quote viewer, a dice roll, or a card raw
- 20min

## A randomness game
```bash
echo -e "\t\tWelcome to the"
echo -e "\t\t\033[5mGame\033[0m"	# echo -e ...  # Interprets escaped characters like \t, \n, \a, and other control characters
echo
waitingnumber=$(( 0 + $RANDOM % 3 ))	# random value bw 0 and 3
mysterynumber=$(( 1 + $RANDOM % 10 ))	# random value bw 1 and 10
declare -a fortunes=(	# declare -a option used to make NAMEs indexed arrays (if supported) beginning from 1
	"Line1"
	"Line2"
	"Line3"
	"Line4"
	"Line5"
	"Line6"
	"Line7"
	"Line8"
	"Line9"
	"Line10"
)
case $waitingnumber in
	0) sleep 1; echo "One moment please..."; sleep 1;;
	1) sleep 1; echo "Yours coming shortly..."; sleep 2;;
	2) sleep 1; echo "Preparing..."; sleep 1;;
	3) sleep 1; echo "So dark today..."; sleep 3;;
esac
echo
echo ${fortunes[mysterynumber]}  # Parameter expansion: ${...} retrieves and transforms stored values
echo
```

## Arguments: 	Allows us to pass information (user input) into a script from the CLI, represented $1, $2 etc. Type it after the script name
		$0 is the name of the script
---
#!/usr/bin/env bash
echo "The $0 script got the argument $1"

./argument1.sh Apple
The ./argument1.sh script got the argument Apple
---

#!/usr/bin/env bash
echo "The $0 script got the argument $1"

./argument1.sh "Tasty Apples!"
The ./argument1.sh script got the argument Tasty Apples!

```bash
#!/usr/bin/env bash
for i in $@	# $@ an array of inputted arguments
do
	echo $i
done

# ./argument2.sh apple banana pear
# apple
# banana
# pear
```
```bash
#!/usr/bin/env bash
for i in $@	# $@ an array of inputted arguments
do
	echo $i
done
echo "There were $# arguments."

# ./argument3.sh apple banana pear
# apple
# banana
# pear
# There were 3 arguments.
```

## Options: to access it use getopts keyword
           can take arguments
           can be used in any order

```bash
while getopts u:p: option	# creates an options string which defines which options I'm looking for; my script will have -u and -p options
do
	case $option in 
		u) user=$OPTARG;;	# access to the arguments with OPTARG
		p) pass=$OPTARG;;
	esac
done
echo "user: $user / pass: $pass"

# ./options1.sh -u Rafe -p 123	
# The order of the options are not important!
# user: Rafe / pass: 123
```

```bash
#!/usr/bin/env bash
while getopts u:p:ab option	# I want to now whether the flag is used, type without column; to enable/disable certain features
do
	case $option in 
		u) user=$OPTARG;;	
		p) pass=$OPTARG;;
		a) echo "Got the A flag";;
		b) echo "Got the B flag";;
	esac
done
echo "user: $user / pass: $pass"

./options2.sh -p Rafe -u 123 -a
Got the A flag
user: 123 / pass: Rafe
```

```bash
#!/usr/bin/env bash
while getopts :u:p:ab option	# column before the first option, I want to know other options used on CLI
do
	case $option in 
		u) user=$OPTARG;;
		p) pass=$OPTARG;;
		a) echo "Got the A flag";;
		b) echo "Got the B flag";;
		?) echo "I don't know what th $OPTARG is!";;	# To get the options not mentioned, which the script not designed to use
	esac
done
echo "user: $user / pass: $pass"

./options3.sh -p -a -z
I don't know what th z is!
user:  / pass: -a
```

## Getting input interactively: read keyword, script pauses untill input is entered, input stored in a variable

```bash
#!/usr/bin/env bash
echo "Whats you name?"
read name
echo "Whats your password?"
read -s password        # -s option: dont show password
read -p "Whats your favorite animal? " animal	# -p option: all in one line, use space after the question the prompt will begin right after it
echo
echo -e "Name: $name\nPassword: $password\nFavorite animal: $animal"
```

```bash
#!/usr/bin/env bash
echo "Which animal"
select animal in "cat" "dog" "bird" "fish"	# User selects from the list of options
do
	echo "You selected $animal!"
	break
done

./input2.sh
Which animal
1) cat
2) dog
3) bird
4) fish
#? 4
You selected fish!
```

```bash
#!/usr/bin/env bash
echo "Which animal"
select animal in "cat" "dog" "quit"	# User selects from the list of options
do
	case $animal in
		cat) echo "Cats like to sleep";;	# You can add ;break if you want to quit after this selection
		dog) echo "Dogs like to play catch";;
		quit) break;;
		*) echo "Not sure what that is";;
	esac
done

./input3.sh
Which animal
1) cat
2) dog
3) quit
#? 2
Dogs like to play catch
#? 1
Cats like to sleep
#? 5
Not sure what that is
#? 3
```

## User ignores to give input cause some problems, how to deal with that?: 

#!/usr/bin/env bash
read -ep "Favorite color? " -i "Blue" favcolor	# -i option will advice us Blue, user can accept or change it
echo $favcolor


```bash
#!/usr/bin/env bash
if (($#<3)); then
	echo "This command requires three commands: "
	echo "username, user id, and favorite number."
else
	# the program goes here
	echo "username: $1"
	echo "user id: $2"
	echo "favorite number: $3"
fi
```

```text
./input6.sh
This command requires three commands:
username, user id, and favorite number.

./input6.sh Rafe 123 111
username: Rafe
user id: 123
favorite number: 111
```

```bash
#!/usr/bin/env bash
read -p "Favorite animal? " fav
while [[ -z $fav ]]	# -z option checks if there is variable entered or not
do
	read -p "I need an answer! " fav
done
echo "$fav was selected."
```
```text
./input7.sh
Favorite animal? Dog
Dog was selected.

./input7.sh
Favorite animal?
I need an answer!
I need an answer!
```

```bash
#!/usr/bin/env bash
read -p "Favorite animal [cat]? " fav	# [cat] indicates user the default answer is the cat
while [[ -z $fav ]]	
do
	fav="cat"	# instead of asking again, assign the default value to variable
done
echo "$fav was selected."
```

```bash
#!/usr/bin/env bash
read -p "What year? [nnnn] " year
until [[ $year =~ [0-9]{4} ]]		# this checks entered number 4 digits, 0 to 9
do
	read -p "A year, please! [nnnn] " year	# until the answer is correct it will ask
done
echo "Selected year: $year"
```

## Challenge:
  - Put together what you learned.

```bash
#!/usr/bin/env bash
# A three-in-one game app
# The game definitions

guess() {
	local -i mynumber=$(( 1 + $RANDOM % 10 ))
	read -p "I'm thinking of a number bw 1 and 10. Guess what?" theguess
	if (( theguess == mynumber )); then
		echo "You got it! Congrats"; echo
	else
		echo "Nope. I was thinking $mynumber. Try it again!"; echo
	fi
	sleep 1
	choosegame
}

flip() {
	local -i mynumber=$(( 1 + $RANDOM % 2 ))
	if (( 1 == mynumber )); then
		local face="HEADS"
	else
		local face="TAILS"
	fi
	printf "I flipped a coin and it was: %s\n\n $face
	sleep 1
	choosegame	
}

dice() {
	local -i mynumber=$(( 1 + $RANDOM % 6 ))
	local -i mynumber2=$(( 1 + $RANDOM % 6 ))
	printf "I rolled two dice and the results are: %s and %s. \n\n $mynumber $mynumber2
	sleep 1
	choosegame	
}

# The game chooser
choosegame(){
	select game in "Guessing Game" "Flip a Coin" "Dice Roll" "Exit"
	do
		case $game in
			"Guessing Game") guess;;
			"Flip a Coin") flip;;
			"Dice Roll") dice;;
			"Exit") exit;;
			*) echo "Please choose from the menu";;
		esac
	done 
}

# If there is an argument provided, jump right to that game. Otherwise, show game choose menu.
case $1 in
	"guess") guess;;
	"flip") flip;;
	"dice") dice;;
	*) choosegame;;
esac
```

## Basic Calculation: bc command 
A mathematical expression containing +,-,*,^, / and parenthesis will be provided. Read in the expression, then evaluate it. Display the result rounded to  decimal places.
```bash
read num 
echo $num | bc -l | xargs printf "%.*f\n" 3
```

### Given  N integers, compute their average, rounded to three decimal places.
```bash
read n
printf "%.3f" $(echo "("$(cat)")/$n" | tr ' ' '+' | bc -l)
```








































