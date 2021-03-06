Connecting to Cirrus
====================

On the Cirrus system interactive access can be achieved via SSH, either
directly from a command line terminal or using an SSH client. In
addition data can be transfered to and from the Cirrus system using
``scp`` from the command line or by using a file transfer client.

This section covers the basic connection methods. The connection
procedure is then expanded on and the use of SSH agent is described for
ease of access.

For SSH agents that we recommend you use on your local workstion, please see 
the :doc:`ssh_clients` for more information.

Interactive access
------------------

To log into Cirrus you should use the "login.cirrus.ac.uk" address:

::

    ssh [userID]@login.cirrus.ac.uk

Initial passwords
~~~~~~~~~~~~~~~~~

The SAFE web interface is used to provide your initial password for
logging onto Cirrus (see the :doc:`../safe-guide/introduction` for more details on requesting
accounts and picking up passwords).

**Note:** you may now change your password on the Cirrus machine itself
using the *passwd* command. This change will not be reflected in the
SAFE. If you forget your password, you should use the SAFE to request a
new one-shot password.

Making access more convenient using a SSH Agent
-----------------------------------------------

Using a SSH Agent makes accessing the resources more convenient as you
only have to enter your passphrase once per day to access any remote
resource - this can include accessing resources via a chain of SSH
sessions.

This approach combines the security of having a passphrase to access
remote resources with the convenince of having password-less access.
Having this sort of access set up makes it extremely convenient to use
client applications to access remote resources, for example:

-  the `Tramp <http://www.gnu.org/software/tramp/>`__ Emacs plugin that
   allows you to access and edit files on a remote host as if they are
   local files;
-  the `Parallel Tools Platform <http://www.eclipse.org/ptp/>`__ for the
   Eclipse IDE that allows you to edit your source code on a local
   Eclipse installation and compile and test on a remote host;

**Note:** this description applies if your local machine is Linux or macOS.
The procedure can also be used on Windows using the PuTTY SSH
terminal with the PuTTYgen key pair generation tool and the Pageant SSH
Agent. See the `PuTTY
documentation <http://the.earth.li/~sgtatham/putty/0.62/htmldoc/>`__ for
more information on how to use these tools.

**Note:** not all remote hosts allow connections using a SSH key pair.
If you find this method does not work it is worth checking with the
remote site that such connections are allowed.

Setup a SSH key pair protected by a passphrase
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using a terminal (the command line), set up a key pair that contains
your e-mail address and enter a passphrase you will use to unlock the
key:

::

    ssh-keygen -t rsa -C "your@email.com"
    ...
    -bash-4.1$ ssh-keygen -t rsa -C "your@email.com"
    Generating public/private rsa key pair.
    Enter file in which to save the key (/Home/user/.ssh/id_rsa): [Enter]
    Enter passphrase (empty for no passphrase): [Passphrase]
    Enter same passphrase again: [Passphrase]
    Your identification has been saved in /Home/user/.ssh/id_rsa.
    Your public key has been saved in /Home/user/.ssh/id_rsa.pub.
    The key fingerprint is:
    03:d4:c4:6d:58:0a:e2:4a:f8:73:9a:e8:e3:07:16:c8 your@email.com
    The key's randomart image is:
    +--[ RSA 2048]----+
    |    . ...+o++++. |
    | . . . =o..      |
    |+ . . .......o o |
    |oE .   .         |
    |o =     .   S    |
    |.    +.+     .   |
    |.  oo            |
    |.  .             |
    | ..              |
    +-----------------+

(remember to replace "your@email.com" with your e-mail address).

Copy the public part of the key to the remote host
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Using you normal login password, add the public part of your key pair to
the "authorized\_keys" file on the remote host you wish to connect to
using the SSH Agent. This can be achieved by appending the contents of
the public part of the key to the remote file:

