# Note for using Remote Login in Ubuntu 24.04

## Problem / Background
* Using ubuntu 24.04 remote login feature from macos kind of difficult, because lack of mature rdp app in macos.
* Using Remote Desktop feature are not fully remote, we need to set it up locally first, then do the remote. This will be repeat on every boot / logged.

## Choosing the solution, Pro and Cons.
There are two scenario you could choose, using Remote Login or Remote Desktop.

### 1. Remote Login:

Pro:
* Can do remote even in login screen.

Cons:
* Cannot use Windows App Beta application

### 2. Remote Desktop:

Pro:
* Can use Windows App Beta application.

Cons:
* Cannot do remote in login screen.
* User need to enable autologin, and disable screen locked feature, to improve remote experience.
* Need to use unencrypted mode (no keyring), to prevent the system, auto changing the password.

## The solution
### Remote Login
* Open Settings, then enable System -> Remote Desktop -> Remote Login.
* Input the login details: username and password.
* By default, remote user cannot change certain settings. We need to add some rule to allow remote admin change it. For example:

Allow the Remote Login settings:
Make a rules file lets say `/etc/polkit-1/rules.d/40-allow-remote-user-admin.rules`
```
polkit.addRule(function(action, subject) {
	polkit.log(action.id);
    if (action.id === "org.gnome.controlcenter.user-accounts.administration") {
        polkit.log("Polkit check for user accounts admin - user: " + subject.user + " is remote: " + !subject.active);
        if (subject.user === "your_user_name") {
            return polkit.Result.AUTH_ADMIN_KEEP;
	}
    }
});
```
Then restart the service `sudo systemctl restart polkit`.

If you need to get other settings just read `action.id` that logged using `polkit.log`.

To get the log we need to turn off the default `--no-debug` flag for polkit.

Edit the service file: `/usr/lib/systemd/system/polkit.service`

Then change `ExecStart=/usr/lib/polkit-1/polkitd --no-debug` into `ExecStart=/usr/lib/polkit-1/polkitd`.

Then reboot or `systemctl daemon-reload`.

### Remote Desktop
* Open Settings, then enable System -> Remote Desktop -> Desktop Sharing (and Remote Control).
* First time enabling this, ubuntu will ask the user to input some keyring password. Dont give password (keep it blank). This will lead to some warning about unencrypted mode. We need to do this to prevent the system changing the login details password on each logged in.
* Input the login details: username and password.
* Open Settings, then disable all feature on Privacy & Security -> Screen Lock. This to minimalize the risk into going to the login screen, since we could not remote the login screen.
* Open Settings, then enable automatic login on System -> Users. Again to avoid the login screen.

## Another note on enabling the ssh
Theres option on enabling ssh on the Settings -> System -> Secure Shell. But this doesn't work.
Because by default, the `sshd` are not installed. So we need to install it first, `sudo apt install openssh-server`.
