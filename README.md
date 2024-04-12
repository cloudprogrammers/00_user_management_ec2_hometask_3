# Linux Users and Groups Management
Welcome to our tutorial series where you'll learn about managing users and groups on an AWS EC2 instance. This series is designed to give you practical experience with both Linux and AWS, focusing on the essential tasks of user management in a cloud computing environment. By the end of these tutorials, you'll know how to create and manage user accounts on a Linux system running on AWS, and you'll be more comfortable working with cloud services.

## _What you will learn_

- **Superuser Privileges on EC2**  You will understand the critical role of running scripts with superuser privileges, especially for tasks that change the system, and how to check for these privileges in your scripts.
- **Handling Command Line Arguments on EC2** Learn how to parse command line arguments to make your scripts more flexible. This includes using the first argument as a username and the rest as a comment for the user account.
- **Secure Access and File Transfer via SSH** We'll cover how to set up SSH access for newly created users. This is important for securely transferring files between your local computer and the EC2 instance using SCP.
- **Interacting with EC2 Instance Metadata** You'll learn how to access and use EC2 instance metadata within your scripts. This is useful for scripts that need information about the instance they're running on.

## Prerequisites
Before starting, make sure you have:
- An AWS account with access to create and manage EC2 instances.
- At least one EC2 instance available and running in your AWS account. <link to script>
- A basic understanding of terminal commands and access to SSH and SCP tools on your local machine.

# EXERCISES

## Exercise 1: Introduction to Linux Users and Groups

**Objective**
Learn how to manage users and groups on an AWS EC2 Linux instance. This exercise will guide you through creating a user and a group on your EC2 instance, setting permissions, and testing access control.

**Task 1: Log Into AWS Console and SSH into Your EC2 Instance**

Log into your AWS Management Console and navigate to the EC2 Dashboard. Find your running EC2 instance.
**Get Instance Details** Click on your instance to view its details. Locate and click the `Connect` button in the top right hand corner of the console window. Then, click the `SSH client` option and copy the connection command that is at the bottom. It should look like this:

```
ssh -i "id_rsa" ubuntu@ec2-51-21-77-247.eu-north-1.compute.amazonaws.com
# or
ssh -i "id_rsa" ubuntu@<public ipv4> 
```

**Execute the SSH Command** Follow the `Connect to instance` instruction and check if you established a successful connection to your EC2 instance via SSH.

**Task 2: Creating a New User and Group:**

**1. Create New group:** 
Start by creating a new group named **heroes** on your EC2 instance. Use the command:
```sh
sudo groupadd heroes
```

**2. Create a New User:** 
Now, create a new user named **hero** and add them to the **heroes** group. Use the **-m** option to create a home directory for the user, and **-G** to specify the group:

```sh
sudo useradd -m -G heroes hero
```

**Set a Password for the New User:** Assign a password to the hero user by running:
```
sudo passwd hero
```

**What we did so far ?**

**Created a Group:** You created a new group named `heroes`. Groups in Linux are used to organize users with similar access rights or responsibilities. By managing groups, you can easily assign permissions to multiple users, enhancing both security and convenience.

**Added a New User:** You introduced a new user named `hero` to the system and assigned them to the `heroes` group. Creating users is fundamental in Linux for assigning specific access rights, running processes, and maintaining security. The `-m` option ensured that hero got a home directory, a space where user-specific files and configurations reside.

**Set a Password:** You secured the hero user account with a password, a crucial step for authentication and access control.

The purpose of these actions is twofold. Firstly, it helps you understand how Linux handles access control and user management—a core aspect of system administration. Secondly, performing these tasks on an AWS EC2 instance introduces you to managing cloud-based Linux environments

**Task 3: Working with Directories and Permissions**

**1. Create a Directory and File:** 
First, navigate to the **/tmp** directory. We will be making our experiment here. It's a catalog for all temporary stuff.
Create a new directory named `hero_files`.
Inside `hero_files`, create a text file named message.txt and write "Hello there!" into it.
All the commands are here:
```
cd /tmp
mkdir hero_files
echo "Hello there!" > hero_files/message.txt
```

**Set Group Ownership and Permissions:**
Change the ownership and permissions so only members of the `heroes` group can read and write to the directory.

```
sudo chown :heroes hero_files/
```
Set permissions to allow read and write access to the file only for users in the `heroes` group.
```
sudo chmod 770 hero_files/
sudo chmod 660 hero_files/message.txt
```

**Summary**
Now, you learned to manage file permissions and group ownership within your EC2 Linux environment, essential for controlling access to system resources.

You created a `hero_files/` directory and set its permissions and ownership to ensure only members of the `heroes` group could read and write to it.

