# 20151127


#### exercise 6
- serial 로 데이터를 수신하고 openTSDB에 저장

- 윈도우에서 USB 드라이버 설치 : http://125.7.128.52:8001/wordpress/pub/inhatc/CDM_Setup.exe
- serial 사용을 위한 python 라이브러리 설치
```sh

# pyserial install
wget http://sourceforge.net/projects/pyserial/files/pyserial/2.7/pyserial-2.7.tar.gz
tar xvfz pyserial-2.7.tar.gz
cd pyserial-2.7
python setup.py install

vim serial_to_openTSDB.py
```

```sh

#!/usr/bin/python

import time
import os
import sys
import serial

packet =''

def bigEndian(s):
        res = 0
        while len(s):
                s2 = s[0:2]
                s = s[2:]
                res <<=8
                res += eval('0x' + s2)
        return res

def sese(s):

        head = s[:20]
        type = s[36:40]

        serialID = s[24:36]
        nodeID = s[55:56]
        seq = s[40:44]

        batt = s[60:64]


        if type == "0070" : # TH
                #print s
                print "battery : " , bigEndian(batt)
                temperature = bigEndian( s[64:68] )
                v1 = -39.6 + 0.01 * temperature

                t = int(time.time())
                print "temperature %d %.2f nodeid=%d" % ( t, v1, bigEndian( nodeID ) )

        else:
                #print >> sys.stderr, "Invalid type : " + type
                pass

if __name__ == '__main__':

        tmpPkt = []
        flag = 0

        test = serial.Serial("/dev/ttyUSB0", 115200)

        while 1:
                Data_in = test.read().encode('hex')

                if(Data_in == '7e'):
                        if(flag == 2) :
                                flag =0
                                tmpPkt.append(Data_in)
                                packet = ''.join(tmpPkt)

                                # send packet
                                sese(packet)

                                tmpPkt = []
                                sys.stdout.flush()
                        else :
                                flag = flag + 1
                                tmpPkt.append(Data_in)
                else :
                        if(flag == 1 and Data_in =='45') :
                                flag =2
                        tmpPkt.append(Data_in)


```
