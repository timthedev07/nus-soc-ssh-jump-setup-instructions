# nus-soc-ssh-jump-setup-instructions

As the deprecation of the SoC Forticlient VPN is approaching, users should be setting up to use the proxy server instead of the VPN.

This guide sets up SSH access to SoC servers through the SoC jump host.

## 1. Why this setup

You cannot SSH directly into internal SoC machines like:

```bash
xlogin.comp.nus.edu.sg
```

unless you are physically at the SoC or, as one previously would, are using the VPN.

Instead, you connect through the SoC jump host:

```bash
stujump.comp.nus.edu.sg
```

The path is:

```text
your laptop
  -> stujump.comp.nus.edu.sg
  -> soc server
```

## 2. Required files

You should be generating a pair of keys if not already:

```
ssh-keygen -t ed25519 -f ~/.ssh/id_ed25519_nus_soc -C "e1234567@nus-soc"
```

throughout the guide we name the key pair as the following

```bash
~/.ssh/id_ed25519_nus_soc
```

to differentiate itself with the other keys you might have.

and the public key is:

```bash
~/.ssh/id_ed25519_nus_soc.pub
```

The private key stays on your machine.

The public key is what you register with SoC.

## 3. Fix SSH permissions

Run:

```bash
chmod 700 ~/.ssh
chmod 600 ~/.ssh/id_ed25519_nus_soc
chmod 644 ~/.ssh/id_ed25519_nus_soc.pub
```

If the `.pub` file is missing, recreate it from the private key:

```bash
ssh-keygen -y -f ~/.ssh/id_ed25519_nus_soc > ~/.ssh/id_ed25519_nus_soc.pub
```

Show the public key:

```bash
cat ~/.ssh/id_ed25519_nus_soc.pub
```

Register that exact public key with the SoC SSH key system.

## 4. Register the public key with SoC

Before testing `stujump`, you need to register the public key you just generated.

You must be on the **SoC VPN** or the **physical SoC/NUS network** for this step.

The normal NUS VPN may not be enough.

SSH into the SoC key registration server:

```bash
ssh e1234567@skeys.comp.nus.edu.sg
```

Replace `e1234567` with your own SoC username, for example:

```bash
ssh e1718960@skeys.comp.nus.edu.sg
```

This opens an interactive key registration menu.

Type:

```text
jump
```

It should then prompt you to paste in the public key.

Print your public key locally:

```bash
cat ~/.ssh/id_ed25519_nus_soc.pub
```

Copy the full output. It should look like:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... e1718960@nus-soc
```

Paste that full public key into the `skeys` prompt.

Then press:

```text
Control + D
```

This exits the input and finishes registering the key.

## 5. Test the jump host first

Do not debug `xlogin` until `stujump` works.

Run:

```bash
ssh -vvv \
  -i ~/.ssh/id_ed25519_nus_soc \
  -o IdentitiesOnly=yes \
  e1234567@stujump.comp.nus.edu.sg
```

You want to see something like:

```text
Offering public key: /Users/yourname/.ssh/id_ed25519_nus_soc
Server accepts key
```

If you see:

```text
Permission denied (publickey)
```

then one of these is wrong:

* the public key was not registered correctly
* the wrong public key was pasted into `skeys`
* the wrong username is being used
* SSH is offering the wrong private key

## 6. Create the SSH config

Edit:

```bash
nvim ~/.ssh/config
```

or use vi; anything but nano, wtf is it

Add:

```sshconfig
Host stujump
    HostName stujump.comp.nus.edu.sg
    User e1234567
    IdentityFile ~/.ssh/id_ed25519_nus_soc
    IdentitiesOnly yes

Host xlogin
    HostName xlogin.comp.nus.edu.sg
    User e1234567
    ProxyJump stujump
    IdentityFile ~/.ssh/id_ed25519_nus_soc
    IdentitiesOnly yes
```

N.b. you should be replacing the xlogin (which is what the dochub page uses as a placeholder) with whatever cluster you want to ssh into.

Fix config permissions:

```bash
chmod 600 ~/.ssh/config
```

## 7. Connect

First test the jump host:

```bash
ssh -vvv stujump
```

It should ideally be outputting something like receiving packets of some size.

Then connect to `xlogin` through the jump host:

```bash
ssh xlogin
```

Replace xlogin with whatever cluster you need.


## 5. Create the SSH config
