# User Management in Linux

## 1. Introduction

Linux is a **multi-user operating system**. This means that more than one person or service can use the same Linux system, sometimes at the same time.

User management helps an administrator:

- Give each person a separate account.
- Control which files and commands each user can access.
- Protect private information.
- Prevent unauthorized administrative access.
- Identify who owns a file or started a process.
- Remove access when a person no longer needs the system.

> **Important:** Most user-management commands require administrator privileges. Log in as `root`, or place `sudo` before the command. For example: `sudo useradd -m john`.

---

## 2. Important User and Group Files

Linux stores local account information in several text files.

### `/etc/passwd`

This file stores basic information about local user accounts.

A typical entry looks like this:

```text
john:x:1001:1001:John Smith:/home/john:/bin/bash
```

The fields, separated by colons, are:

1. `john` - Username used to log in.
2. `x` - Shows that the password hash is stored securely in `/etc/shadow`.
3. `1001` - User ID, also called the UID.
4. `1001` - Primary group ID, also called the GID.
5. `John Smith` - Comment or account description field.
6. `/home/john` - User's home directory.
7. `/bin/bash` - User's login shell.

**Why it is needed:** Linux uses this file to connect a username with a UID, home directory, primary group, and login shell.

> `/etc/passwd` must be readable by normal programs, so password hashes are not stored there.

### `/etc/shadow`

This file stores protected password information, including:

- Password hashes.
- The date of the last password change.
- Minimum and maximum password ages.
- Password-expiration warning periods.
- Account-expiration information.

**Why it is needed:** Password information is sensitive. Normally, only `root` and specially authorized system processes can read this file.

> Linux normally stores a one-way password **hash**, not a readable or decryptable copy of the password.

### `/etc/group`

This file stores local group information.

A typical entry looks like this:

```text
developers:x:1005:john,maya
```

The fields are:

1. `developers` - Group name.
2. `x` - Indicates that secure group-password information, if used, is in `/etc/gshadow`.
3. `1005` - Group ID, or GID.
4. `john,maya` - Supplementary members of the group.

**Why it is needed:** Groups make it easier to give several users the same permissions.

### `/etc/gshadow`

This file stores protected group information, such as:

- Secure group-password data.
- Group administrators.
- Group members.

**Why it is needed:** It protects sensitive group-management information from normal users.

> Do not normally edit these four files directly. Use commands such as `useradd`, `usermod`, and `groupadd`, which perform validation and update related files correctly.

---

## 3. Understanding Linux Command Syntax

A Linux command commonly has this form:

```text
command option argument
```

For example:

```bash
useradd -m john
```

- `useradd` is the command.
- `-m` is an option that changes the command's behavior.
- `john` is an argument containing the new username.

Words such as `username`, `groupname`, `old_username`, and `new_username` in this guide are placeholders. Replace them with real values, and do not type the placeholder names literally.

---

## 4. Creating Users

### 4.1 Create a user with `useradd`

```bash
sudo useradd username
```

Replace `username` with the required login name.

Example:

```bash
sudo useradd john
```

#### Command breakdown

- `sudo` - Runs the command with administrator privileges.
- `useradd` - Creates a new user account.
- `username` - The login name of the account to create.

#### Why use it?

Use this command when you need to create a new account. `useradd` is available on most Linux distributions and is suitable for scripts and administrative work.

> Whether a home directory is created automatically can depend on distribution settings, such as `/etc/login.defs` and files under `/etc/default`. Use `-m` when you explicitly want a home directory.

### 4.2 Create a user and a home directory

```bash
sudo useradd -m username
```

Example:

```bash
sudo useradd -m john
```

#### Parameter breakdown

- `-m` - Creates the user's home directory if it does not already exist. Usually the directory will be `/home/username`.
- `username` - The login name to create.

#### Why use `-m`?

A normal human user needs a place for personal files, settings, shell history, and application configuration. Without a home directory, some programs may not work as expected.

### 4.3 Create a user with a chosen login shell

```bash
sudo useradd -m -s /bin/bash username
```

Example:

```bash
sudo useradd -m -s /bin/bash john
```

#### Parameter breakdown

- `-m` - Creates the user's home directory.
- `-s` - Sets the user's login shell.
- `/bin/bash` - The path to the Bash shell that `-s` assigns to the user.
- `username` - The new login name.

#### Why choose a shell?

The login shell is the program that starts when the user opens a terminal or logs in through a text session. Common examples include `/bin/bash`, `/bin/zsh`, and `/bin/sh`.

The selected shell should exist on the system and should normally be listed in `/etc/shells`.

### 4.4 Create a user with `adduser`

On Debian, Ubuntu, and related systems, you can use:

```bash
sudo adduser username
```

Example:

```bash
sudo adduser john
```

#### Command breakdown

- `sudo` - Runs the command with administrator privileges.
- `adduser` - A friendly, interactive user-creation tool on Debian-based systems.
- `username` - The login name to create.

#### What happens?

The command usually:

1. Creates the account.
2. Creates a home directory.
3. Copies default files into the home directory.
4. Asks for a password.
5. Asks for optional information, such as the full name.

#### Why use it?

`adduser` is easier for beginners because it guides the administrator through the process. Its availability and behavior can vary across Linux distributions.

### `useradd` versus `adduser`

- `useradd` is a lower-level command and is widely available.
- `adduser` is often an interactive helper, especially on Debian-based systems.
- `useradd` is common in automation because its behavior is controlled through options.
- `adduser` is convenient for manually creating a user.

---

## 5. Managing Passwords

### Set or change another user's password

```bash
sudo passwd username
```

Example:

```bash
sudo passwd john
```

#### Command breakdown

- `sudo` - Gives the command administrator privileges.
- `passwd` - Manages a user's password and password state.
- `username` - Identifies the account whose password will be set or changed.

The command asks you to type the new password twice. The password is not displayed while typing.

#### Why is this needed?

A newly created account may not have a usable password. Setting a password allows password-based login when that login method is enabled.

A normal user can change their own password with:

```bash
passwd
```

No username is required because the command applies to the currently logged-in user.

---

## 6. Password Expiration and Account Locking

### 6.1 Set a maximum password age

```bash
sudo chage -M 90 username
```

Example:

```bash
sudo chage -M 90 john
```

#### Parameter breakdown

- `chage` - Changes password-aging information.
- `-M` - Sets the **maximum** number of days for which a password can be used.
- `90` - The password expires after 90 days.
- `username` - The account to update.

#### Why use it?

Some organizations require users to change passwords regularly. This command applies that policy to a particular account.

> Password-expiration policies should match the organization's security policy. Frequent forced changes can encourage weak, predictable passwords if they are not managed carefully.

To view a user's password-aging information:

```bash
sudo chage -l username
```

- `-l` is a lowercase letter L.
- It means **list** the account's current password-aging information.

### 6.2 Lock a user's password

```bash
sudo passwd -l username
```

#### Parameter breakdown

- `passwd` - Manages password information.
- `-l` - Locks the password by marking the stored password hash as unusable.
- `username` - The account whose password is locked.

#### Why use it?

Lock an account when password login should be stopped temporarily, such as when an employee is on extended leave.

> Locking the password may not stop every possible access method. Existing sessions, SSH keys, scheduled tasks, or other authentication methods may still work. Fully disabling access can require additional administrative steps.

### 6.3 Unlock a user's password

```bash
sudo passwd -u username
```

#### Parameter breakdown

- `passwd` - Manages password information.
- `-u` - Unlocks the password.
- `username` - The account to unlock.

#### Why use it?

Use this when the user is allowed to sign in with their password again.

---

## 7. Modifying Existing Users

The `usermod` command changes an account that already exists.

> Before changing a username, home directory, or UID-related settings, it is safest to make sure the user is logged out and has no running processes.

### 7.1 Change a username

```bash
sudo usermod -l new_username old_username
```

Example:

```bash
sudo usermod -l jonathan john
```

#### Parameter breakdown

- `usermod` - Modifies an existing account.
- `-l` - Changes the user's login name. It is a lowercase letter L.
- `new_username` - The new login name.
- `old_username` - The account's current login name.