**Task 4: Test the Access Control**

Attempt to read and write to `message.txt` as a user not in the heroes group. You should be denied access in both cases:
```
cat /tmp/hero_files/message.txt # READ operation
echo "New message" > /tmp/hero_files/message.txt # WRITE operation
```

**How do we solve this ?** 
Well, we have 2 options. Only users who belong to `heroes` group can read, write and execute the files inside `/tmp/hero_files` directory.
First, we can switch now to the user which belongs to the `heroes` group:
```
su - hero
```
And try to access files. Do it now, and see what effect does it have on the command output. 

The **second** option is to add the current user to `heroes` group. By doing this, the user we are logged as in, will gain all the privileges that the `heroes` group has. 
Use the `whoami` command to display the current username.
```
whoami
# or
id
```

Adding the current user to the heroes group will grant them the permissions associated with that group, specifically the ability to read and write to the message.txt file in the `/tmp/hero_files/` directory.

**Add User to the Group:**
Use the usermod command to add your current user to the heroes group. You'll need to replace <your-username> with the username you identified in Step 1.
```
sudo usermod -aG heroes <your-username>

```
Note: The `-aG` option appends the user to the specified group without removing them from their existing groups.
You can combine above command with the previous one, using `command substitution`

```
sudo usermod -aG heroes $(whoami)
```

**Verify and Test Access Again** :
After adding your user to the heroes group, you need to log out and log back in for the group changes to take effect. This is necessary because the current session does not automatically refresh group memberships.
**Log Out and Log In:**
Depending on your environment, logging out and back in can be done through the GUI or by closing and reopening your terminal session.

**Verify Group Membership:**
```
id
```

**Test File Access again**
Attempt to read and write to message.txt again. Now that your user is part of the heroes group, you should have the necessary permissions

