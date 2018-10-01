---

title: Lab &ndash; Password Cracking
number: 9
---

{% include vm-setup-boilerplate.html vm='[My prepared Kali VM]( /security-assignments/virtual-machines/#kali )'%}

# Part 0: Introduction to Linux Tutorial

1.  Complete the ["Intro to Linux" tutorial]( /security-assignments/intro-to-linux ). There is a deliverable associated with this tutorial which counts towards your participation grade.



# Part 1: Testing Passwords

1.	Visit the following URL:

    [https://lowe.github.io/tryzxcvbn/](https://lowe.github.io/tryzxcvbn/)

2.	Try out different passwords to see how strong they are.

    **Optional:** If you want to learn more about password strength estimation, see [this video and paper](https://www.usenix.org/conference/usenixsecurity16/technical-sessions/presentation/wheeler)

    
    
# Part 2: Online Password Attack

1.	Open a terminal window in your Kali VM, and type `cd /usr/share/wordlists`.
2.	Unzip the `rockyou.txt.gz` archive as follows:

        gunzip rockyou.txt.gz 

3.	`ls` command grants a view to see which files are in this folder. Type `wc -l rockyou.txt` to see how many entries are in this password dictionary file (‘-l’ is a lower-case ‘L’). 

    This file comprises all unique passwords from the 32 million RockYou password breach you read about in your reading.

    
    
4.	Using this password list, launch an online password attack using `THC-Hydra`. 

    Visit [https://is.theorizeit.org](https://is.theorizeit.org) in a browser. Let's pretend that you forgot the password. Let's "recover" the password.
    
    Type the following (all on one line):

        hydra -V -l istheory -P rockyou.txt https-get://is.theorizeit.org/auth/

    **Note:** The trailing slash (‘/’) is needed.

    Where:
    
    * `hydra` is the password cracking tool to execute
    * `-V` means verbose, and will show you the username and password combination being attempted
    * `-l istheory` sets “istheory” as the login name. Note: that’s a lowercase ‘L.’
    * `-P rockyou.txt` is the password dictionary file to use. 
    * `https-get` means a GET request over HTTPS. Note that Hydra supports many protocols (e.g., ftp, ssh).
    * `is.theorizeit.org/auth/` is the password-protected URL to be accessed.

    **Note:** you can also use THC-Hydra with web forms: [http://insidetrust.blogspot.com/2011/08/using-hydra-to-dictionary-attack-web.html](http://insidetrust.blogspot.com/2011/08/using-hydra-to-dictionary-attack-web.html)
    
    In Greek and Roman mythology, [Hyrda](https://en.wikipedia.org/wiki/Lernaean_Hydra) is a mythical sea monster with many heads. When a head is cut off, it is replaced by another. `THC Hydra`, the tool you are using,
    likewise launches multiple "heads" (tasks), each of which makes a battery of attacks (password guesses) before dying off and being replaced by another head (task). By default, Hyrda runs with 16 concurrent heads.
    
    The Hyrda output will tell you at what time it started, how many passwords it has tried so far, and at what time it stopped.
    
    {% include lab_question.html question='What was the password (Scan the results to find the line beginning with <code>[443][http-get]</code>)?' %}
    
    **Question:** Approximately how many passwords a second were you able to try? **Hint:** You may need to calculate this from the start and end time along with number of guesses made.
    
    {% include lab_question.html question='For the lab submission, you will be asked to look at sample output from a `hydra` run, and determine how many passwords were tried per second in the sample output.' %}
    
    
    
    
    
# Part 3: Offline Attack Using Hashcat

Cracking in Hashcat:

<div class='alert alert-info'>If you're feeling adventurous, you can install hashcat on your host computer, where you'll get massive speed improvements. Hashcast needs to be able to directly interface with
the CPU/GPU in order to fly -- it can't so as well in a virtualized environment.</div>

1.	On your Kali machine, open up a terminal and navigate to your home directory (`/root/`).
2.	Hashcat uses a command-line interface. To see available options and syntax, type: 

		hashcat --help
	
4.	First, a dictionary attack will be performed against a password-protected Word document. The following command uses a Python script to obtain the hash of the Word document password:

		python office2john.py hashcat.doc
		
	Note: 
	* office2john.py can be obtained [here](https://raw.githubusercontent.com/magnumripper/JohnTheRipper/bleeding-jumbo/run/office2john.py), but I already put it in your home dir.
	* hashcat.doc is available [here](https://raw.githubusercontent.com/deargle/deargle.github.io/master/security-assignments/hashcat.doc). To obtain it, run 
		
			wget https://raw.githubusercontent.com/deargle/deargle.github.io/master/security-assignments/hashcat.doc
	

5.	In the output you’ll see the name of the file followed by the type. The type is shown with a $ at the beginning and end of it. You’ll need to copy the type and everything until “:::”. For example, the hash looks like the following, all on one line:

		$oldoffice$1*04477077758555626246182730342136*b1b72ff351e41a7c68f6b45c4e938bd6*0d95331895e99f73ef8b6fbc4a78ac1a 

	Save the hash into a file in your home dir. 

	Note: make sure the entire hash is on one line within the text file. **Don't add extra spaces at the end.** 
	If you get a "line-length exception" in the next step, make sure there's not a typo in the beginning of the hash.

6.	While still in your home dir, run the following command (all on one line). Reference the hash file you just created, and choose an arbitrary name for an output file. Once the password is cracked,
    you will read your output file to see the cracked password. It will be appended to the end of the hash following a `:` symbol.

		hashcat --force -a 0 -m 9700 --potfile-disable -o <<outputFileName.txt>> <<HashInputFileName.txt>> /usr/share/wordlists/rockyou.txt

	Or alternatively, if you prefer to do it without making an input file, put the hash string right in the terminal, surrounded by single quotes:  

		hashcat --force -a 0 -m 9700 --potfile-disable -o <<outputFileName.txt>> '[hash string]' /usr/share/wordlists/rockyou.txt
	
	Where the switches correspond to:

	{: .table .table-condensed } 
	| \-\-force | This is necssary to get hashcat to run in a VM environment (it doesn't normally like to). | 
	| -a 0 | Straight dictionary attack against the hash |
	| -m <<Office_Flag>> | The corresponding flag for the version of Office in use (see table in `hashcat --help`) |
	| \-\-status | Provides an update of the status of the process without giving a prompt |
	| -o <<Output_File>> | The location where the cracked hashes will be saved. <br>The results will also be saved on the .pot file, unless otherwise specified. In our case, we disabled it. |
	| <<Hash>> | The saved password hash. |
	| <<Dictionary>> | The list of words that will be used to try and crack the password. |

	<br/>

    {% include lab_question.html question='What is the password for <code>hashcat.doc</code>?' %}
    
    
    Do the same for the file `john.doc` (available [here](https://raw.githubusercontent.com/deargle/deargle.github.io/master/security-assignments/john.doc), use `wget` as above to obtain it from url `https://raw.githubusercontent.com/deargle/deargle.github.io/master/security-assignments/john.doc`). 

    {% include lab_question.html question='What is the password for <code>john.doc</code>?' %}
    
    
    Examine [the hashcat cracking benchmarks](https://gist.github.com/epixoip/a83d38f412b4737e99bbef804a270c40) for a [Brutalis](https://sagitta.pw/hardware/gpu-compute-nodes/brutalis/). See [here](https://hashcat.net/forum/thread-4908.html) and [here](https://hashcat.net/forum/thread-2906.html) for help interpreting hashcat cracking speed notation.
    
    
    {% include lab_question.html question='Roughly how many passwords per second can Hashcat running on a Brutalis try on a <code>.doc</code> file?' %}
        
    {% include lab_question.html question='How much faster is Hashcat in cracking .doc MS Office documents (option 9700, "&lt;= 2003 $0|$1, MD5 + RC4") compared to Office 2013 documents (option 9600)?' %}
   
        
    **Optional:** Install `hashcat` on your own machine (not the VM). See how your benchmarks compare against a Brutalis. Note that running benchmarks on the VM will break once it reaches `scrypt` before complete results are reported.

		hashcat -b --force
        
   
    {% include lab_question.html question='How does an offline password attack compare with the online hydra attack you attempted earlier?' %}


    
    
# Part 4. Cracking Linkedin Hashes Using Hashcat

In this section, you’ll see how many hashes you can recover from the 2016 LinkedIn password breach. This breach of 177,500,189 unsalted SHA1 password hashes represents the data of all 
LinkedIn users as of 2012. Among these passwords, only 61,829,207 are unique.

However, in interest of your time, this section will require you to crack only 500,000 of these passwords. After you complete this lab, you’re welcome to crack all of the LinkedIn 
hashes. Ask me for a copy.

1.	Navigate to your home directory, where you will find a copy of the file `LinkedIn_HalfMillionHashes.txt` (also available [here](https://raw.githubusercontent.com/deargle/deargle.github.io/master/class/cu/mgmt4250/LinkedIn_HalfMillionHashes.txt)).

2.	Open a terminal. To get your feet wet, perform a "straight" dictionary attack using the `rockyou.txt` wordlist again, as follows (one line):

		hashcat --force -m 100 --potfile-disable --remove --outfile=LinkedIn_cracked.txt LinkedIn_HalfMillionHashes.txt /usr/share/wordlists/rockyou.txt

	Note: This command may take 5–10 minutes to run. To see the status of a running job in Hashcat, press the “s” key (it might take up to 15 seconds for Hashcat to report its status).
    
    Hashcat will report how many passwords it "recovered" when it finishes.
    
    <div class='alert alert-warning'>These commands use the <code>--remove</code> flag. This will remove cracked hashes from the input file. So, if you run these commands more than once without changing anything, it won't crack anything after the first time.</div>
    

    You can always count the number of lines in your outfile (`LinkedIn_cracked.txt`) to see how many you've cracked so far, total:
           
        wc -l LinkedIn_cracked.txt
    
    Or count the number of passwords left (it started with half a million):
        
        wc -l LinkedIn_HalfMillionHashes.txt

    To see hashes cracked in real time, in another terminal shell, type the command: `tail -f LinkedIn_cracked.txt`. Type `control+c` to exit the `tail` command.


    {% include lab_question.html question='How many passwords were you able to recover using this command?' %}


    
    
6.	Run another attack that uses a rules-based method (one line):

		hashcat --force -m 100 --potfile-disable --remove --outfile=LinkedIn_cracked.txt LinkedIn_HalfMillionHashes.txt -r /usr/share/hashcat/rules/best64.rule /usr/share/wordlists/rockyou.txt

	Rules apply common patterns to password dictionaries to crack even more hashes. You can read about rules in Hashcat here: [https://hashcat.net/wiki/doku.php?id=rule_based_attack](https://hashcat.net/wiki/doku.php?id=rule_based_attack). 

	The “best64.rule” is one of the most effective sets of Hashcat rules. It is continually refined using input and testing from the password cracking community. You can view the contents of the best64.rule here:

	[https://github.com/hashcat/hashcat/blob/master/rules/best64.rule](https://hashcat.net/wiki/doku.php?id=rule_based_attack])

	You can read an explanation of these set of rules here:
	
	[http://kaoticcreations.blogspot.com/2011/09/explanation-of-hashcat-rules.html](http://kaoticcreations.blogspot.com/2011/09/explanation-of-hashcat-rules.html)

    {% include lab_question.html question='How many total passwords were you able to recover after using this rules based attack in combination with the earlier straight attack?' %}
    
	
	

    
    
	
7. Run another attack that uses a hybrid method that uses a dictionary attack combined with a “mask,” which is a pattern that is appended to each password in the password dictionary:

		hashcat --force -m 100 --potfile-disable --remove --outfile=LinkedIn_cracked.txt LinkedIn_HalfMillionHashes.txt -i -a 6 /usr/share/wordlists/rockyou.txt ?d?d

	The `?d?d` at the end means to append two digits between 0–9 each at the end of each password in the rockyou.txt password dictionary.

    {% include lab_question.html question='How many total passwords were you able to recover after using this hybrid attack combination with the earlier straight and rules-based attacks?' %}
    

    
    
8. If you would like to try using a different character set for your mask, you can use the following masks below. Note that each mask below is for one character. If you wanted to test four digits at the end of each password, the mask would be: ?d?d?d?d.

		?l = abcdefghijklmnopqrstuvwxyz
		?u = ABCDEFGHIJKLMNOPQRSTUVWXYZ
		?d = 0123456789
		?s =  !"#$%&amp;'()*+,-./:;&lt;=&gt;?@[\]^_`{|}~
		?a = ?l?u?d?s
		?b = 0x00 - 0xff

    **Optional:** Experiment with other rules found in `/usr/share/hashcast/rules`.
        
	**Optional:** Another common password pattern is to prepend digits at the beginning of passwords. If you would like try this mask, run the following command:

		hashcat --force -m 100 --potfile-disable --remove --outfile=LinkedIn_cracked.txt LinkedIn_HalfMillionHashes.txt -i -a 7 ?d?d /usr/share/wordlists/rockyou.txt


		
<div class='alert alert-info'>Want even more practice? You can download the massive Troy Hunt haveibeenpwned SHA1 password hash list 
on the bottom of <a href='https://haveibeenpwned.com/Passwords'>this page</a>.</div>

<div class='alert alert-info'>See <a href='https://github.com/danielmiessler/SecLists/tree/master/Passwords'>Daniel Miessler's wordlist collection</a> for more wordlists besides rockyou to try.</div>
    
    
    
    
# Part 5. Secure Password Hashing

Refer again to the [benchmark output for a Brutalis](https://gist.github.com/epixoip/a83d38f412b4737e99bbef804a270c40).

{% include lab_question.html question='How much slower is Hashcat in cracking bcrypt hashes compared to SHA1 hashes?' %}

Read about the bcrypt algorithm [here](https://en.wikipedia.org/wiki/Bcrypt#Algorithm), and also [here](https://stackoverflow.com/questions/6832445/how-can-bcrypt-have-built-in-salts)

{% include lab_question.html question='Imagine that bcrypt is set to a work factor of 12. How many hashing rounds will Bcrypt go through to compute the final hash?' %}

{% include lab_question.html question='An attacker knows that a user generated their password using 8 random lowercase letters exclusively (so character space of 26, length of 8). On average, an attacker needs to try only half of all possible passwords in order to brute force the password. The attacker has access to a Brutalis. How long would it take to crack the password hash if SHA1 had been used? bcrypt with the benchmarks shown for brutalis?' %}




# Learn more:

[http://arstechnica.com/security/2013/03/how-i-became-a-password-cracker/](http://arstechnica.com/security/2013/03/how-i-became-a-password-cracker/)

Official Hashcat documentation: [http://hashcat.net/wiki/](http://hashcat.net/wiki/)