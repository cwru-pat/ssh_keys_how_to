# Guide to seting up SSH keys
This guide was written for macOS but should work on \*nix based systems.

## The short and the quick is
- Create and set permissions (if needed), and goto the ssh directory
```shell
mkdir -m 0700 ~/.ssh/
cd ~/.ssh/
```
- Generate keys:
  ``` shell
  ssh-keygen -t rsa -b 4096 -C "<your_comment>"
  ```
    - enter `your_key_file_name`
    - enter passphrase (twice)

- Load keys into keychain and into agent
  ```shell
  ssh-add -K your_key_file_name
  ```
- Create config file (this is optional but useful. See `man ssh_config` or read the rest of the document)
- Load keys onto remote servers
```shell
ssh-copy-id -i /path/to/key/file user@host
```
- Connect to remote server with `ssh user@host` or with `ssh short_name` if you have used a `ssh_config` file.
- To add a key to GitHub you will need to copy the contents of the `.pub` corresponding to your key and add it to https://github.com/settings/keys

## Details and commentary

Here are some thoughts and opinions of the things I have told you to do.

I have chosen RSA encryption because that is a reasonable standard. Do not use DSA as it is not considered a secure algorithm. The elliptic curve options are also reasonable to use.

We will be using a 4096 as that is the current maximum standard for encryption.

There is no reason not to generate a separate key for each server. This will prevent a single key being compromised from compromising all of your computers.

A further step of protection is to use passphrases for each key. This means that even if someone gets access to the private key file, they will still need to enter a passphrase to decrypt and use it. Not to worry you won't need to enter a passphrase each time you use the key as this can be loaded into your system keychain.

Now the more detailed version of the short:

Navigate to or create `~/.ssh/`. Note if you create `~/.ssh/` by hand you should change the permissions to `0700`, you can do this on create as `mkdir -m 0700 ~/.ssh/`. Alternatively if you `ssh` somewhere you will automatically create the directory with the correct permissions. We will store everything related to SSH things from keys to configs in this directory.

To generate a key you will need to enter:
```shell
ssh-keygen -t rsa -b 4096 -C "<your_comment>"
```

The `-t` flag determines the type of encryption (`rsa` is good, do not use `dsa`) and `-b` the strength (`4096` is the current standard). Note that `<your_comment>` can be any comment, perhaps something to remember the key by or identify yourself.

After running the command you will get three prompts:
  1. The first will be for a file name. Some utilities assume that keys have the name space `id_<type>_*` where `<type>` is something like `rsa`. Thus it may be wise to follow this naming scheme.
  2. The second will be a passphrase. This is something you will need to enter when loading the key into the agent (which we will do later) and whenever the agent times out. We will load this into our Keychain so that you don't have to re-enter this each reboot or such. However you can make this more strict. This prevents the keys from being useful by themselves as the passphrase will be necessary at least on first use.
  3. The last will be the passphrase again.

Next we will add this key to our `ssh-agent` (the daemon that takes care of using the keys while connecting) with
```shell
ssh-add -K your_key_file_name
```
This line will also add the key to the Keychain. You can check this as it will be added as an application password with name `SSH: name_of_key_file` (on macOS).

You can list the keys loaded into the agent with `ssh-add -l`. To delete a key `shh-add -d your_key_file_name_to_delete`.

Now that we have created the key files we will set up of SSH config file. Although for simple setups it is not necessary to do, this can simplify the logging in process and ensures that key passphrases are loaded in and from your keychain correctly.

Create a file named `~/.ssh/config`. This file uses `#` for line comments. I recommend reading the man page for the syntax and options of this file with `man ssh_config`, but this document should provide the basics.

This config file allows you to set up options for both specific servers and groups of them such as your remote username, what port to connect, what key to use, setup short-names for the servers locally, and  many other useful things. This file can take advantage of simple patterns and tokens (see the `man` page for examples). Note that the file loads the first entry found for each host and later entries won't overwrite. Thus general behavior should follow specific behavior. An example of a file may be
```shell
Host server1
#short name of server
  IdentityFile /path/to/key/file1
  #path to the private key
  User user1
  #username on the remote server
  HostName %h.domain.edu
  #actual address: server1.domain.edu
Host server2
  IdentityFile /path/to/key/file2
  HostName %h.domain.edu
Host *
#use for servers if the field is not defined.
  AddKeysToAgent yes
  #Loads keys into the ssh-agent
  UseKeychain yes
  #use keychain to unlock key files
  IdentitiesOnly yes
  #Only use key files named
```
Note that this file correctly recognizes the use of `~`.

With this config file we would connect to `server1` with `ssh server1` using the key `/paht/to/key/file1` and user name `user1`. In other words it runs the command
```shell
ssh -i /path/to/file user1@server1.domain.edu
```
However before this will actually work we will need to load the public key in the remote servers `authorized_keys` file. The simplest way of doing this is to use `ssh-copy-id` as
```shell
ssh-copy-id -i /path/to/key/file server_short_name
```
Alternatively this can be done by `scp`ing the file to the remote server and concatenating it to the `~/.ssh/authorized_keys`.

Lastly to add a ssh key to your GitHub account (for easy access to private repos you have access to) you will need to add the contents of the `.pub` file corresponding to the key you want to use to https://github.com/settings/keys.
