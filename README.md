# eNB s1 Emulator


This is an eNB emulator application done in python3 to interact with MME (S1AP) and SGW (S1-U). 
This application can be used to perform and simulate several EMM and ESM procedures, including user plane traffic.
This application was tested with real MMEs (lab environment).

<p align="center"><img src="images/eNB.png" width="50%"></p>

You have the capability to use a modem to perform external authentication with USIM.
The modem needs to support the AT commands AT+CSIM. You can define the ttyUSBx to use in the options (-u).

External authentication with USIM can also be performed with a Smartcard reader, or even through https (see https://github.com/fasferraz/USIM-https-server)

Note: If none of these options is defined, a default KASME and XRES are used (check corresponding variables inside *session_dict_initialization* function).



I had previously done some experiments with SCTP using the native socket module from python3, but starting a ASN.1 module for S1AP from scratch was a big challenge. Fortunately I found some magnificent python modules for ASN.1 and S1AP done by P1 Security that I highly recommend:

https://github.com/P1sec

They have plenty of modules for almost anything related to Mobile Developments, but for my project i just used the S1AP from pycrate_asn1dir module, and CM from the CryptoMobile module (that has all the ciphering and integrity protocols needed for NAS). 
In order to derive the integrity and ciphering keys from KASME/CK/IK) i used another module: the Crypto.Hash (pip3 install pycryptodome) that has the HMAC and SHA256 functions needed for KDF.

For serial communication towards the modem I use the pyserial module (pip3 install pyserial)

For smartcard reader communication I use the smartcard module (pyscard module from https://pypi.org/project/pyscard/) and card module (from https://github.com/mitshell/card).

For https request to https server I use the requests module.


So in resume, these are the required external (non-standard) modules:

```
from pycrate_asn1dir import S1AP
from pycrate_asn1rt.utils import *
from CryptoMobile.CM import *
from CryptoMobile.Milenage import Milenage
from Crypto.Hash import HMAC
from Crypto.Hash import SHA256
```

For IMSI, and authentication these are the required external (non-standard) modules:

(at least one of these options should be available if we need to use real USIM)

```
try:
   import serial
except:
    pass
try:
    from smartcard.System import readers
    from smartcard.util import toHexString,toBytes
    try:
        from card.USIM import *
    except:
        pass
except:
    pass
try:    
    import requests
    requests.packages.urllib3.disable_warnings()
except:
    pass
    
```

For authentication the application also accepts Ki and OP/OPC for Milenage operation (usefull for testing with developments like open5gs, where the USIM parameters are defined in the HSS/UDR).


Many variables needed for SA1P and NAS are defined inside the *session_dict_initialization* function. 
You can change them to meet your own needs.

When you call the scritp these are the options available:

```
python3 eNB_LOCAL.py -h
Usage: eNB_LOCAL.py [options]

Options:
  -h, --help            show this help message and exit
  -i ENB_IP, --ip=ENB_IP
                        eNB Local IP Address
  -m MME_IP, --mme=MME_IP
                        MME IP Address
  -g GATEWAY_IP_ADDRESS, --gateway_ip_address=GATEWAY_IP_ADDRESS
                        gateway IP address
  -u SERIAL_INTERFACE, --usb_device=SERIAL_INTERFACE
                        modem port (i.e. COMX, or /dev/ttyUSBX), smartcard
                        reader index (0, 1, 2, ...), or server for https
  -I IMSI, --imsi=IMSI  IMSI (15 digits)
  -E IMEI, --imei=IMEI  IMEI-SV (16 digits)
  -K KI, --ki=KI        ki for Milenage (if not using option -u)
  -P OP, --op=OP        op for Milenage (if not using option -u)
  -C OPC, --opc=OPC     opc for Milenage (if not using option -u)
  -o PLMN, --operator=PLMN
                        Operator MCC+MNC
  
  ```
 
Note: Gateway IP Address (option -g) is needed when the MME or SGW are not in the local LAN. With user plane activated, the default route points to a tunnel interface, so this Gateway IP Address is needed so that MME address and SGW address are also reachable (using /32 routes) using this IP as next-hop address. In case of multiple interfaces, this IP address must be in the same network as the source interface used for eNB address (option -i).


Example usage fo eNB address - 172.16.168.130, and MME address - 172.16.168.8 (eNB and MME in the same LAN), and ttyUSB2:

```python3 eNB_LOCAL.py -i 172.16.168.130 -m 172.16.168.8 -u /dev/ttyUSB2```


This is the application user interface, where we can see the current options and procedures supported:
  
  <p align="center"><img src="images/application.png" width="100%"></p>
  
This application implements S1-U, so after an successful Attach with PDN Connectivity activation, you can use the laptop applications (browser, terminal, etc...) to send/receive traffic over the GTP-U connection towards the SGW using the session IP address, using a tunnel interface.

In case the session is a NB-IoT session you can also send the user plane over NAS.

The basic flow could be for example, option 15 - to bring up the s1 interface, and then option 20 - to perform attach.


The application supports currently the following options:

- S1 Setup type: LTE, NB-IoT, or both
- Mobile Identity Type: IMSI or GUTI
- Attach PDN: Default APN, or Specific APN
- Session Type: 4G, 5G or NB-IoT
- Session Sub-Type: No PSM and No eDRX, PSM, eDRX or both PSM and eDRX
- PDN Type ipv4, ipv6 or ipv4v6
- Control Plane Service Request with Radio Bearer or without Radio Bearer
- Attach Type: EPS Attach or Combined EPS/IMSI Attach
- TAU Type: TA Updating, Combined TA/LA Updating or Combined TA/LA Updating with IMSI Attach
- Process Paging: Enabled or Disabled
- SMS Update type: Additional Update Type SMS Only: False or True
- eNB Cell and TAC can change

In terms of procedures, the application supports the following ones:

- S1 Setup Request
- S1 Reset
- Attach
- Detach
- TAU
- TAU Periodic
- Service Request
- UE Context Release
- Send SMS (a predefined one)
- Control Plane Service Request
- E-RAB Modification Indication
- Secondary RAT Data Usage Report
- PDP Connectivity
- PDN Disconnect
- Activate/Deactivate GTP-U for Control Plane
- Activate/Deactivate Data over NAS
  
  
  
  
You can also find more information in https://fabricioapps.blogspot.com/2020/07/mme-part-i-enb-emulator.html

# Update 
Application now also supports:

- Emergency Attach (option 8)
- EPS Mobility Identity can also be IMEI (for Emergency Attach. Option 2)
- PDN Connectivity Request with Request Type = "Emergency" (Attach normally, then set Attach Type to "EPS Emergency Attach", and when chosing option 40, Request Type will be equal to "Emergency")
