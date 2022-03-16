# linux-u2f-via-ssh
This is mainly for a reference for me to come back to later.  I accept no responsibility if you use it.<br>
Just about everything was pulled from https://www.raspberrypi.org/blog/setting-up-two-factor-authentication-on-your-raspberry-pi/

## Run an upgrade first
```shell
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get full-upgrade -y
```

## Enable ssh and start it
```shell
sudo systemctl enable ssh
sudo systemctl start ssh
```

## Enabling challenge-response
This is an automatic script that makes a bckup of `/etc/ssh/sshd_config`
and Changes `ChallengeResponseAuthentication no` to `ChallengeResponseAuthentication yes` in `/etc/ssh/sshd_config`
```shell
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config~
sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
```

## Restart the ssh daemon
```shell
sudo systemctl restart ssh
```

## Install 2factor authentication
Each user has to be setup separate and this sets up the 2FA codes for the current user.  Check current user logged in.
```shell
whoami
```
Since the file `sshd` is changed, until each user is setup for 2FA, they may not be able to log via ssh.
```shell
sudo apt-get install libpam-google-authenticator -y
google-authenticator
```
Scan the QR code via a 2FA app and record emergency scratch codes


## Enable 2factor
There are 2 ways to ask for the 2FA code.  Before or after the password.  Run only one of them.
### Ask for code before the password
```shell
sudo cp /etc/pam.d/sshd /etc/pam.d/sshd~
sudo sed  -i '/\@include common-auth/i \\nauth required pam_google_authenticator.so\n' /etc/pam.d/sshd
```
### Ask for code after the password
```shell
sudo cp /etc/pam.d/sshd /etc/pam.d/sshd~
sudo sed  -i '/\@include common-auth/a \\nauth required pam_google_authenticator.so\n' /etc/pam.d/sshd
```

### Restart the daemon
```shell
sudo systemctl restart ssh
```

## All Set
If ssh is the only way you log in, open another ssh session to verify functionality before logging out of the intial seession.
