# linux-u2f-via-ssh
I accept no responsibility if anyone uses this.  Use at your own risk.<br>
Just about everything was pulled from https://www.raspberrypi.org/blog/setting-up-two-factor-authentication-on-your-raspberry-pi/<br>
When this is enabled, it is enabled system wide for all users.  If it is not fully setup for a particular user, that user will not be able to connect via ssh into the machine.<br>
This does not enable 2FA for elevating privlidges, only for ssh for login only.

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
Since the file `sshd` is changed, until each user is setup for 2FA, they may not be able to log via ssh.<br>
Run this as the user that needs 2FA enabled

```shell
google-authenticator
```
Scan the QR code via a 2FA app and record emergency scratch codes.<br>
Select Y or N on each of them.  Recommended Y for each.

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
