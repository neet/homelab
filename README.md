# homelab

Homelab setup of mine

## Install

First, install dependency via uv

```
uv sync --all-extras --dev
```

Put your password in `.vault_pass`

```
echo "XXXXXXX" > .vault_pass
```

Then run the following command to start the playbook

```
ansible-playbook playbook.yml --check
```

## License

MIT
