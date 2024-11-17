### راه‌اندازی Sing-Box روی OpenWRT

در این آموزش تلاش شده است تا با ساده‌ترین روش ممکن و به صورت خوانا، نحوه راه‌اندازی **Sing-Box** روی سیستم عامل **OpenWRT** توضیح داده شود تا افراد با هر سطح دانش بتوانند از این آموزش بهره‌مند شوند و از اینترنت آزاد استفاده کنند.

---

### نصب ابزارها و پکیج‌های مورد نیاز  

ابتدا باید پکیج‌های ضروری را نصب کنیم. دستورات زیر را در ترمینال OpenWRT اجرا کنید:

```bash
opkg update
opkg install kmod-inet-diag kmod-netlink-diag kmod-tun iptables-nft nano
```

سپس ابزار **Sing-Box** را نصب کنید:

```bash
opkg install sing-box
```

> **نکته:**  
> در زمان نگارش این آموزش، نسخه‌ای که از طریق دستور بالا نصب می‌شود، نسخه **1.9.7** است. در حال حاضر آخرین نسخه Stable این ابزار **1.10.1** می‌باشد.  
اگر می‌خواهید آخرین نسخه را نصب کنید، دو روش وجود دارد:

---

### روش اول: نصب دستی نسخه جدید (استفاده از Passwall Builds)  

