# homelab

Homelab setup of mine

## Install

First, you need to install Python dependencies via uv

```
uv sync --all-extras --dev
```

And Ansible Galaxy as well

```
ansible-galaxy install -r requirements.yml
```

Then put your password in `.vault_pass`

```
echo "XXXXXXX" > .vault_pass
```

Then run the following command to start the playbook

```
ansible-playbook playbook.yml --check
```

## License

MIT
