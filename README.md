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

## Exercise 2: Building a User Management Script - Part 1
**Objective**
This exercise is designed to guide you through the initial steps of creating a Bash script for user management on a Linux system. You'll start with a script structure that checks for superuser privileges and reads user input. Your task is to complete the script by adding a user based on the provided input, managing the password, and ensuring the user changes the password at the first login.

**1. Handle User Input** :
Here's the initial structure of your script, add-local-user.sh. This part of the script ensures it's run with superuser privileges and collects necessary information from the user:
**Task:** Understand why it's crucial to run certain scripts with superuser privileges, especially when modifying system settings or managing user accounts. 
Before going further, see [the study material](https://docs.google.com/document/d/1gMYi7TxPrvmNcBsFKtygX6Sy0uiidkMfSDDO8XnX0kk/edit?usp=sharing).

```
#!/bin/bash

# Ensure the script is run with superuser privileges
if [[ "${UID}" -ne 0 ]]; then
    echo 'Please run the script using sudo or as a root.'
    exit 1 # Exit with an error
fi
```

**2: Reading User Input**
**Task 1:** You will be creating an account with data collected from user.
Here is a quick refresher on how to parse user input and save it to variable.

```
# Prompt for user input
read -p 'Enter the username to create: ' USER_NAME
read -p 'Enter the name of the person or application that will be using this account: ' COMMENT
read -p 'Enter the password to use for the account: ' PASSWORD
```

**2: Create user and password**

**Task 1:** 
Add the User: Use the collected input (`USER_NAME` and `COMMENT`) to add a new user to the system. Remember to create a home directory for the user.
```
useradd -c "${COMMENT}" -m ${USER_NAME}
```

**Task 2:** Check if `useradd` Succeeded. 
This is your part. 
You need to verify in the script, whether the user creation process succeeded.
Here is the
[material for reference.](https://docs.google.com/document/d/1rjk55kJ_JVwFoNCq018LBULVIjBkWHYj3N5Z0bk4a4o/edit?usp=sharing)

**Task 3:** Set the User Password
Securely add the password for the newly created account. Be cautious of how you handle the password in your script.

```
echo ${PASSWORD} | passwd --stdin ${USER_NAME}
```

**Task 4:**
This is your part. 
Your goal is to complete and test out the working script.
>Requirements:
**1.** Ensure the Password Add Was Successful
Similar to step 2, check the success of the password assignment and notify if there's an error.
**1.** Enforce Password Change on First Login
Use the `passwd` command to force the user to change their password the next time they log in.
**1.** Display Account Details
Similar to step 2, check the success of the password assignment and notify if there's an error.

**Testing:** Test your script with different inputs to ensure it behaves as expected under various scenarios.


## Exercise 3: Building a User Management Script - Part 2
**Objective**
Refine your user management script by adjusting it to accept the command-line arguments instead of relying on capturing user input. Incorporate a usage statement that guides the user on how to properly execute the script, especially when required arguments, like the account name, are missing. This exercise teaches you to improve script usability and provide clearer communication in your tools, an essential skill for script development and user support.

**Tasks**

**Task 1:** Check for the Presence of an Account Name
Modify the Script to Check Arguments:
Immediately after ensuring the script is run with superuser privileges, add a check to see if an account name was provided as an argument.
If not, display a usage statement and exit with status 1.

```
if [[ "${#}" -lt 1]]; then
    echo "Usage: ${0} USER_NAME [COMMENT]..."
    echo "Create an account on the local system with the name of USER_NAME and a name of person or application that will use the script with COMMENT."
    echo "Example: ${0} test_user test_user@company.com"
    exit 1
fi
```

See, that we use one of the [special variables.](https://docs.google.com/document/d/1rjk55kJ_JVwFoNCq018LBULVIjBkWHYj3N5Z0bk4a4o/edit?usp=sharing) in Linux to reach our goal.
We also use `-lt` which means literally `less than`.

**Task 2**: Clear Usage Statement 
Modify the message

>"Create an account.." 

to better fit your particular taste. You can provide clearer examples.

**Task 3** Test your script:
* Attempt to run your script without supplying an account name to ensure that your usage statement is displayed as expected.
* Execute the script with an account name to verify that it proceeds as when the necessary information is provided.


**Task 4** Dynamically assign variables:
* Assign the first argument as the username
```
USER_NAME="${1}"
```

Use all remaining arguments as the comment for the account
The `shift` command shifts the positional parameters to the left, making $2 become $1, $3 become $2, etc.
Here, it's used to remove the first argument (username) from the list, leaving only the comment.
 ```
shift
COMMENT="${*}"
```
* Now proceed to create the user with the parsed username and comment
```
useradd -c "${COMMENT}" -m ${USER_NAME}
```

## Exercise 4: Building a User Management Script - Part 3 - generate strong, random passwords

This set of exercises builds upon your existing script by utilizing command line arguments for user and comment input and make the password generation automatic for the new account. These features improve user experience, ensure security of the passwords and increase automation level.

**Objective**
Implement a feature to automatically generate a password for the new account.

**Tasks:**
* Implement a function or a one-liner within your script to generate a random password.
* Ensure the password meets common security criteria (length, complexity).

# Key Points to Remember

**Difference between system and regular users** System users are created to run specific services, while regular users are created for human users or to run specific applications.

**Group management:** Groups are used to organize users that share common access and permission requirements.
Essential options when creating user accounts: The useradd command includes several options that are crucial for customizing the creation of user accounts.

**Superuser Privileges** Essential for scripts that alter system configurations or manage user accounts to ensure they execute with the necessary permissions.

**Handling User Input** Mastery in processing command-line arguments for setting usernames and comments enhances script versatility and user experience.

**Error Handling** Implementing checks for command execution status (${?}) and providing informative feedback are crucial for robust scripts.

**Usage Statements** Clear, informative usage statements guide users on proper script execution, improving usability and reducing errors.

**Command Substitution and Special Variables** Utilizing Bash features like command substitution ($(...)) and special variables (**$1**, **$@**, etc.) allows for dynamic and flexible script operations.

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
