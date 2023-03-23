# Getting started

Here we will detail the first steps on accessing our HPC cluster, Kelvin2.

## Applying for an account

To gain an account to access Kelvin2, please fill the form found here : 

[https://www.ni-hpc.ac.uk/Access/](https://www.ni-hpc.ac.uk/Access/)

Please allow 48 hours for your account to be created.
Users will be emailed confirmation when their account has been proccessed.
<br />
<br />

## Credentials
### Username
- QUB - Username will be your student/staff number.<br />
- UU & EPSRC - Username will be the first letter of your first name, followed by your surname in lowercase e.g Joe Bloggs -> jbloggs

### Password
- QUB - Password will be that which is associated with QUB AD and QOL.
- UU & EPSRC - Password will be created when setting up remote access.
<br />

## Connecting to Kelvin2
Connection to Kelvin2 is via ssh.<br />
<br />
There are 2 ways to connect to Kelvin2:<br />
1. QUB network<br />
2. Remote access<br />


#### QUB Network
 If you are within a QUB network you can connect to kelvin2 using your credentials and the following ssh command : 

     ssh kelvin2.qub.ac.uk 

#### Remote Access

 Kelvin2 can be accessed from outside the QUB network by using SSH keys.<br /> 
 - Kelvin server name: login.kelvin.alces.network<br />
    - Port: 55890

<ins>Remote access from a Linux computer<ins><br />


- Open a terminal and create a public/private key pair on the remote machine:

        ssh-keygen -t rsa –f ~/.ssh/my-kelvin-key

All users MUST set a password for the passphrase when prompted.

- Copy the public key generated to Kelvin2

The easiest way to do this is to place the  contents of the public key file into the authorized users file in your Kelvin2 home directory.

    cat ~/.ssh/my-kelvin-key.pub

Login to Kelvin2 and paste the above output into the end of the file:

     ~/.ssh/authorized_keys

Note: if you are away from the QUB campus and hence do not have access to Kelvin2,  send the public key via a method found [here](https://www.ni-hpc.ac.uk/contact/), whereupon an administrator will add it for you.

-	Login using SSH

        ssh –p 55890 –i ~/.ssh/my-kelvin-key <username>@login.kelvin.alces.network

<ins>Remote access from a Windows computer<ins><br />

Two popular terminal emulators are putty and MobaXTerm. If using putty there is a tool called puttygen which will generate public/private keys.<br />

- Open a terminal and create a public/private key pair on the remote machine:

        ssh-keygen

Call your key `my-kelvin-key`.<br />
 All users MUST set a password for the passphrase when prompted.

- Copy the public key to Kelvin2.<br />

The easiest way to do this is to place the contents of the public key file into the authorized users file in your Kelvin2 home directory.

    type my-kelvin-key.pub

Login to Kelvin2 and paste the above output into the end of the file:

     ~/.ssh/authorized_keys
 
Note: if you are away from the QUB campus and hence do not have access to Kelvin2,  send the public key via a method found [here](https://www.ni-hpc.ac.uk/contact/), whereupon an administrator will add it for you.

-	Login using SSH

        ssh –p 55890 –i ~/.ssh/my-kelvin-key <username>@login.kelvin.alces.network


## Copying data to Kelvin2
To copy files across to kelvin use pscp or scp. Give the full path to the directory/file to be copied and connect using your credentials, like below:<br />

<ins>Windows OS with putty installed<ins><br />

    pscp –p 55890 "C:\Users\MyPC\Documents\test.txt" -i my_kelvin_key <username>@login.kelvin.alces.network

<ins>Linux OS:<ins><br />

    scp –p 55890 "C:\Users\MyPC\Documents\test.txt" -i my_kelvin_key <username>@login.kelvin.alces.network
