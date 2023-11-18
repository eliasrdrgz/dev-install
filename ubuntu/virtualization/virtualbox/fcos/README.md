# Fedora CoreOS VirtualBox


key and known_hosts
```bash
> ~/.ssh/known_hosts
ssh-keygen -C core -f ~/.ssh/fedora
```

fedora.bu
```yaml
variant: fcos
version: 1.4.0
passwd:
  users:
    - name: core
      ssh_authorized_keys:
        - ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDW7qFS6VKEtbtuf/GeUKiOiclqgh2fS7b06G0A9PaDUqPaOQM05FjSjfDoGCQcswR/knZGSVk4sTTuFGQNOef+V0WZkqdQL16FMzWoKU8jESLKnpyxh2qkOGktJHh3zwpopkn7QDHj9WAjfpeBRAsZda0zNL9R7o/Z+h9eRAoFWmqdxehwjKsMkAVD5pDxQtYfV6O4ny/8x4o4XZPXC75mcrLwdxXb6NUVeTV9N2yYapHKj1LXSTmdGLQ0VTXQhZzUdfn85Z3258WadN0pjZHePXwyATzfOfItV+8nUZpOhoPIqi6NkA8FrjaJ3tQJ71zJxGP854G2XB5bwR5c4cpd37jqWD0ngIgq+efmOc8LT68w5IV38ZDHK04UZS8TIbQQ3KGA+rXmw0rk0UrOVrf5q4OyhcpXFLH9N+FpwpEEk1douN8emtNZNO0f7vdzp93XW0AjP4uyVwPeSVmJW5VMlpU3jI5EwhEacYvuixsaZcSMdyZLUZ3RPfOvG4jw4Bs= core
```

butane > ignition
```bash
docker run --interactive --rm quay.io/coreos/butane:release --pretty --strict < fedora.bu > fedora.ign
```

Virtualbox OVA import, config, boot and stop
```bash
vboxmanage import ~/sw/fedora-coreos-38.20231027.3.2-virtualbox.x86_64.ova
VBoxManage modifyvm "Fedora CoreOS stable" --natpf1 "guestssh,tcp,,2222,,22"
vboxmanage guestproperty set "Fedora CoreOS stable" /Ignition/Config "$(cat fedora.ign)"
#vboxmanage guestproperty get "Fedora CoreOS stable" /Ignition/Config 
# VBoxManage modifyvm "Fedora CoreOS stable" --uartmode1 file "/tmp/fedora-vm.log"
# VBoxManage modifyvm "Fedora CoreOS stable" --uart1 0x3F8 4

vboxmanage startvm "Fedora CoreOS stable" 

vboxmanage controlvm "Fedora CoreOS stable" poweroff
```

```bash
ssh -i ~/.ssh/fedora core@localhost -p 2222
```

## References

https://builds.coreos.fedoraproject.org/prod/streams/stable/builds/38.20231027.3.2/x86_64/fedora-coreos-38.20231027.3.2-virtualbox.x86_64.ova
https://docs.fedoraproject.org/en-US/fedora-coreos/provisioning-virtualbox/
https://docs.fedoraproject.org/en-US/fedora-coreos/authentication/
https://docs.fedoraproject.org/en-US/fedora-coreos/producing-ign/
