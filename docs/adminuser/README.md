
## Create admin user and provide sudo access

Login to Your ubuntu server and create admin user with suo access

Step 1 — Logging Into Your Server

```sh
ssh root@your_server_ip_address
```

Step 2 — Adding a New User to the System

```sh
adduser ocpadmin
```

Output

```sh
Adding user `ocpadmin' ...
Adding new group `ocpadmin' (1001) ...
Adding new user `ocpadmin' (1001) with group `ocpadmin' ...
Creating home directory `/home/ocpadmin' ...
Copying files from `/etc/skel' ...
New password: 
Retype new password: 
passwd: password updated successfully
Changing the user information for ocpadmin
Enter the new value, or press ENTER for the default
	Full Name []: OCP Admin
	Room Number []: 
	Work Phone []: 
	Home Phone []: 
	Other []: 
Is the information correct? [Y/n] Y
```

Next, you’ll be asked to fill in some information about the new user. It is fine to accept the defaults and leave this information blank

Step 3 — Adding the User to the sudo Group

```sh
usermod -aG sudo ocpadmin 
```