#### Why use it?

Use this when a login name must be corrected or changed.

> This command changes the login name, but it does not automatically rename the home directory or necessarily rename the user's primary group.

To change both the login name and home directory, you can perform carefully planned commands such as:

```bash
sudo usermod -l new_username old_username
sudo usermod -d /home/new_username -m new_username
```

### 7.2 Change a home directory

```bash
sudo usermod -d /new/home/directory -m username
```

Example:

```bash
sudo usermod -d /home/jonathan -m jonathan
```

#### Parameter breakdown

- `-d` - Sets a new home-directory path in the account information.
- `/new/home/directory` - The full path of the new home directory.
- `-m` - Moves the contents of the current home directory to the new location.
- `username` - The account to modify.

#### Why use `-d` and `-m` together?

`-d` changes the recorded home path. `-m` also moves the existing files. Without `-m`, the account may point to a new location while the old files remain in the previous directory.

> Back up important files before moving a home directory, and ensure the user is logged out.

### 7.3 Change the default shell

```bash
sudo usermod -s /bin/zsh username
```

#### Parameter breakdown

- `-s` - Sets the new login shell.
- `/bin/zsh` - The full path to the Z shell executable.
- `username` - The account to modify.

#### Why use it?

Different shells provide different features and configuration styles. The requested shell must be installed before it is assigned.

Check whether the shell exists:

```bash
command -v zsh
```

- `command` - A shell built-in used here to locate a command.
- `-v` - Prints information showing how the specified command would be found.
- `zsh` - The program being checked.

---

## 8. Deleting Users

### 8.1 Delete an account but keep its home directory

```bash
sudo userdel username
```

#### Command breakdown

- `userdel` - Deletes a user account.
- `username` - The account to delete.

#### Why keep the home directory?

Keeping it allows an administrator to review, archive, or transfer the user's files later.

### 8.2 Delete an account and its home directory

```bash
sudo userdel -r username
```

#### Parameter breakdown

- `-r` - Removes the user's home directory and mail spool in addition to deleting the account.
- `username` - The account to delete.

#### Why use `-r`?

Use it when the account and its local personal files are no longer needed.

> **Warning:** This can permanently delete data. Back up required files first. Files owned by the user outside the home directory may remain and must be checked separately.

To search for files owned by a user before deletion:

```bash
sudo find / -user username 2>/dev/null
```

- `find` - Searches the filesystem.
- `/` - Starts the search at the filesystem root.
- `-user username` - Matches files owned by the named user.
- `2>/dev/null` - Hides permission-error messages by redirecting standard error, file descriptor `2`, to `/dev/null`.

---

## 9. Working with Groups

A **group** is a collection of users. Groups make permission management easier. Instead of giving access to users one at a time, an administrator can grant access to a group.

A user has:

- One **primary group**, used by default when the user creates files.
- Zero or more **supplementary groups**, which provide additional access.

### 9.1 Create a group

```bash
sudo groupadd groupname
```

Example:

```bash
sudo groupadd developers
```

#### Command breakdown

- `groupadd` - Creates a new group.
- `groupname` - The name of the group to create.

#### Why use it?

Create a group when several users need the same access, such as access to a shared project directory.

### 9.2 Add a user to a supplementary group

```bash
sudo usermod -aG groupname username
```

Example:

```bash
sudo usermod -aG developers john
```

#### Parameter breakdown

- `usermod` - Changes an existing user.
- `-a` - **Appends** the user to the group list instead of replacing existing supplementary groups.
- `-G` - Specifies one or more supplementary groups.
- `groupname` - The group to add. Multiple groups can be supplied as a comma-separated list, such as `developers,docker`.
- `username` - The user to modify.

#### Why must `-a` and `-G` be used together here?

Using `-G` without `-a` replaces the user's current supplementary-group list. That could accidentally remove access to other groups. `-aG` adds the new membership while preserving existing supplementary memberships.

The user may need to log out and log in again before a new group membership is applied to all sessions.

### 9.3 View group memberships

