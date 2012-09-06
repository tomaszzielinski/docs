========================
 Miscellaneous tips&tricks
========================

Postfix
=======

* After updating ``/etc/aliases``, in order for Postfix to see the updated aliases,
  ``newaliases`` command has to be issued in bash
* Set up a Gmail relay:

  * http://serverfault.com/questions/119278/configure-postfix-to-send-relay-emails-gmail-smtp-gmail-com-via-port-587
  * http://productforums.google.com/forum/#!category-topic/gmail/composing-and-sending-messages/7QWAO_aunhc
  * http://www.postfix.org/postconf.5.html#relay_transport
  * http://www.postfix.org/TLS_README.html
  * Postfix configuration is not that complex, there's a lot of options but the docs are well-written
  * http://www.howtoforge.com/forums/showthread.php?p=105989


git
======
* In the precommit hook one can add ``ack-grep "pdb\.set_trace\(\)"``
  to find all remaining pdb calls. `You can also do much more there <http://tech.yipit.com/2011/11/16/183772396/>`_.


Virtualbox (+ Ubuntu)
=======================

* Port mapping in the NAT mode `<http://superuser.com/questions/424083/virtualbox-host-ssh-to-guest>`_.
  Then: ``ssh -p 2222 user@localhost``.
* An imported Vbox image cannot connect to the network: `<https://forums.virtualbox.org/viewtopic.php?f=6&t=24383>`_.
  One have to comment out entries in ``/etc/udev/rules.d/70-persistent-net.rules`` (in the guest OS of course)
  as they contain MAC address of the formerly used virtual machine, and the restart the guest.
* Using host's DNS resolver in the NAT mode: `<http://www.virtualbox.org/manual/ch09.html#nat_host_resolver_proxy>`_.
  That makes host's ``/etc/hosts`` used for DNS lookups in the virtual machine.
* Watch out for DNS resolving in Ubuntu Precise (12.04+):
  `<http://www.stgraber.org/2012/02/24/dns-in-ubuntu-12-04/>`_,
  `<https://plus.google.com/105897381673403508112/posts/UNrkEtAw6MX>`_.

