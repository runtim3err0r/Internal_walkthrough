# Internal_walkthrough

1. Enumeration:
As always starting with nmap scan of the target `nmap -sC -sV <ip>`. Nmap revealed only 2 ports open: http on 80 and ssh on 22.
Port 80 is a default apache webpage and searching for that version reveals no vulnerability. Further enmuration with dirbuster reveales a "/blog" page. Going there we can see there is a Login page using WordPress.
Trying default credentials like admin:admin gives a message "invalid password for user admin" which indicates there is a user admin.

2. Brute-forcing WordPress:
So in order to get the password and check for other users we run `wpscan` tool. It shows only one user and thats admin. No other usefull info is there so going straight for the bruteforce. Command to use `wpscan --url <ip> -P /usr/share/wordlists/rockyou.txt -U admin`. We get the result 'my2boys' as a password. Navigating over to wordpress page we login as admin user.

3. Getting initall shell:
Checking the 'Posts' on the website we find there is a user "Will" and his password. Thats a rabbit hole though, because we already know there is only admin user. Logic here is to find a .php file we can edit and insert a php-reverse-shell.
We find a file '404.php' in Appearance > Theme Editor and we just insert the php code for reverse shell into this file. We start the listener `nc -lvnp 4444` and execute the file on "http://internal.thm/blog/wp-content/themes/twentyseventeen/404.php".
We should then get our first shell on port 4444.

4. Further enumeration:
Now that we have a shell we search for some .txt files and right there we find there is a user aubreanna with the password. Those will be a SSH credentials. Logging with this user on SSH we find a user.txt which is a 1st flag and a jenkins.txt.
We find there is jenkins service running on 172.17.0.1 so the logic here is to tunnel over to that IP. We exit current session and do `ssh -L 8080:172.17.0.1:8080 aubreanna@<ip>`. This will give us access to jenkins service and we can just navigate
over to the localhost in a browser. So `http://127.0.0.1:8080` opens up a jenkins login page. We try 'admin:admin' but nothing. Its time to bruteforce again.

5. Using hydra to bruteforce jenkins:
We first need to catch a login request in burp suite so we can build a command for hydra. The command should look something like this:
`hydra -l admin -P /usr/share/wordlists/rockyou.txt 127.0.0.1 -s 8080 -V http-form-post '/j_acegi_security_check:j_username=^USER^&j_password=^PASS^&from=%2F&Submit=Sign+in:Invalid username or password'`.
After couple of seconds we get a password 'spongebob'.

7. Using jenkins console to get a shell:
Since we are logged as admin, the logic is to find a console where we can execute commands. Going over to 'Manage Jenkins', we find one. It accepts Groovy scripts so "https://gist.github.com/frohoff/fed1ffaab9b9beeb1c76" has everything I need.
Copying this command but instead `String cmd="cmd.exe";` I will put `String cmd="/bin/sh";` because its a Linux box. Also, change the IP address to my machine. Starting the listener in another terminal and executing this command we get a shell.

8. Getting a root user:
So once the terminal opens, doing `ls` reveals a .txt file with a note. Its a 'root' user and his password. So, we  start another ssh with these credentials and we are in as root. A flag is there already.




   
