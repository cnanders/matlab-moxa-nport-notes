# matlab-moxa-nport-notes
Notes on Using Moxa NPort Serial Device Servers with MATLAB serial(), tcpip() and tcpclient()

# Operation Mode

There are two ways to use the NPort

## 1. TCP Server Mode
1. MATLAB talks to the NPort using `MATLAB.tcpip()` or `MATLAB.tcpclient()`
2. NPort talks to the hardware using serial (RS-232, RS-485)


## 2. Real COM Mode
1. Install NPort middleware software on the CPU to create a virtual COM port.
2. MATLAB talks to virtual COM port (NPort middleware) using `MATLAB.serial()` as if it was a real COM port
3. NPort middlware talks to NPort using system tcpip
4. NPort talks to serial device using serial (RS-232, RS-484) 

The rest of this docuemnt refers exclusively to operation in TCP Client Mode since

# Configuring TCP ServerMode With Web Browser

In a browser, navigate to the IP of the Moxa.  In the web-based configuration tool.  

1. Click “Operating Settings” -> “Port 1” 
2. Set the dropdown to “TCP Server”
3. Scroll to the bottom and click “Submit”

- For single-serial-port Moxas, the default tcpip port is 4001.  Matlab will neet to configure the `tcpclient` or  `tcpip` instance with this port.  
- For multi-serial-port Moxas, each serial port on the Moxa is assigned a different TCP/IP port, starting with 4001, e.g.: 4001, 4002, 4003, etc.

# Serial Settings (Baud Rate, Data Bits, etc.) on Moxa and Serial Device Must Match

In a browser, navigate to the IP of the Moxa.  In the web-based configuration tool:

1. Click “Serial Settings” -> “Port 1” (or other port) navigation item on the left
2. Configure the Moxa to communicate with the device using the parameters that were configured on the device. 

## Example: Keithley 6482

On the Keithley, 

1. Click “Menu” -> “Communication” -> “Serial”  
2. Set all serial parameters to desired values.  

In a browser, navigate to the IP of the Moxa.  In the web-based configuration tool:

1. Click “Serial Settings” -> “Port 1” (or other port) navigation item on the left
2. Configure the Moxa to communicate with the Keithley using the parameters that were configured on the hardware. Keithley hardware.

## Example: Micronix MMC 103

- The FTDI board in the MMC-103 uses a baud rate of 38400. The baud rate on the MMC-103 cannot be changed. 
- The Baud Rate of the Moxa must be configured to 38400.
- If the Baud Rate of the Moxa is not set to 38400, communication between the NPort and the FTDI board on the MMC-103 **won't work**.

# Data Packing

**When NPort receives data from the seril device, does it immediately send it out over the network?**

Serial data sent from the serial device to the NPort accumulates in the NPort’s serial buffer until one of two things happen

1. The buffer fills up to the specified `Packing Length` value.  *Note, however, that when `Packing Length` is set to zero, the NPort immediately packs serial data for network transmission.*
2. The configured delimiter character(s) are received *When this option is enabled, `Packing Length` parameter has no effect.*


After the enabled criteria is satisfied the data is packed for network transmission from the Moxa NPort to the client. 

The simplest solution is to set `Packing Length` to zero and do not bother with delimeters.  In this case, any time the serial device sends the NPort serial data, that data is immediately packed for network transmission from Moxa NPort to the MATLAB `tcpclient` instance, increasing the `BytesAvailable` property of the `tcpclient`.  

The downside of this approach is that sometimes a single “response” from the serial device is separated into multiple network packets from Moxa NPort to the MATLAB `tcpclient`, but this is not a big deal.

## Configuring Data Packing to Use Delimiters (Optional)

If you want to configure data packing to use delimiters, here is an example setup. 

- Configure the serial device to use a carriage return terminator. Recall that ASCII is a 8-bit character system (256 characters).  The base10 representation of the carriage return character is 13, which is 0x0D in hex, or 00001101 in binary.  
- Set the NPort “Operation Settings” -> Data Packing -> Delimiter 1 to “0d” (hex) and enable it.  
- Once enabled, Moxa NPort will buffer all serial data sent from the serial device (possibly over multiple transmissions) until the carriage return is received from the serial device, after which the data is packed for network transmisison. 


## Great Resource
[RS-232, RS-422, RS-485 Serial Communication General Concepts](http://www.ni.com/white-paper/11390/en/)

