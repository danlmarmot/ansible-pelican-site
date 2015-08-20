To edit encrypted.yml, open a command prompt in the vault directory, then decrypt with:

```
ansible-vault decrypt encrypted.yml
```

(if encrypted.yml doesn't exist, create it.)

When you're done editing, re-encrpyt it:

```
ansible-vault encrypt vault/encrypted.yml
```