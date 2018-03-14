
vagrant status

vboxmanage list runningvms

vboxmanage showvminfo $(vboxmanage list runningvms|cut -f 2 -d ' ') | grep "Memory size"
