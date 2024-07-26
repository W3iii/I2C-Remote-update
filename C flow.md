---
title: C flow

---

C flow
===
下面描述成簡易flow
> UI
> main.c
>> XO2_api.c
>>>XO2_cmds.c  

==main.c== call ==XO2ECA_apiProgram()== call ==XO2_api.c== function

UI介面
---
![UI](https://hackmd.io/_uploads/S1DQVg6uA.png)


main.c (基本架構並不包含所有程式碼)
---
```c=
#include "ufm_0123.h"
#include "ufm_DEAD.h"


#define UFM_START_PAGE 256   // Start here for user data (beginning is EBR init vals)

const char *JEDECDesignList[3] = 
{
    "???",
    "ufm_0123(number pattern)",
    "ufm_DEAD(word pattern)"
};

int main()
{
    if (display)
    {
        // 顯示ui介面
    }
    
    switch (selection)
    {
        case 'P':
            // CALL XO2_api.c 函式 XO2ECA_apiProgram()
            // 此函式包含所有I2C flow !!!!
        case 'V':
            // 選是否要Verify
        case '1':
            // 選讀入的JEDEC檔
        case '2':
            // 選讀入的JEDEC檔
    }
}
```
Case
 - P: 基本上包含所有flow
 - v: 要驗證就選、不要就不用選此流程
 - 1、2:有宣告路徑在int main之上，選擇哪個JEDEC檔 

XO2_api.h
---
```c=
/*
 *  COPYRIGHT (c) 2011 by Lattice Semiconductor Corporation
 *
 * All rights reserved. All use of this software and documentation is
 * subject to the License Agreement located in the file LICENSE.
 */

/** @file XO2_api.h */
 
#ifndef LATTICE_XO2_API_H
#define LATTICE_XO2_API_H

#define XO2ECA_PROGRAM_TRANSPARENT 0x10 // program in Background, user logic runs while doing it
#define XO2ECA_PROGRAM_OFFLINE     0x00 // program in Direct mode, user logic halts
#define XO2ECA_PROGRAM_VERIFY	   0x20 // Verify Programming of any of the above modes
```
定義==Program mode variable==
 - 在Verify case時就會設定 ==VerifyMode = XO2ECA_PROGRAM_VERIFY==;

```c=

#define XO2ECA_ERASE_PROG_UFM      0x08 // Erase/program UFM sector
#define XO2ECA_ERASE_PROG_CFG      0x04 // Erase and program CFG sector
#define XO2ECA_ERASE_PROG_FEATROW  0x02 // Erase/program Feature Row
#define XO2ECA_ERASE_SRAM          0x01 // Erase SRAM (used in Offline mode)

#define NOT_IMPLEMENTED_ERR   (-1000)

int XO2ECA_apiProgram(XO2Handle_t *pXO2dev, XO2_JEDEC_t *pProgJED, int mode);

int XO2ECA_apiClearXO2(XO2Handle_t *pXO2dev);

int XO2ECA_apiEraseFlash(XO2Handle_t *pXO2dev,  int mode);

void XO2ECA_apiJEDECinfo(XO2Handle_t *pXO2dev, XO2_JEDEC_t *pProgJED);

int XO2ECA_apiJEDECverify(XO2Handle_t *pXO2dev, XO2_JEDEC_t *pProgJED);


int XO2ECA_apiReadBackCfg(XO2Handle_t *pXO2dev, unsigned char *pBuf);


int XO2ECA_apiReadBackUFM(XO2Handle_t *pXO2dev, int startPg, int numPgs, unsigned char *pBuf);


int XO2ECA_apiWriteUFM(XO2Handle_t *pXO2dev, int startPg, int numPgs, unsigned char *pBuf, int erase);

int XO2ECA_apiGetHdwStatus(XO2Handle_t *pXO2dev, unsigned int *pVal);


int XO2ECA_apiGetHdwInfo(XO2Handle_t *pXO2dev, XO2RegInfo_t *pInfo);


#endif
```
定義是否==erace variable==以及所有 XO2ECA_apiProgram() 會call到的functions

Function XO2ECA_apiProgram()
---
```c=
#include "XO2_cmds.h"
//此為最細函式 詳細內容參考XO2_cmds.h

int XO2ECA_apiProgram()
{
    XO2ECAcmd_openCfgIF();
    XO2ECAcmd_EraseFlash();
    XO2ECAcmd_CfgResetAddr();
    XO2ECAcmd_CfgWritePage();
    XO2ECAcmd_CfgResetAddr();
    XO2ECAcmd_CfgReadPage();
    XO2ECAcmd_UFMResetAddr();
    XO2ECAcmd_UFMWritePage();
    XO2ECAcmd_UFMResetAddr();
    XO2ECAcmd_UFMReadPage();
    XO2ECAcmd_FeatureRowWrite();
    XO2ECAcmd_FeatureRowRead();
    XO2ECAcmd_setDone();
    
    while (i && (XO2ECAcmd_Refresh(pXO2dev) != OK))
    {
    	--i;
    }

PROG_ABORT:
    XO2ECAcmd_closeCfgIF(pXO2dev); 
    XO2ECAcmd_Bypass(pXO2dev); 
    return(ret);
}
```
## XO2_cmds.c 有詳細函式內容

XO2ECAcmd_openCfgIF();
 - 276行
![74](https://hackmd.io/_uploads/SkJNCy6OC.png)


XO2ECAcmd_EraseFlash();
 - 894行
![0E](https://hackmd.io/_uploads/SkzX0yadC.png)


XO2ECAcmd_CfgResetAddr();
 - 969行
![46](https://hackmd.io/_uploads/BJ3GR16uC.png)


XO2ECAcmd_CfgWritePage();
 - 1101行
![70](https://hackmd.io/_uploads/BkUbCyadA.png)

XO2ECAcmd_CfgReadPage();
 - 1030行
![73](https://hackmd.io/_uploads/rkobCypuC.png)

XO2ECAcmd_UFMWritePage();
 - 1337行
![C9](https://hackmd.io/_uploads/Hyzx0JTO0.png)

XO2ECAcmd_UFMReadPage();
 - 1261行
![CA](https://hackmd.io/_uploads/Bye-CJT_A.png)

XO2ECAcmd_FeatureRowWrite();
 - 1474行
![E4](https://hackmd.io/_uploads/Sy1i6JpuC.png)

XO2ECAcmd_FeatureRowRead();
 - 1562行
![E7](https://hackmd.io/_uploads/Hykq61auR.png)

XO2ECAcmd_setDone();
 - 449行
![5E](https://hackmd.io/_uploads/SyxP6yp_R.png)

XO2ECAcmd_Refresh();
 - 388行
![79](https://hackmd.io/_uploads/rkGIp1pOA.png)

