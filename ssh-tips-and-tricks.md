# SSH: Tips and Tricks

## Prevent Connection close due to Timeout

Add the following content which sends a keep alive signal very 4 minutes to `~/.ssh/config`:

```
Host *
  ServerAliveInterval 240
```

... if the file is new, set the access permission correctly:

```shell
chmod 0600 ~/.ssh/config
```

## Escape from a frozen Terminal Session

Press the following keys in sequence:

*[ENTER] [~] [.]*


