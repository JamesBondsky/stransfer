*stransfer.sh* is a script to facilitate routine operations of websites' transfer from the one server to another. It's a really quick and easy way to copy your database and files.

# Used utilities
This utilities are necessary for *stransfer.sh*:
- sshpass
- OpenSSH client/server
- scp or rsync
- mysql

# Options
*stransfer.sh* reads its settings from a configuration file. This file is named *.stransfer.conf* by default and located in the same directory with the main script. You can change the  location of the file using a key *-f*:

`sh stransfer.sh -f path/to/your/config/.stransfer.conf`

The configuration file is included by *source* command so you need to set the options as bash variables

## List of variables:
- *TRANSFER_METHOD* - the method of files transfering, rsync (by default) or scp
- *REMOTE_SSH_CONNECTION* - the ssh connection string in a format *\<user>@\<host>*
- *AUTH_TYPE* - a type of ssh authentification. Allowed values: *stransferPubkey* or *sshpass*.
   
   ssh with different *AUTH_TYPE*:

   AUTH_TYPE=sshpass: `sshpass -p "${REMOTE_SSH_PASSWORD}" ssh -p "${SSH_PORT}"  -o BatchMode=yes -o ConnectTimeout=5 "${REMOTE_SSH_CONNECTION}"`

   AUTH_TYPE=stransferPubkey: `ssh -p "${SSH_PORT}" -i "~/.ssh/stransfer"  -o BatchMode=yes -o ConnectTimeout=5 "${REMOTE_SSH_CONNECTION}"`

   AUTH_TYPE isn't set: `ssh -p "${SSH_PORT}" -o BatchMode=yes -o ConnectTimeout=5 "${REMOTE_SSH_CONNECTION}"`

   If the public key authentification is necessary, check the option *PubkeyAuthentication* isn't set to *no* in the sshd configuration  
- *REMOTE_SSH_PASSWORD* - the password for ssh conection. It's used when *AUTH_TYPE* is set to *sshpass*
- *SSH_PORT* - the port of the ssh connection, it's set to *22* by default
- *LOCAL_PATH* - the files' transfering aim directory on the local server  
- *REMOTE_PATH* - the files' transfering source directory on the remote server
- *CLEAR_LOCAL_DIR* - if it set to *true* (by default), a local directory is cleared before files transfer
- *LOCAL_DB_NAME* - a database name on the current server
- *LOCAL_DB_USER* - a user with permissions to the local database
- *LOCAL_DB_PASSWORD* - the local database user password
- *REMOTE_DB_NAME* - a database name on the remote server
- *REMOTE_DB_USER* - a user with permissions to the remote database
- *REMOTE_DB_PASSWORD* - the remote database user password
- *LOCAL_DUMP_PATH* - a temporary local directory for database dump
- *REMOTE_DUMP_PATH* - a temporary remote directory for database dump

# Usage
When the script is running, select the operation in the main menu:

1. **Create a rsa public key and add it to the remote server**<br />
It tries to create and add ssh public key to remote server using ssh-keygen and ssh-copy-id. Created key is named *stransfer* and has not got a password. This key will be used when you set the option *AUTH_TYPE* to *stransferPubkey*.
2. **Copy files from the remote server (rsync or scp)**<br />
Copy files from *REMOTE_PATH* to *LOCAL_PATH* using *TRANSFER_METHOD*.
3. **Copy database from the remote server**<br />
Create a database dump in the remote server then copy and import it to local database. Now only mysql is supported so it uses mysqldump to a dump creation. Options *LOCAL_DUMP_PATH* and *REMOTE_DUMP_PATH* set directories to dump storage, both directories will be removed after the operation's finish.
4. **Exit**<br />
Quit the script

Also you can run every step using the key *-s*:

`sh stransfer.sh -s addStransferPubKey -s copyFiles -s copyDatabase`

# Known problems and solutions
When you create copy rsa public key (the operation #1 on the main menu) you can see the error message:
<br />
<br />
`
/usr/bin/ssh-copy-id: ERROR: No identities found
`
<br />
<br />
or:
<br />
<br />
`
Bad port 'exec sh -c 'cd; umask 077; test -d .ssh || mkdir .ssh ; cat >> .ssh/authorized_keys && (test -x /sbin/restorecon && /sbin/restorecon .ssh .ssh/authorized_keys >/dev/null 2>&1 || true)''
`
<br />
<br />
The reason may be that you use the old openssh version and the syntax of ssh-copy-id was changed. Try to set the variable *OLD_SSH_VERSION* to *true* and repeat the operation.
<br />
In contrast, if you see a message:
<br />
<br />
`
Usage: /usr/bin/ssh-copy-id [-h|-?|-f|-n] [-i [identity_file]] [-p port] [[-o <ssh -o options>] ...] [user@]hostname
        -f: force mode -- copy keys without trying to check if they are already installed
        -n: dry run    -- no keys are actually copied
        -h|-?: print this help
`
<br />
<br />
it means you have a modern version of ssh, but the variable *OLD_SSH_VERSION* is set to *true*. 

# Lisence

[MIT](LICENSE.md)
