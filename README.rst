=======
nvmemonitor
=======

Tool for monitoring host's nvme devices smart-attributes

This tool reads smart attributes which provided by nvme-cli tools from list of hosts and compare values with specified parameters in config.

Installation
------------

ensure the nvme-cli is installed on the hosts you need to monitor

- ``cd /opt``
- ``git clone https://github.com/igor-kremin/automon.git nvmemonitor``
- ``/opt/nvmemonitor/nvmemonitor --add example.com``

Upgrade
-------

- ``cd /opt/nvmemonitor``
- ``git pull``


Configuration
-------------

- ``vim /opt/nvmemonitor/nvmemonitor.conf``
- config example:

.. code-block:: none

    available_spare__less=10
    percentage_used__more=80
    temperature__more=60

    [host1.exmaple.com]
    device /dev/nvme0n1
    device /dev/nvme1n1
    percentage_used__more=90

    [host2.exmaple.com:4444]
    device /dev/nvme0n1
    device /dev/nvme1n1
    device /dev/nvme2n1

Configuration file allow comments, from symbol ``#`` to end of line.

Syntax of nvmemonitor.conf file:
list of monitored paramethers which are the same for all listed hosts.
<param>__<less|more>=<value>      ex: available_spare__less=10 - will inform you if the smart value would be less 90

``<host>[:port]`` ex:	[server2.exmaple.com:4444]
device /dev/<nvme>  ex: device /dev/nvme0n1
list of monitored paramethers for the host previously defined.
<param>__<less|more>=<value>      ex: percentage_used__more=90 - will inform you if the smart value would be grater 90 only on the server2.exmaple.com

sample output and list of available attributes from nvme-cli software 

.. code-block:: none

    critical_warning                        : 0
    temperature                             : 40 C (313 Kelvin)
    available_spare                         : 100%
    available_spare_threshold               : 10%
    percentage_used                         : 6%
    endurance group critical warning summary: 0
    data_units_read                         : 1,127,748,199
    data_units_written                      : 2,821,015,917
    host_read_commands                      : 104,667,029,322
    host_write_commands                     : 110,234,002,637
    controller_busy_time                    : 296,158
    power_cycles                            : 20
    power_on_hours                          : 24,532
    unsafe_shutdowns                        : 1
    media_errors                            : 0
    num_err_log_entries                     : 0
    Warning Temperature Time                : 0
    Critical Composite Temperature Time     : 0
    Temperature Sensor 1           : 40 C (313 Kelvin)
    Thermal Management T1 Trans Count       : 0
    Thermal Management T2 Trans Count       : 0
    Thermal Management T1 Total Time        : 0
    Thermal Management T2 Total Time        : 0

For ``temperature  attributes`` Celsius value is selected.

``alert`` command will use program ``/opt/bin/alert-via-telegram`` to send the test result to telegram
``alert-via-telegram``, thanks to Gena Makhomed, can be found here https://github.com/makhomed/automon/blob/master/bin/alert-via-telegram

``alert`` program receive one command. See source of ``/opt/automon/bin/alert-via-telegram`` program for details. 
Using ``/opt/automon/bin/alert-via-telegram`` as example you can write own alert program for sending alerts via email or SMS or via any other way.


Command line arguments
----------------------

.. code-block:: none

    nvmemonitor alert - to send smart test results to alert program 
    nvmemonitor --add <host>[:port]  adds host and his devices to config nvmemonitor.conf
    nvmemonitor --list <host>[:port] shows devices on host

Before first run
----------------

If you want to use alert to telegram you have to  to create Telegram bot and configure telegram-send script.
Detalis see in https://pypi.python.org/pypi/telegram-send documentation.

Secure Shell
------------

To work properly you need to configure promptless ssh connection to necessary hosts.
It can be done via ``ssh-keygen -t rsa`` and copy public key from ``/root/.ssh/id_rsa.pub``
to ``/root/.ssh/authorized_keys`` on monitored servers. 
Also you need to check connection with monitored server with command ``ssh example.com`` and answer ``yes`` to ssh question:

or you can use the following commands
.. code-block:: none
    ssh-keygen -q -N "" 
    ssh-copy-id host1.example.com


Automation via cron
-------------------

Ensure configuration file exists ``/opt/nvmemonitor/nvmemonitor.conf`` and define hosts to check inside it.
After it configure cron job, for example, in file ``/etc/cron.d/nvmemonitor``:

.. code-block:: none

    0 * * * * root /opt/nvmemonitor/nvmemonitor alert
