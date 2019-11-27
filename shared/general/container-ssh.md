To help you debug and develop your applications, we've provided a browser based terminal and a command line tool for easy SSH access to your devices. With these tools, you have console access to any of your running containers, as well as to the host OS, letting you test out small snippets of code and check system logs on your device.

__Note:__ Host OS SSH access is available for devices running {{ $names.os.lower }} version 2.7.5 and above.

## Using the dashboard web terminal

To use this feature, navigate to your application and select the device you want to access. You will see a *Terminal* window below the *Logs* window:

![SSH Terminal](/img/common/device/terminal.png)

If your device is online, select a target and click the blue *>_ Start Terminal session* button. A terminal session should be initiated for you in a second or two. If you would like a bigger window for the terminal, you can click the *Expand* button in the upper-right corner.

__Note:__ To copy and paste in the terminal window,  you cannot use the normal Ctrl + C and Ctrl + V shortcuts. You can either select Copy and Paste from a menu, or use Ctrl + Insert for copy and Shift + Insert for Paste. For MacOS users, ⌘ + C and ⌘ + V work as expected.

## Using `{{ $names.company.short }} ssh` from the CLI

If you prefer to work from the command line, you can use [`{{ $names.company.short }} ssh`][balena-ssh] to connect to your application containers and the host OS. First, you will need to install the [{{ $names.company.lower }} Command Line Interface (CLI)](/tools/cli/). Once that is set up, run the following in your development machine's terminal:

```shell
$ {{ $names.company.short }} ssh <device-uuid>
```

`<device-uuid>` is the unique identifier for the device you want to access, which can be found via the dashboard or in the output of the `{{ $names.company.short }} devices` CLI command. By default, SSH access is routed into the host OS shell. However, you can SSH into a service by specifying its name as part of the command:

```shell
$ {{ $names.company.short }} ssh <device-uuid> main
```

This also works in multicontainer applications, simply pass the name of the appropriate service as defined in docker-compose.yml you'd like to access the shell for.

For devices running development images on your local network, you can create a direct SSH connection by specifying the hostname using the `<device-uuid>.local` syntax or local IP address of the device (development images have SSH enabled by default). Using {{ $names.company.short }} ssh in this way doesn't use the balena VPN and instead makes a direct SSH connection to the device. For example:

```shell
$ balena ssh <device-uuid>.local
```

When not accessing via a local network or for production images, `{{ $names.company.short }} ssh` uses the {{ $names.company.short }} VPN to create a secure tunnel to the device and then forward SSH traffic between it and your development machine. For this method, you must have a SSH key configured on your development machine and added to the {{ $names.cloud.lower }} dashboard.

## Add an SSH key to {{ $names.cloud.lower }}

To add an SSH key, go to the _Preferences_ page of {{ $names.cloud.lower }} and select the _SSH Keys_ tab.

![SSH key preferences](/img/common/main_dashboard/eekVBTI.png)

You may either import an existing SSH key from GitHub or manually enter the public SSH key of an existing SSH key on your development machine.

If you do not have an existing key you can follow [GitHub's documentation][github-ssh], skipping the step about adding the key to your GitHub account, and instead adding the key to your {{ $names.cloud.lower }} account.

## Install an SSH server in a container

`{{ $names.company.short }} ssh` makes use of the {{ $names.company.short }} VPN connection to access a device. This allows you to access and test devices wherever they are. If you want to SSH only on the internal network, you can install an SSH server in your container, as we show in the [balena-openssh][balena-openssh] project.

One note is that if you run your own SSH in the container you won't automatically get your environment variables in the SSH session. To bring them in, simply run `. <(xargs -0 bash -c 'printf \"export %q\n\" \"\$@\"' -- < /proc/1/environ)`. Now any operations or code you run from the SSH session will be able to access the environment variables you set on your {{ $names.company.lower }} dashboard ([see gitter discussion for more info](https://gitter.im/resin-io/chat?at=57be336fce157d1b57a19e82)). Alternatively, use the following command in a Dockerfile to update the root's `.profile` so {{ $names.company.short }} variables are sourced at each tty/ssh login:

```
echo ". <(xargs -0 bash -c 'printf \"export %q\n\" \"\$@\"' -- < /proc/1/environ)" >> /root/.profile
```

[balena-ssh]:/reference/cli/#ssh-uuid-
[github-ssh]:https://help.github.com/en/github/authenticating-to-github/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent
