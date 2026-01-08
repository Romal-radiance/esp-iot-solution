## 1. What Topics Are CONFIRMED So Far

A. BLE Stack & Platform  
✅ Bluedroid is used   
✅ Not NimBLE  
✅ Works with Android and iOS clients (protocol-wise)  
✅ Espressif provides Android reference app only  

B. OTA Architecture  
✅ BLE → Ring Buffer → OTA Task → esp_ota_write()  
✅ OTA partition selection logic understood  
✅ Full image OTA supported  
✅ Delta OTA supported   
✅ Encrypted OTA supported   

C. Firmware Flow  
✅ BLE data reception decoupled using ring buffer  
✅ OTA runs in a dedicated FreeRTOS task  
✅ Proper esp_ota_begin → write → end → set_boot_partition  
✅ Restart after successful OTA  

## 2. What Topics Are PENDING WORK  

A. Production Security  
⛔ SEC2 PROD mode not implemented, default is with none security 
⛔ Salt/verifier provisioning strategy missing  
⛔ Secure Boot + Flash Encryption not validated with BLE OTA  

B. Mobile Side  
⛔ iOS client not implemented  
⛔ OTA resume after disconnect not validated  
⛔ Power loss mid-OTA behavior not tested  
⛔ Rollback validation after crash not verified  


## Note
In application later we need to do configuration of this ble_ota folder, because it is not get compiled stand alone it 
requires entire esp-iot-solution-master to get compiled.