1. به سایت [SourceForge](https://sourceforge.net/projects/openwrt-passwall-build/files/releases/) بروید.  
2. نسخه OpenWRT خود را پیدا کنید (مثلاً اگر نسخه OpenWRT شما **23.05** است، گزینه مربوط به آن را انتخاب کنید).  
3. نوع سخت‌افزار خود را مشخص کنید (برای مثال، اگر OpenWRT روی یک کامپیوتر قدیمی اجرا می‌شود، گزینه **x86_64** مناسب است).  
4. وارد پوشه **passwall_packages** شوید و فایل **Sing-Box** با فرمت **IPK** را دانلود کنید.  

پس از دانلود فایل:

1. از طریق صفحه وب OpenWRT وارد بخش **System > Software** شوید.  
2. روی گزینه **Upload Package...** کلیک کرده و فایل دانلود شده را آپلود کنید.  
3. منتظر بمانید تا نصب تکمیل شود.  

برای اطمینان از نصب موفقیت‌آمیز، دستور زیر را اجرا کنید:

```bash
sing-box version
```

---

### روش دوم: افزودن لیست Passwall به OPKG  

این روش باعث می‌شود بتوانید آخرین نسخه Sing-Box را به صورت خودکار از طریق **opkg** نصب کنید:

1. ابزارهای مورد نیاز را نصب کنید:

```bash
opkg update
opkg install wget-ssl
```

2. دستورات زیر را وارد کنید:

```bash
wget -O passwall.pub https://master.dl.sourceforge.net/project/openwrt-passwall-build/passwall.pub
opkg-key add passwall.pub

read release arch << EOF
$(. /etc/openwrt_release ; echo ${DISTRIB_RELEASE%.*} $DISTRIB_ARCH)
EOF
for feed in passwall_luci passwall_packages passwall2; do
  echo "src/gz $feed https://master.dl.sourceforge.net/project/openwrt-passwall-build/releases/packages-$release/$arch/$feed" >> /etc/opkg/customfeeds.conf
done

opkg update
opkg install sing-box
```

> **نکته:** پیشنهاد می‌شود آخرین نسخه ابزارها را بلافاصله پس از انتشار نصب نکنید؛ چراکه ممکن است مشکلاتی داشته باشند. به همین دلیل در این آموزش از نسخه **1.9.7** استفاده شده است.

---

### تنظیم و ویرایش کانفیگ Sing-Box  

فایل کانفیگ Sing-Box در مسیر زیر قرار دارد:

```
/etc/sing-box/
```

برای ویرایش فایل کانفیگ، ابتدا به مسیر فوق بروید:

```bash
cd /etc/sing-box
nano config.json
```

> **نکته:**  
> در این آموزش، نمونه کانفیگ مناسب نسخه‌های **1.9.7** و **1.10.1** در همین آموزش قرار داده شده است. تنها کافی است بخش **outbound** را به کانفیگ اضافه کنید.
> برای ورژن `1.9.7` کانفیگ با نام `config197.json` را استفاده کنید
> برای ورژن `1.10.1` کانفیگ با نام `config1101.json` را استفاده کنید 

#### استخراج کانفیگ مناسب  

اگر از پنل **S-UI** استفاده می‌کنید، کافی است خروجی **JSON Subscription** را انتخاب کرده و بخش **Outbound** را در کانفیگ جای‌گذاری کنید.  

---

### بررسی صحت کانفیگ  

برای اطمینان از درستی کانفیگ، دستور زیر را اجرا کنید:

```bash
sing-box check
```

اگر پاسخی نمایش داده نشد، یعنی کانفیگ شما صحیح است.  

در صورتی که فایل کانفیگ با نام دیگری ذخیره شده باشد:

```bash
sing-box check -c yourconfig.json
```
> **نکته:**  
> در صورتی که کانفیگ خود را با نامی به غیر از `config.json` یا در پوشه ای دیگر ذخیره کرده باشید باید از تگ `-c` استفاده کنید و مسیر کانفیگ خود را وارد کنید 

برای مرتب کردن کانفیگ (دلخواه پیشنهادی) :

```bash
sing-box format -c yourconfig.json -w
```

---

### اجرای Sing-Box  

برای اجرای ابزار:

```bash
sing-box run -c yourconfig.json
```

در اولین اجرا، ابزار فایل‌های مورد نیاز مانند **geosite-ir** را دانلود می‌کند. این مرحله ممکن است کمی زمان‌بر باشد.  

---
### ویرایش و راه‌اندازی سرویس Sing-Box

برای مدیریت خودکار **Sing-Box** به‌صورت یک سرویس، فایل `sing-box` در مسیر `init.d`را ویرایش کرده و دستوراتی را برای مدیریت راه‌اندازی، توقف، و بارگذاری مجدد تنظیم کنید. در اینجا مراحل کامل آمده است:

---

#### گام ۱: ویرایش فایل `/etc/init.d/sing-box`

فایل را با ویرایشگر متن باز کنید:

```bash
nano /etc/init.d/sing-box
```

سپس کد زیر را به فایل به اخرین خط اضافه کنید:

### **توصیه‌ها:**
1. پیشنهاد می‌شود این کد را در فایل `init.d/sing-box` اضافه کنید، نه اینکه کدهای موجود را جایگزین کنید. این کار به کاهش تداخل‌ها و مشکلات احتمالی کمک می‌کند.

```bash
#!/bin/sh /etc/rc.common

START=99
USE_PROCD=1

#####  فقط این بخش را تغییر دهید #####
PROG=/usr/bin/sing-box 
RES_DIR=/etc/sing-box/ # مسیر منابع / دایرکتوری کاری / محل ذخیره لیست‌های آی‌پی یا دامنه
CONF=./config.json   # محل فایل کانفیگ، می‌تواند مسیر نسبی به $RES_DIR باشد
#####  فقط این بخش را تغییر دهید #####

start_service() {
  sleep 10 
  procd_open_instance
  procd_set_param command $PROG run -D $RES_DIR -c $CONF

  procd_set_param user root
  procd_set_param limits core="unlimited"
  procd_set_param limits nofile="1000000 1000000"
  procd_set_param stdout 1
  procd_set_param stderr 1
  procd_set_param respawn "${respawn_threshold:-3600}" "${respawn_timeout:-5}" "${respawn_retry:-5}"
  procd_close_instance
  iptables -I FORWARD -o tun+ -j ACCEPT
  echo "sing-box is started!"
}

stop_service() {
  service_stop $PROG
  iptables -D FORWARD -o tun+ -j ACCEPT
  echo "sing-box is stopped!"
}

reload_service() {
  stop
  sleep 5s
  echo "sing-box is restarted!"
  start
}
```

---

#### گام ۲: تنظیم دسترسی‌های اجرایی

فایل را اجرایی کنید:

```bash
chmod +x /etc/init.d/sing-box
```

---

#### گام ۳: افزودن به استارتاپ (شروع به کار کردن sing-box به محض روشن شدن سرور)

برای اضافه شدن سرویس به استارتاپ و اجرای خودکار آن هنگام راه‌اندازی مجدد دستگاه:

```bash
/etc/init.d/sing-box enable
```

برای اجرای سرویس sing-box:

```bash
/etc/init.d/sing-box start
```

---
### **توصیه‌ها:**
1. هنگام توقف یا راه‌اندازی مجدد سرویس، حتماً تنظیمات **iptables** را بازبینی کنید تا تداخل ایجاد نشود.


---

### تنظیمات OpenWRT  

#### افزودن Interface  

1. به بخش **Network > Interface** بروید.  
2. یک **Interface** جدید با مشخصات زیر ایجاد کنید:  
   - **Name:** نام دلخواه (مثلاً `SingBoxTUN`)  
   - **Protocol:** گزینه `Unmanaged`  
   - **Device:** نام آداپتور شبکه (مثلاً `sing-tun`)  

#### تنظیمات Firewall  

1. به بخش **Network > Firewall** بروید.  
2. یک **Zone** جدید با مشخصات زیر ایجاد کنید:  
   - **Name:** نام دلخواه (مثلاً `SingBox`)  
   - **Input/Output/Forward:** روی `Accept` تنظیم شود.  
   - **Masquerading:** فعال شود.  
   - **Covered Networks:** نام Interface ایجاد شده.  
   - **Allow Forward From Source Zones:** گزینه `LAN`.  

---
### تنظیم دستگاه‌ها برای استفاده از Sing-Box و OpenWRT

پس از راه‌اندازی **Sing-Box** و پیکربندی آن روی **OpenWRT**، مرحله بعدی تنظیم دستگاه‌های شما (مانند گوشی‌ها و لپ‌تاپ‌ها) برای اتصال به شبکه است. در ادامه مراحل برای انجام این کار آمده است:

---

#### در اندروید:

1. **باز کردن تنظیمات شبکه**  
   - به **تنظیمات** > **Network and Internet** بروید.
   -    ![آیکون تنظیمات شبکه Wi-Fi](https://github.com/user-attachments/assets/22b639f3-1a2d-4792-bae3-cf624237d87e)

2. **یافتن شبکه Wi-Fi فعلی**  
   - روی آیکون چرخ‌دنده کنار شبکه Wi-Fi متصل شده ضربه بزنید.

   ![نمایش آدرس IP](https://github.com/user-attachments/assets/1c97790a-588b-45fd-b0a5-dc4ba0741ca9)

3. **یافتن آدرس IP گوشی**  
   - در صفحه تنظیمات Wi-Fi به دنبال گزینه‌ای به نام **IP Address** بگردید.  
     این آدرس IP را یادداشت کنید؛ در مرحله بعد به آن نیاز خواهید داشت.

4. **ویرایش تنظیمات شبکه**  
   - روی **آیکون مداد** برای ویرایش شبکه ضربه بزنید.

   ![ویرایش شبکه Wi-Fi](https://github.com/user-attachments/assets/3c6c1b9a-6c20-407c-b7c8-9d0ba4ce5337)

5. **فعال کردن گزینه‌های پیشرفته**  
   - به پایین صفحه بروید و گزینه **Advanced Options** را باز کنید.  
   - تنظیم **IP Settings** را از **DHCP** به **Static** تغییر دهید.

6. **وارد کردن تنظیمات شبکه استاتیک**  
   اطلاعات زیر را وارد کنید:
   - **IP Address**: آدرس IP که در مرحله 3 یادداشت کردید.  
   - **Gateway**: آدرس IP مربوط به **سرور OpenWRT**.  
   - **Network Prefix Length**: بدون تغییر باقی بگذارید.  
   - **DNS 1**: آدرس IP مربوط به **سرور OpenWRT**.

   نمونه تنظیمات:  
   ![نمونه تنظیمات شبکه استاتیک](https://github.com/user-attachments/assets/f0e39e29-0149-442b-a6bd-6f8f3b17a6e1)

7. **اعمال تنظیمات**  
   - اتصال Wi-Fi گوشی را یک‌بار خاموش و دوباره روشن کنید تا تنظیمات جدید اعمال شوند.

---

#### نکات:  
- مقادیر **Gateway** و **DNS 1** باید به آدرس IP سرور **OpenWRT** اشاره کنند که Sing-Box روی آن اجرا می‌شود.  
- اطمینان حاصل کنید که آدرس IP استاتیکی که به دستگاه خود اختصاص می‌دهید، در همان محدوده آدرس شبکه شما باشد.

با انجام این مراحل، دستگاه اندرویدی شما ترافیک خود را از طریق تنظیمات **Sing-Box** روی روتر OpenWRT هدایت می‌کند. این مراحل را برای سایر دستگاه‌ها نیز تکرار کنید تا آن‌ها را به این شبکه متصل کنید.
---

### جمع‌بندی  

تا این مرحله، ابزار Sing-Box و تنظیمات مربوطه با موفقیت نصب و راه‌اندازی شده‌اند. اکنون می‌توانید اینترنت آزاد را روی دستگاه‌های خود مانند گوشی، لپ‌تاپ و غیره تجربه کنید.