::

    -bash-4.1$ cat ~/.ssh/id_rsa.pub | ssh user@login.cirrus.ac.uk 'cat - >> ~/.ssh/authorized_keys'
    Password: [Password]

(remember to replace "user" with your username).

Now you can test that your key pair is working correctly by attempting
to connect to the remote host and run a command. You should be asked
for your key pair *passphase* (which you entered when you creasted the
key pair) rather than your remote machine *password*.

::

    -bash-4.1$ ssh user@login.cirrus.ac.uk 'date'
    Enter passphrase for key '/Home/user/.ssh/id_rsa': [Passphrase]
    Wed May  8 10:36:47 BST 2013

(remember to replace "user" with your username).

Enabling the SSH Agent
~~~~~~~~~~~~~~~~~~~~~~

So far we have just replaced the need to enter a password to access a
remote host with the need to enter a key pair passphrase. The next step
is to enable an SSH Agent on your local system so that you only have to
enter the passphrase once per day and after that you will be able to
access the remote system without entering the passphrase.

Most modern Linux distributions (and macOS) should have ssh-agent
running by default. If your system does not then you should find the
instructions for enabling it in your distribution using Google.

To add the private part of your key pair to the SSH Agent, use the
'ssh-add' command (on your local machine), you will need to enter your
passphrase one more time:

::

    -bash-4.1$ ssh-add ~/.ssh/id_rsa
    Enter passphrase for Home/user.ssh/id_rsa: [Passphrase]
    Identity added: Home/user.ssh/id_rsa (Home/user.ssh/id_rsa)

Now you can test that you can access the remote host without needing to
enter your passphrase:

::

    -bash-4.1$ ssh user@login.cirrus.ac.uk 'date'
    Warning: Permanently added the RSA host key for IP address '192.62.216.27' to the list of known hosts.
    Wed May  8 10:42:55 BST 2013

(remember to replace "user" with your username).

Adding access to other remote machines
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you have more than one remote host that you access regularly, you can
simply add the public part of your key pair to the 'authorized\_keys'
file on any hosts you wish to access by repeating step 2 above.

SSH Agent forwarding
~~~~~~~~~~~~~~~~~~~~

Now that you have enabled an SSH Agent to access remote resources you
can perform an additional configuration step that will allow you to
access all hosts that have your public key part uploaded from any host
you connect to with the SSH Agent without the need to install the
private part of the key pair anywhere except your local machine.

This increases the security of the key pair as the private part is only
stored in one place (your local machine) and makes access more
convenient (as you only need to enter your passphrase once on your local
machine to enable access between all machines that have the public part
of the key pair).

Forwarding is controlled by a configuration file located on your local
machine at ".ssh/config". Each remote site (or group of sites) can have
an entry in this file which may look something like:

::

    Host cirrus
      HostName login.cirrus.ac.uk
      User user
      ForwardAgent yes

(remember to replace "user" with your username).

The "Host cirrus" line defines a short name for the entry. In this case,
instead of typing "ssh login.cirrus.ac.uk" to access the Cirrus login
nodes, you could use "ssh cirrus" instead. The remaining lines define
the options for the "cirrus" host.

-  ``Hostname login.cirrus.ac.uk`` - defines the full address of the
   host
-  ``User username`` - defines the username to use by default for this
   host (replace "username" with your own username on the remote host)
-  ``ForwardAgent yes`` - tells SSH to forward the local SSH Agent to
   the remote host, this is the option that allows you to store the
   private part of your key on your local machine only and export the
   access to remote sites

Now you can use SSH to access Cirrus without needing to enter my
username or the full hostname every time:

::

    -bash-4.1$ ssh cirrus 'date'
    Tue Dec 20 16:48:32 GMT 2016

You can set up as many of these entries as you need in your local
configuration file. Other options are available. See the `ssh_config
man page <http://linux.die.net/man/5/ssh_config>`__ (or ``man
ssh_config`` on any machine with SSH installed) for a description of the
SSH configuration file.