```
cat /tmp/hero_files/message.txt
echo "New message" >> /tmp/hero_files/message.txt
cat /tmp/hero_files/message.txt
```
To go further, see [the study material](https://docs.google.com/document/d/14oAojwZD8ULcECaUFB4dqbmuZb2K-c05tKXaKY1bZwk/edit?usp=sharing) for information why write and format in bash this way.

**Summary**
You tested the effectiveness of Linux file permissions and group ownership by attempting to access the hero_files directory before and after adjusting your group membership.

This exercise demonstrated the immediate impact of permissions and group ownership on file access, illustrating a practical aspect of system security.
By switching users and modifying group memberships, you experienced firsthand how Linux enforces access controls based on user and group associations.

**Task 5: Setting Up SSH Access for the New User**

**Objective**
Learn how to configure SSH access for the newly created hero user on your AWS EC2 instance. 

**1 Generating SSH Keys (on your local machine):**
First, you need to generate a new SSH key pair to use for hero. Open a terminal and run:

```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/hero_key
```

This command creates a new SSH key pair stored in the `~/.ssh/hero_key` (private key) and `~/.ssh/hero_key.pub` (public key) files.

**2. Uploading the Public Key to Your EC2 Instance**
To allow hero to log in via SSH, you must add the public key to the ~/.ssh/authorized_keys file in the hero user's home directory on your EC2 instance.

1. SSH into your EC2 instance using your regular method (e.g., as ec2-user).
2. Switch to the hero user:

```
sudo su - hero
```

3. Check if the `~/.ssh` directory exists:
```
cd ~/.ssh
```
4. If it does not exist, create it:
```
mkdir ~/.shh
```
5. Append the public key to the `~/.ssh/authorized_keys` file. Replace your_public_key with the content of your hero_key.pub file. First, copy the contents of your public key (from your local machine):
```
cat ~/.ssh/id_rsa.pub
```
the name of the key may be different, but it will be `.pub` file for sure.
6. Put the content of your public key into `authorized_keys` file on your EC2
```
echo your_public_key >> ~/.ssh/authorized_keys
```

## Exercise 2: Building a basic Bash script 
**Objective**
Develop a Bash script that interacts with AWS EC2 instance metadata and executes commands based on user input. This exercise will help you understand how to script interactions with a Linux environment and retrieve metadata from an EC2 instance, enhancing your scripting skills for cloud operations.

**Tasks**

**1. Logging Into Your EC2 Instance:**
Start by SSH into your EC2 instance using the hero user and the SSH key you set up in the previous task:
```
ssh -i ~/.ssh/hero_key hero@Your-Instance-IP
```
Ensure you replace Your-Instance-IP with the actual public IP of your EC2 instance.

**Task:** Understand why it's crucial to run certain scripts with superuser privileges, especially when modifying system settings or managing user accounts. 
Before going further, see [the study material](https://docs.google.com/document/d/1gMYi7TxPrvmNcBsFKtygX6Sy0uiidkMfSDDO8XnX0kk/edit?usp=sharing).

**2 2. Creating the Script:**
Open vim to start writing your script:

```
vim ec2_manager.sh
```

**3. Script Setup:**
Start your script with the shebang line and set it to execute using Bash:
```
#!/bin/bash
```
Shebang (#!): Is the first line in your script, telling the system which interpreter to use to execute the script. For shell scripts, this is typically #!/bin/bash or #!/usr/bin/env bash Here you have everything explained about it:
[Shebang Explained.](https://docs.google.com/document/d/1xOhWo0Wz0ur-8l34e8K4DtKxyQ8dyvVNinGC7kgSF5U/)

**2. Make your script executable:**

```
chmod +x ec2_manager.sh.
```

**3. Implementing the Menu:** 
Use the `echo` command to print each option to the terminal. The `echo` command in shell scripting is used to display lines of text or string variables. It's one of the most basic and frequently used commands in shell scripting. After displaying each option, you'll want to capture the user's choice. This is done using the `read` command, which reads a single line from standard input (i.e., the user's keyboard input) and assigns it to a variable. In this case, you could use read -p `"Enter choice: "` choice to prompt the user directly and store their input in a variable named `choice`

```
echo "Select an operation:"
echo "1) Launch htop on instance"
echo "2) Launch tcpdump utility on instance"
echo "3) Show EC2 instance metadata"
read -p "Enter choice: " choice
```

**4. Run the script**
Verify it prompts the user for input. Fire it up several times. Test out each option. For now, it does not do anything.

**5. Processing User Input:**
Now, we will be adding the required features for each of the options that user may choose. The `if` conditional lets us specify the condition under which the code will run. We will test it out, by covering the case where the user types in 1, choosing from the menu above.

```
if [ "$choice" == "1" ]; then
```
To go further, see the [study material](https://docs.google.com/document/d/14oAojwZD8ULcECaUFB4dqbmuZb2K-c05tKXaKY1bZwk/) for information why write and format in bash this way. You will gain deeper understanding on how to deal with variables in Bash.

After this, add a test `echo` command, to see whether the logic works:

```
if [ "$choice" == "1" ]; then
    echo "User chosen option $choice"
fi
```
run the script typing in `./ec2_manager.sh`, select `1` as an input and see if it works and what value it returns.

**6. Replace the logic inside the first if statement**
Now, we want to replace the logic inside the `if` statement with actual feature we want to use. 
```
if [ "$choice" == "1" ]; then
    echo "Launching htop..."
    htop
fi
```

**7. Implement remaining features**
Now, we need to expand our `if` conditional to cover the remaining cases. We do it using `elif` and `else` statements.

```
if [ "$choice" == "1" ]; then
    echo "Launching htop..."
    htop
elif [ "$choice" == "2" ]; then
    echo "Launching tcpdump..."
    tcpdump
elif [ "$choice" == "3" ]; then
    echo "Retrieving instance metadata.."
    # Metadata token retrieval
    TOKEN=$(curl -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
    # Fetching metadata
    METADATA=$(curl -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/)
    echo "$METADATA"
fi
```

**8. Saving and Testing the Script:**
Save your script in `vim` and test it out. Start it several times, trying each of the options the scipt offers.

**Summary**
In this exercise, you created a Bash script that leverages the EC2 instance metadata API and responds dynamically to user input, executing different system commands.

# Key Points to Remember

**1. Script Permissions and the Use of Shebang (#!)**
Scripts need to be executable to run. Using `chmod +x scriptname.sh` changes the script's permissions, making it executable. The shebang `(#!)` at the beginning of scripts specifies the interpreter, ensuring scripts run under the correct environment.

** Capturing and Processing User Input:**
User input handling, through read commands or script parameters, enables interactive scripts and dynamic execution paths based on user decisions. It enhances the script's flexibility and user-friendliness.


# Commands Reference

| Command | Description |
| ------ | ------ |
| `useradd` | Creates a new user account on the system. |
| `passwd` | Sets or changes the password for a user account |
| `echo` | Displays a line of text; used here for providing feedback or setting passwords. |
| `if, [[ ]]` | Conditional statements; essential for error checking and flow control. |
| `shift` | Shifts positional parameters; useful for processing command-line arguments beyond the first one. |
| `$(...)` | Command substitution; allows the capture of command output and store them in variables. |
| `/dev/urandom` | Provides random bytes; useful for generating secure passwords. |
| `tr, fold, head` | Text processing commands; used in combination to filter and select password characters. |