```bash
groups username
```

Example:

```bash
groups john
```

#### Command breakdown

- `groups` - Displays group memberships.
- `username` - The account to inspect. If omitted, the command shows groups for the current session.

#### Why use it?

Use it to confirm that a user has been added to the expected groups.

A more detailed alternative is:

```bash
id username
```

- `id` - Displays the user's UID, primary GID, and group memberships.
- `username` - The account to inspect.

### 9.4 Change a user's primary group

```bash
sudo usermod -g new_primary_group username
```

Example:

```bash
sudo usermod -g developers john
```

#### Parameter breakdown

- `-g` - Sets the user's primary group.
- `new_primary_group` - The group that should become primary. It must already exist.
- `username` - The account to modify.

#### Why use it?

New files normally receive the user's primary group unless directory settings or other rules change that behavior. Change the primary group when the user's main work or organizational group changes.

> Changing the primary group does not automatically change the group ownership of files that already exist.

---

## 10. Sudo Access and Privilege Escalation

`sudo` allows an authorized user to run selected commands with elevated privileges, usually as `root`.

This is safer than sharing the `root` password because:

- Access can be granted per user or group.
- Permissions can be limited to individual commands.
- Administrative command use can be logged.
- Access can be removed by changing group membership or sudo rules.

### 10.1 Add a user to the `sudo` group on Debian-based systems

```bash
sudo usermod -aG sudo username
```

#### Parameter breakdown

- `usermod` - Modifies the account.
- `-a` - Appends the group without removing existing supplementary groups.
- `-G` - Updates supplementary-group membership.
- `sudo` - The administrative group used by Debian-based systems.
- `username` - The user receiving sudo access.

### 10.2 Add a user to the `wheel` group on many RHEL-based systems

```bash
sudo usermod -aG wheel username
```

#### Parameter breakdown

- `-a` - Appends the group membership.
- `-G` - Specifies a supplementary group.
- `wheel` - The group commonly configured for sudo access on RHEL-family systems.
- `username` - The user receiving the membership.

#### Why use these groups?

The system's sudo configuration commonly allows members of these groups to run administrative commands. The exact configuration can differ, so verify it on the system.

After adding the group, the user should log out and log back in. Test the configuration with:

```bash
sudo whoami
```

- `sudo` - Runs the following command with elevated privileges.
- `whoami` - Prints the effective username.

If the setup works and the default target is used, it normally prints `root`.

---

## 11. Granting Permission for Specific Commands

Giving full sudo access may be unnecessary. A safer approach is to allow only the commands a user needs.

### 11.1 Edit sudo configuration safely

```bash
sudo visudo
```

#### Command breakdown

- `sudo` - Runs the editor with administrative privileges.
- `visudo` - Opens the sudoers configuration safely, locks it against simultaneous editing, and checks the syntax before saving.

#### Why use `visudo`?

A syntax error in the sudoers configuration can stop sudo from working. `visudo` helps detect errors before they cause a problem.

A better organizational approach is often to create a separate rule under `/etc/sudoers.d/`:

```bash
sudo visudo -f /etc/sudoers.d/username
```

- `-f` - Tells `visudo` to edit a specific file instead of the main `/etc/sudoers` file.
- `/etc/sudoers.d/username` - The separate configuration file to edit.

### 11.2 Understand a sudoers rule

```text
username ALL=(ALL) NOPASSWD: /path/to/command
```

This line means the named user may run the specified command through `sudo` without entering a password.

#### Field-by-field explanation

- `username` - The user to whom the rule applies.
- First `ALL` - The rule applies when the user runs the command from any host covered by this sudoers configuration. This field is mainly important when one sudoers file is managed for multiple hosts.
- `(ALL)` - The user may run the command as any target user. In modern syntax, `(ALL:ALL)` can mean any target user and any target group.
- `NOPASSWD:` - The user is not asked for their password when using this rule.
- `/path/to/command` - The exact absolute path of the allowed executable, such as `/usr/bin/systemctl`.

Example:

```text
john ALL=(root) /usr/bin/systemctl restart nginx
```

