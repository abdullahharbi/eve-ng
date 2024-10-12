# EVE-NG NXOS K9 VPC  used management or interface or vlan 

تطبيق عملى لبروتكول VPC على  K9 و k7 

![image](https://github.com/user-attachments/assets/9f7e7a27-75c9-4a52-b2ea-33a9867411a6)




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

vlan 10

interface Ethernet0/2
  switchport access vlan 10
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

vlan 10

interface Ethernet0/2
  switchport access vlan 10

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
سوف تظهر لك مجموعة من المسارات فقط يهمك هذا المسار الذي يحدد ملف الاقلاع 
```
964875776    Jun 14 10:01:57 2018  nxos.7.0.3.I7.4.bin
```

اكتب هذه الاوامر لتحديد الملف الذي سوف يتم الاقلاع منه و من ثم احفظ التغيرات
```
switch(config)# boot nxos bootflash: nxos.7.0.3.I7.4.bin

switch(config)# copy running-config startup-config
```
سوف يظهر لك هذا العداد الذي يوضح ان عملية النسخ اكتملة


[########################################] 100%

بعد هذا تستطيع تهيئة k9 \
******** تنبية مهم جدا جدا  ***********\
في المهمة الاولى يتم فقط تحديد نوع الاتصال بين أجهزة NXOS اما management او interface او vlan  اما باقي المهام تبقى كما هي بدون تغيير\
المهمة الاولى هي كيفية تفعيل peer-keepalive بين اجهزة NXOS بستخدام management



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

vlan 10


```
المهمة الاولى هي كيفية تفعيل peer-keepalive بين اجهزة NXOS بستخدام interface

NXOS-1
```
conf t
hostname NXOS-1
feature lacp
feature vpc
vrf context AAA
vpc domain 1
  role priority 10
  peer-keepalive destination 10.10.10.2 source 10.10.10.1 vrf AAA

interface Ethernet1/3
  no switchport
  vrf member AAA
  ip address 10.10.10.1/24
  no shutdown

```
NXOS-2
```
conf t
hostname NXOS-2
feature lacp
feature vpc
vrf context AAA
vpc domain 1
  role priority 20
  peer-keepalive destination 10.10.10.1 source 10.10.10.2 vrf AAA

interface Ethernet1/3
  no switchport
  vrf member AAA
  ip address 10.10.10.2/24
  no shutdown

```
المهمة الاولى هي كيفية تفعيل peer-keepalive بين اجهزة NXOS بستخدام vlan

NXOS-1
```
conf t
hostname NXOS-1
feature lacp
feature vpc
feature interface-vlan
vrf context BBB
vlan 10
vpc domain 1
  role priority 10
  peer-keepalive destination 10.10.10.2 source 10.10.10.1 vrf BBB

interface vlan 5
  vrf member BBB
  ip address 10.10.10.1/24
  no shutdown

interface Ethernet1/3
  switchport access vlan 10


```
NXOS-2
```
conf t
hostname NXOS-2
feature lacp
feature vpc
feature interface-vlan
vrf context BBB
vlan 10
vpc domain 1
  role priority 20
  peer-keepalive destination 10.10.10.1 source 10.10.10.2 vrf BBB

interface vlan 5
  vrf member BBB
  ip address 10.10.10.2/24
  no shutdown

interface Ethernet1/3
  switchport access vlan 10

```



هذه الاوامر كفيلة بتفعيل peer-keepalive

![image](https://github.com/user-attachments/assets/6df44f25-57d6-4858-9573-570279c6e34a)

المهمة الثانية هي  كيفية تفعيل peer-link status 

NXOS-1
```
interface e1/1-2
  switchport mode trunk
  channel-group 1 mode active
  no shutdown
int po1
  vpc peer-link
  no shutdown

vlan 10

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

vlan 10

```
هذه الاوامر كفيلة بتفعيل peer-link status \
امر vlan ليس له علاقة في تشغيل peer-link status  سوف يعمل بدونه ولكن في حالة طلب منك ان تعمل على فيلان رقم 10 

![image](https://github.com/user-attachments/assets/cce870d1-9c67-4aaa-a483-542b6a6e5132)


المهمة الثالثة هي  كيفية تفعيل channel-group بين السويتشات و NXOS K9
يجب الانتباه الى رقم interface  فقد لا يعمل بسبب اختلاف ارقام interface

NXOS-1
```
interface Ethernet1/6
  switchport mode trunk
  channel-group 2 mode active
  no shutdown

interface Ethernet1/7
  switchport mode trunk
  channel-group 3 mode active
  no shutdown

int po2
  vpc 2
  no shutdown

int po3
  vpc 3
  no shutdown

```
NXOS-2
```
interface Ethernet1/6
  switchport mode trunk
  channel-group 3 mode active
  no shutdown

interface Ethernet1/7
  switchport mode trunk
  channel-group 2 mode active
  no shutdown

int po2
  vpc 2
  no shutdown

int po3
  vpc 3
  no shutdown


```
هذه الاوامر كفيلة بتفعيل channel-group بين السويتشات و NXOS K9

![image](https://github.com/user-attachments/assets/d7350693-b6bd-4f3e-87eb-c92c77d9a44e)

الخطوه الاخيره تكوين ip لاجهزة PCs وعمل ping لتاكد من ان جميع الاعدادت صحيحة

PC-5
```
ip 192.168.1.1/24
save

```
PC-6
```
ip 192.168.1.2/24
save
```
هذه الاوامر كفيلة بأن يكون هناك ping بين اجهزة PCs

![image](https://github.com/user-attachments/assets/8aa65091-ae00-4bd4-810d-443a4bb244ad)

 حفظ الاعدادات على NXOS 1 and 2
```
copy running-config startup-config

```
 حفظ الاعدادات على SW-2 and SW-3

```
wr
```

###########################################################################################################

من هنا يبداء اعداد titanium - k7


اوامر يجب تنفيذها على جميع سويتشات  k7  يفضل التسلسل
```
license grace-period
install feature-set fabricpath
feature-set fabricpath
vlan 1-200
  mode fabricpath
  exit


```

اوامر يجب تنفيذه على جميع سويتشات k7  مع تغير الرقم على كل سويتش 
```
fabricpath switch-id 1 

fabricpath domain default
  root-priority 254

```
يجب ان يكون لكل سويتش switch-id مختلف مثلا الثاني يكون switch-id 2 و الثالث يكون switch-id 3 \
يجب ان يكون لكل سويتش  root-priority مختلف مثلا اعلى اولويه يكون root-priority 254 و الذي يلية في الالولوية يكون root-priority 253 والذي يلية يكون root-priority 252 وهكذا 


في المرحلة التالية  تطبيق هذه الاوامر على جميع سويتشات k7
```
interface Ethernet2/1-6
  switchport
  switchport mode fabricpath
  no shutdown


interface Ethernet2/7
  switchport
  switchport access vlan 10
  no shutdown
```
في اخر مرحلة تطبيق trank على interface الذي يربط بين k9 و k7  والموجود حسب هذا المخطط في NXOS-7
```
interface Ethernet2/8
  switchport
  switchport mode trunk
  no shutdown

```

 ومن ثم اعداد اجهزة الحاسب PCs

 PC-5
```
ip 192.168.1.3/24
save
```
PC-6
```
ip 192.168.1.4/24
save
```
PC-7
```
ip 192.168.1.5/24
save
```
PC-8
```
ip 192.168.1.6/24
save
```

حفظ جميع الاعدادات 
```
copy running-config startup-config
```

بتوفيق للجميع و لا تنسونا من صالح دعائكم 
