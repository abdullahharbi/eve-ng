# EVE-NG NXOS K9 VPC  used mangmant Configuration

تطبيق عملى لبروتكول VPC على  K9

![image](https://github.com/user-attachments/assets/9f7e7a27-75c9-4a52-b2ea-33a9867411a6)



## إعدادات الأجهزة
### إعدادات Switch 2 (sw2)
```bash
conf t
hostname sw2

int range e0/0-1
  switchport trunk encapsulation dot1q
  switchport mode trunk
  duplex full
  no shutdown
  channel-group 2 mode passive

sh run

interface Port-channel2
  switchport trunk encapsulation dot1q
  switchport mode trunk

wr
```



### إعدادات Switch 3 (sw3)
```bash


conf t
hostname sw3

int range 0/0-1
  switchport trunk encapsulation dot1q
  switchport mode trunk
  duplex full
  no shutdown
  channel-group 3 mode passive

sh run

interface Port-channel3
  switchport trunk encapsulation dot1q
  switchport mode trunk

interface Ethernet0/3
  switchport trunk encapsulation dot1q
  switchport mode trunk

wr
```

اعدادات يجب عملها على جميع اجهزة NXOS اول مره فقط 



continue with Power On Auto Provisioning] (yes/skip/no)[no]:   yes

هذا الامر لكي لا تضطر الى ادخال كلمة سر معقدة

Do you want to enforce secure password standard (yes/no) [y]: no

Enter the password for "admin": admin\
Confirm the password for "admin": admin\
Would you like to enter the basic configuration dialog (yes/no): no

login: admin\
Password: admin
```
switch# dir
```
سوف تظهر لك هذه المسارات فقط يهمك هذا المسار الذي يحدد ملف الاقلاع 
```
964875776    Jun 14 10:01:57 2018  nxos.7.0.3.I7.4.bin
```

4096    Oct 11 06:47:12 2024  .rpmstore/\
       4096    Oct 11 06:47:33 2024  .swtam/\
     671311    Oct 11 07:34:10 2024  20241011_064913_poap_30659_1.log\
    1048622    Oct 11 07:16:25 2024  20241011_064913_poap_30659_init.log\
          0    Oct 11 06:47:22 2024  bootflash_sync_list\
  964875776    Jun 14 10:01:57 2018  nxos.7.0.3.I7.4.bin\
          0    Oct 11 07:34:49 2024  platform-sdk.cmd\
       4096    Oct 11 06:49:07 2024  scripts/\
       4096    Oct 11 06:49:07 2024  virt_strg_pool_bf_vdc_1/\
       4096    Oct 11 06:47:59 2024  virtual-instance/\
         59    Oct 11 06:47:49 2024  virtual-instance.conf\

اكتب هذا الاوامر لتحديد الملف الذي سوف يتم الاقلاع منه و من ثم حفظ التغيرات
```
switch(config)# boot nxos bootflash: nxos.7.0.3.I7.4.bin

switch(config)# copy running-config startup-config
```
سوف يظهر لك هذا العداد الذي يوضح ان عملية النسخ اكتملة


[########################################] 100%

بعد هذا تستطيع تهيئة k9 

المهمة الاولى هي كيفية تفعيل peer-keepalive بين اجهزة NXOS



NXOS-1
```
conf t
hostname NXOS-1
feature lacp
feature vpc
vpc domain 1
  role priority 10
  peer-keepalive destination 10.10.10.2 source 10.10.10.1 vrf management

interface mgmt0
  ip address 10.10.10.1/24

```
NXOS-2
```
conf t
hostname NXOS-2
feature lacp
feature vpc
vpc domain 1
  role priority 20
  peer-keepalive destination 10.10.10.1 source 10.10.10.2 vrf management

interface mgmt0
  ip address 10.10.10.2/24
```
هذه الاوامر كفيلة بتفعيل peer-keepalive

![image](https://github.com/user-attachments/assets/6df44f25-57d6-4858-9573-570279c6e34a)

المهمة الثانية هي  كيفية تفعيل Peer status 

NXOS-1
```
interface e1/1-2
  switchport mode trunk
  channel-group 1 mode active
  no shutdown
int po1
  vpc peer-link
  no shutdown

```
NXOS-2
```
interface e1/1-2
  switchport mode trunk
  channel-group 1 mode active
  no shutdown
int po1
  vpc peer-link
  no shutdown

```
هذه الاوامر كفيلة بتفعيل Peer status 

![image](https://github.com/user-attachments/assets/cce870d1-9c67-4aaa-a483-542b6a6e5132)


