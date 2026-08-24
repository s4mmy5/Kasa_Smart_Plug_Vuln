 Basic Information
- Name: Kasa Smart Wi-Fi Plug Mini
- Model: EP10P2
- S/N: Y264328X03277
- P/N: 0184600821
- Wifi: 802.11b/g/n
- FCC ID: 2AXJ4EP10
- IC ID: 26583-EP10

![Image of Kasa Smart Wi-Fi Plug Mini as one of the first search results on Amazon](./assets/Amazon_Listing.png)

# FCC ID Information
- FCC ID: 2AXJ4EP10
- Filing: https://fccid.io/2AXJ4EP10

# Internals
SoC: RTL8710CF

## UART Pins
Power pins are P1, P2, P3.
TP1, TP2, TP3, TP4, TP5, TP6 pins are also available.

Ground: TP1, P3

Voltages:
- P1: 1.9 V
- P2: 0 V
- P3: Ground
- TP1: Ground
- TP2: 4.5 V
- TP3: 1.6 V
- TP4: 4.9 V
- TP5: 1.6 V
- TP6: 1.6 V

# API Prodding
 
Request: 
```
GET /v2/things?page=0&pageSize=500&deviceTypes=IOT.SMARTPLUGSWITCH%2CIOT.SMARTBULB%2CIOT.IPCAMERA%2CIOT.HUB%2CIOT.RANGEEXTENDER%2CIOT.RANGEEXTENDER.SMARTPLUG%2CIOT.ROUTER%2CSMART.KASAHUB%2CSMART.KASAENERGY%2CSMART.KASAPLUG%2CSMART.KASASWITCH%2CSMART.TAPOSENSOR
```

Response:
```json
{
  "thingName": "800697671ADBD01235F778F911F13ABE25A05801",
  "appServerUrl": "https://use1-app-server.iot.i.tplinknbu.com",
  "appServerUrlV2": "https://use1-app-server.iot.i.tplinkcloud.com",
  "pcAppServerUrl": "https://n-use1-wap.tplinkcloud.com",
  "pcAppServerUrlV2": "https://n-use1-wap.i.tplinkcloud.com",
  "status": 1,
  "thingModelId": "kasa:plug:v2",
  "role": 0,
  "authType": 0,
  "shareType": "",
  "familyId": "default1",
  "roomId": "3QV0771Y",
  "commonDevice": true,
  "commonDeviceV2": false,
  "nickname": "SFVNUFRZRFVN",
  "avatarUrl": "",
  "onboardingTime": 1786735691,
  "category": "plug",
  "model": "EP10(US)",
  "crossRegion": true,
  "deviceName": "Smart Wi-Fi Plug Mini",
  "deviceType": "IOT.SMARTPLUGSWITCH",
  "oemId": "41372DE62C896B2C0E93C20D70B62DDB",
  "hwId": "AE6865C67F6A54B756C0B5812472C825",
  "hwVer": "1.0",
  "fwVer": "1.0.5 Build 221021 Rel.183404",
  "fwId": "00000000000000000000000000000000",
  "region": "America/New_York",
  "mac": "58D81223C8BA",
  "isSubThing": false
}
```

## Local Communication

Turns out the plug and app communicate locally with an in-house protocol, credits to [this person](https://www.softscheck.com/en/blog/tp-link-reverse-engineering/#1-security-analysis-summary) for writing a wireshark dissector for it!

A few of the available endpoints are:
- get_sysinfo
- cnCloud
- download_firmware
- get_download_state

### Firmware Update routine

Since I did not have the tools to directly dump the firmware I proceeded to look for a binary in tp-link's ![download center](https://www.tp-link.com/us/support/download/ep10/v1/). Unlike the guy in the previous blog my board gave no indication of embedded linux and this was confirmed by TP-LINK as I was not entitled to any of that sweet GPL licensed code in their. Thus, my next logical option was to capture a firmware update routine. I cliked the update button and observed the following:

1. App polls for correct firmware version and receives an HTTPS download link.
Request:
```
https://n-use1-wap.i.tplinkcloud.com/api/v2/common/getSecureFwList?token=ff131fff-ATGKqaOuoqaQLqIOhKFpcDf&appName=Kasa_Android_Mix&appVer=3.4.602&termID=09F7A5BF0F0300423B878FD64381AAAC&ospf=Android+15&netType=wifi&locale=en_US&brand=TPLINK&model=Pixel+8a&termName=Google+Pixel+8a&termMeta=1
```

Response
```json
{
    "msg": "Success",
    "result": {
        "fwList": [
            {
                "fwVer": "1.1.1 Build 250908 Rel.112508",
                "fwReleaseDate": "2026-02-28",
                "fwTitle": "Hi, a new firmware with performance improvement is available for your EP10.",
                "fwReleaseLog": "Modifications and Bug Fixes:\n1. Improved time synchronization accuracy.\n2. Enhanced device local communication security.\n3. Enhanced device stability.",
                "fwSecureUrl": "https://ota-download.tplinkcloud.com/firmware/EP10_FCC_1.1.1_Build_250908_Rel.112508_2025-09-08_11.26.29_1772244511144.bin",
                "fwReleaseLogUrl": "undefined yet",
                "fwLocation": 0,
                "fwType": 1,
                "fwMd5": "7E4679D311ACEC1DBABB65884355082B",
                "status": "Normal",
                "isBeta": "N",
                "isRevoked": "N"
            }
        ]
    },
    "error_code": 0
}
```

2. The app locally communicates a (downgraded) HTTP download link to the plug.
Rquest: 
```json
{
  "context": {
    "source": "46a4d58b-6279-432c-ae23-e115c2db8354"
  },
  "system": {
    "download_firmware": {
      "url": "http://ota-download.tplinkcloud.com/firmware/EP10_FCC_1.1.1_Build_250908_Rel.112508_2025-09-08_11.26.29_1772244511144.bin"
    }
  }
}
```
### Note: Authentication
WIP. Accessing the firmware site directly returns an Access Denied response. It could be an interesting side quest to test if impersonation of the plug is possible to download other firmware versions.

Response:
```json
{
  "system": {
    "download_firmware": {
      "err_code": 0
    }
  }
}
```

3. There are some other less interesting status request in the middle to check the update progress as a percentage. Using the `get_download_state` command.

## Firmware Static Analysis

WIP. Ghidra is able to identify some interrupt handlers but there's no section data available nor any partition information. I also haven't figured out if it's possible to emulate the dumped firmware given that the update code likely implements it's own routines to update the SoC. 


# Next Steps
- Statically analyze firmware sample to see if there are any interesting routines or discover how the update is applied. This could lead to some cool device takeover exploits.
- Check the security of the firmware update endpoint. I wonder what authentication protocol is being used.
- TPs seem very inconsistent, logic analyzer is needed to decode protocol.
    - Using logic analyzer while plug is connected is risky. I will need to desolder wifi board first and power with 3.3 V power supply instead.
