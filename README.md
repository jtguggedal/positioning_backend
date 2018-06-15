# Indoor positioning system

This repository contains files to be used for indoor positioning platform based on Nordic Semiconductor's PCA20036 nodes. 

For now, only the very basics are covered in this readme, and more detail will be added at a later time.

## Requirements
- The positioning platform
  - Nodes: PCA20036, firmware [here](https://github.com/jtguggedal/positioning_firmware/tree/master/node)
  - Tags: PCA10056, firmware [here](https://github.com/jtguggedal/positioning_firmware/tree/master/tag)
  - Switch: PoE enabled Ethernet switch
- Software
  - Python 3, tested with v3.6.4 64-bit
  - MySQL database, tested with v5.7.21
  - PacketSender to send commands to the nodes, download [here](https://packetsender.com/)


## Setup 
- Physical setup
  - Connect the nodes to the switch's PoE enabled ports
  - Connect the computer to the switch
  - Optionally connect the time synchronization lines to the same external time synchronization source or let one of the nodes control the time sync signal
- Software
  - Install Python 3
  - Data processing
    - Run `pip install -r processing/requirements.txt` to install required Python packages
  - Database
    - See the `.sql` files in the [`db`  folder](https://github.com/jtguggedal/positioning_backend/tree/master/db/) folder for database and table configuration examples



## Testing
### Receive data from nodes

To test if receiveing data from the nodes work, do the following steps after the setup is completed:
- Find the computer's IP address given by the switch (in the 10.0.0.x address space)
- In a terminal, run `python node_listener.py --ip <IP address>`
- Confirm that the nodes' high-power LEDs light up after a few seconds, indicating that they have received the computer's IP and port number, and are ready to track tags
- Switch on a tag
- See incoming scan reports in the terminal
- Optionally enable database storing of data
  - In `node_listener.py`, set `DB_ENABLED = True` 
  - Enter the preferred table name: `DB_TABLE_NAME = xx` 
  - Insert the correct database credentials in the `db.connect()` function call


### Apply positioning algorithm
Perform the following steps to apply a positioning algorithm to data stored in a database
- Run the SQL query in `db/create_table_positioning.sql` to create a demo table 
- Run the SQL queries in `db/positioning_test_dump.sql` to populate the demo table with real test data
- In a terminal, run `python processing/analysis/position_estimation.py`
- Data will be fetched from the database, and after a while a graphical representation of the indoor environment will be shown. 
  - The red dots are nodes, the green es the tags true position
  - The black dot is position estimate using raw RSSI values
  - The blue is the estimate using Kalman filtered RSSI values
  - The yellow dot is the position estimate using both Kalman filtered RSSI and position

## Control the nodes
The nodes have a command system installed. By sending commands to the nodes, it's possible to remotely control for instance their high-power LEDs, make a node time sync controller, change the access address for the radio, send a new IP and port for the computer, and make the nodes send who-am-I-messages. 

To send command messages, import predefined commands from `commands/packetsender_commands.ini` into PacketSender. From PacketSender, each command can be modified and sent. 

The commands are strcutured as follows
- ASCII string prefix `CONTROL_COMMAND:` always starts a command message
- The following byte is the command type, followed by a payload length byte
- The subsequent bytes are the payload

An example of a command is to set a node as sync node. The command may look as follows in PacketSender's ASCII view: `CONTROL_COMMAND:F\04\n\00\00\0c` and `43 4f 4e 54 52 4f 4c 5f 43 4f 4d 4d 41 4e 44 3a 46 04 0a 00 00 0c` in HEX representation. The last 4 bytes are the IP address of the node that will function as time sync controller: `10.0.0.12`. Change the last byte to set any other node as controller.