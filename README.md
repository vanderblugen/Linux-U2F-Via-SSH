# linux-u2f-via-ssh
## This enables system wide 2factor for login via ssh<br>
- When this is enabled, it is enabled system wide for all users.  If not fully implemented for a particular user, that user will be unable to connect via ssh.
- I accept no responsibility if anyone uses this.  Use at your own risk.
- Most of this was obtained from https://www.raspberrypi.org/blog/setting-up-two-factor-authentication-on-your-raspberry-pi/
- This is for ssh logon only, not for elevating privlidges.

# Update and Upgrade the OS

```shell
# Update and Upgrade
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get full-upgrade -y
```

# Enable ssh and start it
```shell
sudo systemctl enable ssh
sudo systemctl start ssh
```

# Enable challenge-response
```shell
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config~

SSH_CONFIG="/etc/ssh/sshd_config"

if grep -qE "^#?KbdInteractiveAuthentication " "$SSH_CONFIG"; then
    sudo sed -i 's/^\(#\s*\)*KbdInteractiveAuthentication.*/KbdInteractiveAuthentication yes/I' "$SSH_CONFIG"
elif grep -qE "^#?ChallengeResponseAuthentication " "$SSH_CONFIG"; then
    sudo sed -i 's/^\(#\s*\)*ChallengeResponseAuthentication.*/ChallengeResponseAuthentication yes/I' "$SSH_CONFIG"
else
    echo "KbdInteractiveAuthentication yes" | sudo tee -a "$SSH_CONFIG" > /dev/null
fi
```

# Restart the SSH service for changes to take effect
This will not disconnect existing SSH connections
```shell
sudo systemctl restart sshd
```

# Install google authenticator for 2factor
```shell
sudo apt-get install libpam-google-authenticator -y
```

## Setup 2factor authentication
Each user has to be setup separate and this sets up the 2FA codes for the current user.  Check current user logged in.
```shell
whoami
```
Since the file sshd is changed, until each user is setup for 2FA, they may not be able to log via ssh.<br>
Run this as the user that needs 2FA enabled.<br>
This creates a hidden file `.google_authenticator` in the users home directory where the 2FA information is stored.

```shell
google-authenticator
```

Scan the QR code via a 2FA app (or record the code) and record emergency scratch codes.<br>
Select Y or N for each of the additional security measures.  Recommended Y for each.

## Enable 2factor
RUN THIS ONCE
```shell
# Setup 2factor
sudo cp /etc/pam.d/sshd /etc/pam.d/sshd~
sudo sed  -i '/\@include common-auth/a \\nauth required pam_google_authenticator.so\n' /etc/pam.d/sshd

# Restart the daemon
sudo systemctl restart ssh
```
## All Set
If ssh is the only way you log in, open another ssh session to verify functionality before logging out of the intial seession.
