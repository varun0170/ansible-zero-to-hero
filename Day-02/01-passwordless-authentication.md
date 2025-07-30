# How to setup Passwordless Authentication

## EC2 Instances

### Using Public Key

```
ssh-copy-id -f "-o IdentityFile <PATH TO PEM FILE>" ubuntu@<INSTANCE-PUBLIC-IP>
```

- ssh-copy-id: This is the command used to copy your public key to a remote machine.
- -f: This flag forces the copying of keys, which can be useful if you have keys already set up and want to overwrite them.
- "-o IdentityFile <PATH TO PEM FILE>": This option specifies the identity file (private key) to use for the connection. The -o flag passes this option to the underlying ssh command.
- ubuntu@<INSTANCE-IP>: This is the username (ubuntu) and the IP address of the remote server you want to access.
--------------------------------------------------------------------------------------------------------------------------------------------------------------
########### common error##################################
****- If anyone ran into the issue of /usr/bin/ssh-copy-id: ERROR: No identities found while setting up passwordless authentication via public key, here is what I did

Solution:

I observed that there was no .ssh  directory itself when I did ls -la ~

I used ssh-keygen command to create public key pair
It will ask a few questions like Where you want to store the key file and enter passphrase etc.
Since I wanted by default path in the .ssh directory which ssh-keygen command was creating, so I just hit enter command.

After that , No identities found error got resolved but bad permission warning was there.
I gave 600 to the pem file. 

Hope this works for guys facing No identities found error.****



now you have public kay and private key in your ansible machine(control node)
copy the public key to the to manage nodes 
----------------------------------------------------------------------------------------------------------------------------------------------------------------
### Using Password 

- Go to the file `/etc/ssh/sshd_config.d/60-cloudimg-settings.conf`
- Update `PasswordAuthentication yes`
- Restart SSH -> `sudo systemctl restart ssh`




#######shell script for passwordless authentication#######
#!/bin/bash

# === Configuration ===
PEM_FILE=~/Downloads/ansible.pem         # Path to your .pem file
USER=ubuntu                              # Default user on EC2 instances
KEY_NAME=id_ed25519                      # SSH key name (without extension)
HOSTS=("18.175.154.87" "13.210.92.44")   # List of EC2 public IPs

# === Step 1: Generate SSH key if not exists ===
if [ ! -f ~/.ssh/$KEY_NAME ]; then
  echo "[INFO] Generating SSH key: ~/.ssh/$KEY_NAME"
  ssh-keygen -t ed25519 -f ~/.ssh/$KEY_NAME -N ""
else
  echo "[INFO] SSH key already exists: ~/.ssh/$KEY_NAME"
fi

# === Step 2: Loop through each host and copy SSH key ===
for HOST in "${HOSTS[@]}"; do
  echo "[INFO] Copying SSH key to $USER@$HOST"
  
  ssh-copy-id \
    -i ~/.ssh/$KEY_NAME.pub \
    -o "StrictHostKeyChecking=no" \
    -o "IdentitiesOnly=yes" \
    -o "IdentityFile=$PEM_FILE" \
    $USER@$HOST
  
  if [ $? -eq 0 ]; then
    echo "[SUCCESS] SSH key copied to $HOST"
  else
    echo "[ERROR] Failed to copy SSH key to $HOST"
  fi
done

echo "[DONE] Passwordless SSH setup complete."