This permits `john` to run this exact operation as `root`, normally with sudo's password authentication:

```bash
sudo /usr/bin/systemctl restart nginx
```

#### Why use a specific-command rule?

It follows the **principle of least privilege**: give a user only the access required for the job, rather than full administrator access.

> **Security warning:** Use `NOPASSWD:` only when there is a clear reason. Some commands, editors, interpreters, scripts, or programs that accept user-controlled files can be used to obtain wider access. Rules should use absolute paths and be kept as narrow as possible.

Find an executable's full path with:

```bash
command -v systemctl
```

View the current user's permitted sudo commands with:

```bash
sudo -l
```

- `-l` is a lowercase letter L.
- It lists the sudo privileges available to the current user.

---

## 12. Useful Verification Commands

### Show account identity and group information

```bash
id username
```

### Show the account's `/etc/passwd` entry through the system account database

```bash
getent passwd username
```

- `getent` - Reads an entry from system databases configured through the Name Service Switch.
- `passwd` - Selects the user-account database.
- `username` - Selects the requested account.

`getent` is often better than directly searching `/etc/passwd` because systems may also obtain users from services such as LDAP or other network directories.

### Show a group's database entry

```bash
getent group groupname
```

- `group` - Selects the group database.
- `groupname` - Selects the requested group.

### Show recent login information

```bash
lastlog -u username
```

- `lastlog` - Displays the most recent login information.
- `-u` - Limits the result to a user or UID.
- `username` - The account to inspect.

---

## 13. Simple User-Creation Example

The following creates a normal user, sets a password, adds the user to a project group, and verifies the result:

```bash
sudo groupadd developers
sudo useradd -m -s /bin/bash john
sudo passwd john
sudo usermod -aG developers john
id john
```

What each step does:

1. Creates the `developers` group.
2. Creates `john`, creates `/home/john`, and assigns Bash as the login shell.
3. Sets John's password.
4. Adds John to the `developers` supplementary group without removing other memberships.
5. Displays John's UID, primary group, and supplementary groups.

To grant sudo access on Ubuntu or Debian, an administrator could then run:

```bash
sudo usermod -aG sudo john
```

Only grant this access when the user genuinely needs administrative privileges.

---

## 14. Good Practices

- Use a separate account for each person.
- Do not share passwords or administrator accounts.
- Grant only the access required for each job.
- Prefer groups for shared access.
- Use `-aG` when adding supplementary groups to avoid removing existing memberships.
- Use `visudo` rather than editing sudoers files with a normal editor.
- Use absolute command paths in sudo rules.
- Avoid broad `NOPASSWD` rules.
- Back up important files before moving a home directory or deleting an account.
- Check for running processes and active sessions before renaming or deleting a user.
- Verify changes with `id`, `groups`, `getent`, and `sudo -l`.
- Remember that password locking alone may not disable SSH keys or existing sessions.
- Keep audit logs and review administrator access regularly.

---

## 15. Quick Command Summary

### Create and configure users

```bash
sudo useradd username
sudo useradd -m username
sudo useradd -m -s /bin/bash username
sudo adduser username
sudo passwd username
```

### Manage password aging and locking

```bash
sudo chage -M 90 username
sudo chage -l username
sudo passwd -l username
sudo passwd -u username
```

### Modify users

```bash
sudo usermod -l new_username old_username
sudo usermod -d /new/home/directory -m username
sudo usermod -s /bin/zsh username
```

### Delete users

```bash
sudo userdel username
sudo userdel -r username
```

### Manage groups

```bash
sudo groupadd groupname
sudo usermod -aG groupname username
groups username
id username
sudo usermod -g new_primary_group username
```

### Manage sudo access

```bash
sudo usermod -aG sudo username
sudo usermod -aG wheel username
sudo visudo
sudo visudo -f /etc/sudoers.d/username
sudo -l
```

---

## 16. Final Reminder

User-management commands directly affect access, ownership, and system security. Read each command carefully before running it, especially commands containing `-r`, commands that change group memberships, and sudoers rules. Test changes in a safe environment whenever possible.
