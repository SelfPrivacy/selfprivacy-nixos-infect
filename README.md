This script - is a modded version of original [nixos-infect](https://github.com/elitak/nixos-infect). It have been revritten to suit needs of SelfPrivacy project. Except transforming your Ubuntu Linux system into NixOS GNU/Linux, it deploys standart services that SelfPrivacy offers to their customers.

## Source Distros

This script has been tested and can install NixOS from the following source distros:

On Digital Ocean:
- Ubuntu 20.04 x64

On Hetzner cloud:
- Ubuntu 20.04

On Scaleway cloud:
- Ubuntu 20.04

## Considerations

nixos-infect is so named because of the high likelihood of rendering a system
inoperable. Use with caution and preferably only on newly-provisioned
systems.

*WARNING NB*: This script wipes out the targeted host's root filesystem when it
runs to completion. Any errors halt execution. It's advised to run with
`bash -x` to help debug, as often a failed run leaves the system in an
inconsistent state, requiring a rebuild (in DigitalOcean panel: Droplet
Settings -> "Destroy" -> "Rebuild from original").

## Digital Ocean

*TO USE:*
- Add any custom config you want (see notes below)
- Deploy the droplet indicated at the top of the file, enable ipv6, add your ssh key
- `cat customConfig.optional nixos-infect | ssh root@targethost`

Alternatively, you may utilize Digital Ocean's "user data" mechanism (found in the Web UI or HTTP API), and supply to it the following example yaml stanzas:

```yaml
#cloud-config

runcmd:
  - curl https://raw.githubusercontent.com/elitak/nixos-infect/master/nixos-infect | PROVIDER=digitalocean NIX_CHANNEL=nixos-20.03 bash 2>&1 | tee /tmp/infect.log
```
Potential tweaks:
- `/etc/nixos/{,hardware-}configuration.nix`: rudimentary mostly static config
- `/etc/nixos/networking.nix`, networking settings determined at runtime tweak
  if no ipv6, different number of adapters, etc.

```yaml
#cloud-config
write_files:
- path: /etc/nixos/host.nix
  permissions: '0644'
  content: |
    {pkgs, ...}:
    {
      environment.systemPackages = with pkgs; [ vim ];
    }
runcmd:
  - curl https://raw.githubusercontent.com/elitak/nixos-infect/master/nixos-infect | PROVIDER=digitalocean NIXOS_IMPORT=./host.nix NIX_CHANNEL=nixos-20.03 bash 2>&1 | tee /tmp/infect.log

```

## Hetzner cloud

Hetzner cloud works out of the box. When creating a server provide the following script as "User data" (this has been tested using Ubuntu 20.04 as a base OS).

```
#!/bin/sh

curl https://raw.githubusercontent.com/elitak/nixos-infect/master/nixos-infect | NIX_CHANNEL=nixos-20.03 bash 2>&1 | tee /tmp/infect.log
```

## Motivation

Motivation for this script: nixos-assimilate should supplant this script
entirely, if it's ever completed. nixos-in-place was quite broken when I
tried it, and also took a pretty janky approach that was substantially more
complex than this (although it supported more platforms): it didn't install
to root (/nixos instead), left dregs of the old filesystem (almost always
unnecessary since starting from a fresh deployment), and most importantly,
simply didn't work for me! (old system was being because grub wasnt properly
reinstalled)
