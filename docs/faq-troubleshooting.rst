Kubos FAQs and Troubleshooting Tips
===================================

.. contents:: :local:

FAQs
----

How do I contact y'all?
~~~~~~~~~~~~~~~~~~~~~~~

Our community Slack: `slack.kubos.co <https://slack.kubos.co>`__

Our email: info@kubos.co

How do I set up the Kubos SDK?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See :doc:`installation-docs/sdk-installing`

How do I check if I'm using the latest version of the Kubos Vagrant image?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From your host machine, issue the ``vagrant box outdated`` command.

How do I get the latest version of the Kubos Vagrant image?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

From your host machine, issue the ``vagrant box update`` command.

.. warning:: This will overwrite all files in your existing image

How do I check what version of the Kubos SDK I'm running?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``kubos version``

How do I update the Kubos SDK?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``kubos update --all``

How do I check what my project's target is?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

``kubos target``

How do I see the debug output of my board?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

MSP430
^^^^^^

``minicom msp430``

STM32F407
^^^^^^^^^

Connect an FTDI cable to your board's debug UART port. By default:

-  Yellow wire (TX) --> pin PC6.
-  Orange wire (RX) --> pin PC7.

Then use ``minicom kubos`` to start a serial connection session.

ISIS-OBC
^^^^^^^^

``minicom kubos``

How do I check what version of Kubos Linux I'm running?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Issue the ``uname -r`` command on the target board to display the kernel information. 
The Kubos Linux version will be the *n.n.n* number after "KubOS".
  
:: 

    uname -r    
    4.4.23-KubOS-1.0.0

What's the default Kubos Linux login?
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

kubos/Kubos123

Troubleshooting
---------------

Kubos SDK
~~~~~~~~~

The ``vagrant up`` command just hangs
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you're using Windows 7 SP1, make sure you are using the :ref:`correct version of
Windows PowerShell <powershell>`.

I've tried other steps here, but my Kubos Vagrant image is still behaving weirdly.
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Try logging out and restarting the VM using the ``vagrant reload`` command.

I got some weird errors while running a kubos command (``kubos build``, ``kubos target``, etc)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Run ``kubos update -a`` to ensure that you're using the latest version of the Kubos SDK
    
"error: could not install target {target} at * for {target}"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    error: could not install target {target} at * for {target}
    error: The targets registry does not provide a version of "{target}"
    
Run ``kubos link -a`` to re-establish the required module links for your project

"error: No module.json file."
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    error: No module.json file.
    error: The current directory does not contain a valid module.

You are not currently in a valid Kubos project directory. Alternatively, your project's `module.json` file has
somehow gotten deleted.

"error: failed to create link: [Errno 71] Protocol error"
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

You're trying to create a symlink on a Windows host machine, most likely by trying to initialize a Kubos project
in a shared folder.

Windows does not support symlinks, so you cannot build Kubos projects within a shared folder on a Windows machine.

I can't build my project. I keep getting "Permission denied" errors
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

If you copied your project from another location, it's possible that the files are set up with root permissions
only. Change the project file permissions to allow the local vagrant user to have access.

:: 
    
    sudo chown vagrant:vagrant . -R

Kubos Linux
~~~~~~~~~~~

I transferred a script, but it won't run
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``kubos flash`` preserves the file permissions of everything you transfer. Check that your file has the appropriate execute
permissions turned on.

.. _flash-troubleshooting:
    
Flash Troubleshooting
^^^^^^^^^^^^^^^^^^^^^

Flashing a file to the board can fail for various reasons. Sometimes
simply reattempting the command can correct the problem.

If retrying doesn't work, here is a list of some of the errors you might
see after running the ``kubos flash`` command and the recovery actions
you can take:

"No compatible FTDI device found"
#################################

-  Check that the Kubos Linux target is turned on and connected to your 
   computer
-  Check that no other vagrant images are running. Only one VM can have
   control of the USB, so it may be that another instance currently has
   control of the device. You can shutdown a vagrant image with the
   command ``vagrant halt``
-  Verify that the USB is showing up within the vagrant environment with
   the ``lsusb`` command. You should see an FTDI device
-  Verify that the USB has been mapped to a linux device. Issue the
   command ``ls /dev``. You should see a /dev/ttyUSB\* device. If you
   don't, try rebooting your vagrant image (``vagrant halt``,
   ``vagrant up``)

"Transfer Failed: Connection Failed"
####################################

The SDK was unable to connect to the Kubos Linux target

-  Verify that the USB has been mapped to a linux device. Issue the
   command ``ls /dev``. You should see a /dev/ttyUSB\* device. If you
   don't, try rebooting your vagrant image (``vagrant halt``,
   ``vagrant up``)
-  If this error occurs after the transfer process has started, then the
   SDK likely lost connection to the board. Verify that the board is
   still correctly connected and powered and try the flash command
   again.

"Transfer Failed: Invalid Password"
###################################

The SDK was unable to log into the Kubos Linux target. Verify that the password is
correctly defined in your config.json file by issuing the ``kubos config`` command.

System appears to have hung
###########################

-  If for some reason file transfer fails, it can take a couple minutes
   for the connection to time out and return control.
-  If you've waited a couple minutes and the system still appears hung,
   please let us know so that we can open a bug report.
   
My flash failed for some other reason
#####################################

It's possible that the transfer timed out. Check the `build/{target}/flash.log` file
in your project for more information about why the flash failed.

If you were trying to flash an upgrade file, simply re-enter the ``kubos flash``
command to resume the transfer.