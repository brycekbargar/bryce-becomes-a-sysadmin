# Install KVM #

[Yay for great docs!][kvm-faq]

Oh...

Those are actually just a FAQ and don't actually contain setup instructions...

Good thing there's [a whitepaper][whitepaper] with "Easy" in the name. This totally doesn't look impenatrable and enterprisey at all.

Ok. It's actually pretty straightforward so far.

> "Verify the following kernel modules are loaded, and if not load manually: 

>kvm
  
>kvm_intel (only on Intel-based systems)"

I did this by [using `lsmod`][lsmod].

```
bryce @ kvm lsmod ~
%> lsmod | grep 'kvm'
kvm_intel 162153 0
kvm 525409 1 kvm_intel

bryce @ kvm lsmod | grep 'kvm' ~
%>
```

Thankfully they are loaded, since I have no idea how to load them manually...



[lsmod]: http://www.cyberciti.biz/faq/linux-show-the-status-of-modules-driver/
[whitepaper]: http://linux.dell.com/files/whitepapers/KVM_Virtualization_in_RHEL_7_Made_Easy.pdf
[kvm-faq]: http://www.linux-kvm.org/page/FAQ