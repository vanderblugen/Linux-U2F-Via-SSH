# linux-u2f-via-ssh
#### I am not the only person to come up with this
#### This is mainly for a reference for me to come back to later.  If you want to use it go ahead
#### I pulled just about everything on this from from https://www.raspberrypi.org/blog/setting-up-two-factor-authentication-on-your-raspberry-pi/

## Run an upgrade first
```shell
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get dist-upgrade -y
```

## Enable ssh and start it
```shell
sudo systemctl enable ssh
sudo systemctl start ssh
```

## Enabling challenge-response
### This is an automatic script that 
#### Makes a bckup of /etc/ssh/sshd_config in the /etc/ssh directory
#### changes ChallengeResponseAuthentication no to ChallengeResponseAuthentication yes in /etc/ssh/sshd_config
```shell
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config~
sudo sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
```

## Restart the ssh daemon
```shell
sudo systemctl restart ssh
```

## Install 2factor authentication
### This sets up the codes for the current user and does not for any other user.  You have to connect as the other users to do that. Being that `sshd` is changed the other users may not be able to log into ssh until its setup on them.
```shell
sudo apt install libpam-google-authenticator -y
google-authenticator
```
### Scan that QR code via app and get emergency scratch codes


### Enable 2factor
#### There are 2 ways to ask for the 2FA code.  Before or after the password
##### Ask for code before the password
```shell
sudo cp /etc/pam.d/sshd /etc/pam.d/sshd~
sudo sed  -i '/\@include common-auth/i \\nauth required pam_google_authenticator.so\n' /etc/pam.d/sshd
```
##### Ask for code after the password
```shell
sudo cp /etc/pam.d/sshd /etc/pam.d/sshd~
sudo sed  -i '/\@include common-auth/a \\nauth required pam_google_authenticator.so\n' /etc/pam.d/sshd
```

### Restart the daemon
```shell
sudo systemctl restart ssh
```

## All Set
