# 🇮🇷 فارسی – شبکه و SSH با MobaXterm

[← Workfolio](../../../README.md)


> **هدف:** راه‌اندازی شبکه Gentoo LiveCD روی ماشین مجازی Proxmox و اتصال به آن از طریق SSH و MobaXterm.

## پروژه مرتبط
[🐧 Gentoo Linux auf Proxmox](../README.md)
## اطلاعات موردنیاز
- IP ثابت: `192.168.50.156`
- Prefix: `/24`
- Subnet Mask: `255.255.255.0`
- Gateway: `192.168.50.250`
- کارت شبکه: `ens18`
- پورت SSH: `22`

> رمز عبور را در Notion یا اسکرین‌شات ذخیره نکن. قبل از تکرار کار مطمئن شو IP هنوز برای همین ماشین رزرو شده است.

## ۱. اجرای VM و LiveCD
1. ماشین Gentoo را در Proxmox باز کن.
2. بررسی کن Network Device به Bridge خارجی درست متصل باشد و گزینه **Link down** فعال نباشد.
3. Gentoo Minimal Installation CD را اجرا کن.
4. منتظر بمان تا Prompt به‌شکل `livecd ~ #` ظاهر شود. LiveCD به‌صورت خودکار با کاربر `root` وارد می‌شود.
## ۲. بررسی شبکه
```bash
ip -br a
ip route
```
کارت `ens18` باید `UP` باشد. باید یک IPv4 معتبر و Default Route دیده شود. اگر فقط `169.254.x.x/16` ظاهر شد، DHCP ناموفق بوده است؛ این IP برای MobaXterm مناسب نیست.
## ۳. تنظیم موقت IP ثابت
```bash
ip addr flush dev ens18
ip addr add 192.168.50.156/24 dev ens18
ip link set ens18 up
ip route replace default via 192.168.50.250 dev ens18
```
این تنظیمات فقط تا زمان روشن‌بودن LiveCD باقی می‌ماند و بعد از Restart باید دوباره انجام شود.
## ۴. آزمایش شبکه
```bash
ip -br a
ip route
ping -c 3 192.168.50.250
ping -c 3 1.1.1.1
```
نتیجه درست:
- جلوی `ens18` آدرس `192.168.50.156/24` دیده شود.
- Route به‌شکل `default via 192.168.50.250 dev ens18` باشد.
- Gateway و اینترنت به Ping پاسخ دهند.
اگر `Destination Host Unreachable` دیدی، IP، Prefix، Gateway و Bridge/VLAN در Proxmox را بررسی کن.
## ۵. تعیین رمز موقت root
```bash
passwd root
```
1. رمز جدید را وارد کن و Enter بزن.
2. همان رمز را دوباره وارد کن.
3. هنگام تایپ رمز هیچ حرف یا ستاره‌ای نمایش داده نمی‌شود؛ طبیعی است.
## ۶. روشن‌کردن SSH
```bash
rc-service sshd start
rc-service sshd status
```
باید `status: started` نمایش داده شود.
بررسی اختیاری پورت 22:
```bash
ss -lntp | grep ':22'
```
## ۷. اتصال با MobaXterm
1. در MobaXterm گزینه **Session → SSH** را باز کن.
2. در **Remote host** بنویس: `192.168.50.156`
3. **Specify username** را فعال کن و `root` بنویس.
4. پورت را `22` بگذار.
5. اتصال را باز کن، Host Key اولیه را تأیید کن و رمز تعیین‌شده را وارد کن.
## ۸. تأیید اتصال
بعد از ورود در MobaXterm اجرا کن:
```bash
hostname
whoami
ip -br a
```
`whoami` باید `root` و `ens18` باید `192.168.50.156/24` را نشان دهد. تا قبل از تأیید کامل SSH، کنسول Proxmox را باز نگه دار.
## خطاهای رایج
### `Arman: command not found`
کلمه Arman در Prompt به‌عنوان دستور اجرا شده است. نیازی به واردکردن نام نیست؛ LiveCD از قبل با `root` وارد شده است.
### `Network is unreachable`
IP معتبر یا Default Route وجود ندارد. مراحل ۲ تا ۴ را دوباره بررسی کن.
### IP به‌شکل `169.254.x.x`
DHCP نتوانسته IP بدهد. تنظیمات ثابت مرحله ۳ را انجام بده.
### وضعیت SSH برابر `stopped` است
```bash
rc-service sshd start
```
سپس وضعیت را دوباره بررسی کن.
### MobaXterm وصل نمی‌شود
- از Windows آدرس `192.168.50.156` را Ping کن.
- نام کاربری `root` و پورت `22` را بررسی کن.
- وضعیت `sshd` را بررسی کن.
- Bridge، VLAN، Link و Firewall در Proxmox را کنترل کن.
## نسخه خیلی کوتاه
```bash
ip addr flush dev ens18
ip addr add 192.168.50.156/24 dev ens18
ip link set ens18 up
ip route replace default via 192.168.50.250 dev ens18
ping -c 3 1.1.1.1
passwd root
rc-service sshd start
rc-service sshd status
```
سپس در MobaXterm با `root@192.168.50.156` و پورت `22` متصل شو.
## وضعیت راهنما
- تاریخ ایجاد: ۲۷ اوت ۲۰۲۶
- مرحله پوشش‌داده‌شده: Gentoo LiveCD، شبکه و SSH
## ۹. قبل از پارتیشن‌بندی: فقط بررسی سیستم
هنوز هیچ دیسکی را پارتیشن‌بندی یا فرمت نکن. ابتدا اجرا کن:
```bash
whoami
hostname
ip -br a
ip route
test -d /sys/firmware/efi && echo UEFI || echo BIOS
lsblk -e7 -o NAME,SIZE,TYPE,FSTYPE,MOUNTPOINTS,MODEL
fdisk -l
free -h
cat /etc/resolv.conf
```
این خروجی نوع Boot، دیسک مقصد و طرح مناسب پارتیشن‌بندی را مشخص می‌کند. فقط بعد از بررسی خروجی وارد پارتیشن‌بندی می‌شویم.
**نتیجه بررسی:** VM با BIOS/Legacy بوت شده است. دیسک مقصد `/dev/sda` با ظرفیت ۳۲ گیگابایت و مدل QEMU HARDDISK است و پارتیشنی نشان نمی‌دهد. RAM حدود ۳٫۸ گیگابایت است و Swap فعال نیست. LiveCD از `/dev/sr0` و `/dev/loop0` استفاده می‌کند. DNS ابتدا کار نمی‌کرد چون `/etc/resolv.conf` هیچ Nameserver نداشت. سپس `1.1.1.1` و `8.8.8.8` ثبت شدند و Ping به IP و `gentoo.org` با صفر درصد Packet Loss موفق بود. `dhcpcd` دوباره یک Link-Local ساخت؛ چون Route ثابت درست اولویت دارد، برای جلوگیری از ریسک قطع SSH فعلاً باقی می‌ماند.
**طرح BIOS/MBR برای ****`/dev/sda`**** با ظرفیت ۳۲ گیگابایت:** پارتیشن `/dev/sda1` با اندازه ۱ گیگابایت برای Boot، `/dev/sda2` با اندازه ۴ گیگابایت برای Swap و `/dev/sda3` با تمام فضای باقی‌مانده برای Root. قبل از ذخیره با `w` خروجی `p` بررسی می‌شود.
- مرحله بعدی: آماده‌سازی دیسک تأییدشده با طرح مناسب UEFI/GPT یا BIOS
