# Ubuntu 26のAnsibleで「waiting for privilege escalation prompt」エラーが出る

```
TASK [avahi : Ensure avahi is installed] ****************************************
[ERROR]: Task failed: Timeout (12s) waiting for privilege escalation prompt:
Origin: /Users/neet/ghq/github.com/neet/homelab/roles/avahi/tasks/main.yml:2:3

1 ---
2 - name: Ensure avahi is installed
    ^ column 3

fatal: [compute-mitaka-02.local]: UNREACHABLE! => {"changed": false, "msg": "Task failed: Timeout (12s) waiting for privilege escalation prompt:", "unreachable": true}
```

## やったこと

`sudo.ws` （従来のsudo）に一時的に切り替える。

```
$ sudo update-alternatives --set sudo /usr/bin/sudo.ws
```

パッチがリリースされたらこれで戻す

```
$ sudo update-alternatives --auto sudo
```

## 参考

- ["become: true" not working with Ubuntu 26.04 LTS : r/ansible](https://www.reddit.com/r/ansible/comments/1t6ie61/become_true_not_working_with_ubuntu_2604_lts/)
- [Understanding sudo-rs on Ubuntu 26.04](https://linuxconfig.org/understanding-sudo-rs-on-ubuntu-26-04#switching-sudo)

