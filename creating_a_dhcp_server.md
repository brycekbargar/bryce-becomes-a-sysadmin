# Creating a DHCP Server #

So... I was all ok with following the guide until I went to [install SpaceWalk][install-spacewalk] and saw it required Java. I nope'd my way out of there as quickly as possible...

It's not that I'm particularly against Java. I mean, I'm a WinForms developer by day....

It's just that I don't really like using technology I don't understand and Java is pretty high up there on the "Black Box and Rat's Nest Configuration" scale. I mean, sure, I could learn everything I need to about it but doing that once with .NET I think gives me enough exposure to "enterprise" technologies for a lifetime...

### The Search for Alternatives ###

I have been exposed to both Puppet and Chef in my career. Though maybe I'm biased because I was trying to manange Windows environments with them, I find them both to be unintuitive and difficult.

Chef, despite having a large collection of community cookbooks always seems to turn into a hot mess of untested and spaghettified ruby scripts specific to a given environment, (i.e. `prod`, `dev`, `qa`...). Then there's the massive chain of "optional" tools, each complicated enough to require dedicated study unless you're ok with blindly following StackOverflow posts. RVM, knife, berkshelf, chef-solo...

Puppet is just weird. It's like a sub-dialect of Ruby (which I'm already not a fan of...). On top of that it has a lot of the same issues as Chef.

Ansible has recently been of interest to me. It seems built for simplicity and  transparency, two things I really like... Buuuut, Ansible Tower is $5000/year. There is talk about this changing with [Red Hat's purchase of Ansible][red-hat-ansible].

> "Q. Will Red Hat open source all of Ansible's technology?

>Most of Ansible is already open source today. Red Hat has long shown its commitment to open-sourcing the technology it acquires, and we have no reason to expect a change in this approach. Our specific plans and timeline will be determined over the coming months."

Of the alternatives, [RunDeck uses java][rundeck] and [Semaphore seems to be a slow-progressing OSS project][semaphore]. In addition, I'm not really sure if Ansible can create and install a machine on KVM from scratch. 

So I started looking at VM provisioning tools and found Cobbler, which SpaceWalk actually uses internally (albeit [a forked version][forked-cobbler]). After spending 10 minutes trying to figure out what any of the cobbler docs were saying and also stumbling upon multiple other provisioning tools in the process ([including stacki which seems promising][stacki]) I realized that I have no idea what I'm doing...

![SpaceWalking Dog](http://i.giphy.com/xDQ3Oql1BN54c.gif)



[stacki]: http://www.stacki.com/
[forked-cobbler]: https://www.redhat.com/archives/spacewalk-list/2016-March/msg00060.html
[semaphore]: https://github.com/ansible-semaphore/semaphore/pulse
[rundeck]: http://rundeck.org/ 
[red-hat-ansible]: https://www.redhat.com/en/about/blog/faq-red-hat-acquires-ansible
[install-spacewalk]: http://linoxide.com/linux-how-to/setup-spacewalk-system-management-centos-rhel-7/ "Installing Spacewalk")