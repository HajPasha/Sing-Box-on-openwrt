# Sing-Box-on-openwrt
نحوه راه اندازی ابزار Sing-Box بر روی OpenWRT :
در ابتدا این مقاله باید اعلام کنم که سعی کردم به راحت ترین نحو ممکن و خوانا ترین حالت این آموزش را ارائه دهم زیرا دوست دارم همه افراد با هر سطح دانشی بتواند از این آموزش و در حالت کلی از اینترنت آزاد به راحتی بهره ببرد 
پکیج ها و ابزار های مورد نیاز را ابتدا نصب میکنیم :
opkg update
opkg install kmod-inet-diag kmod-netlink-diag kmod-tun iptables-nft nano 

سپس بعد از نصب پکیج های بالا اقدام به نصب Sing-Box میکنیم 
 opkg install sing-box

 در زمان تألیف این آموزش آخرین ورژنی که از طریق دستور نصب بالا میشود ورژن 1.9.7 میباشد (در حال حاضر اخرین ورژن Stable نسخه 1.10.1 میباشد)![image](https://github.com/user-attachments/assets/ae84032b-151f-4b6b-bd73-fbd1da773a31)
در صورتی که علاقه به نصب آخرین نسخه را میباشید دو راه در پیش است 
روش اول : اپلود اخرین نسخه به صورت دستی ( استفاده از Passwall builds) : 
برای اینکار وارد سایت https://sourceforge.net/projects/openwrt-passwall-build/files/releases/ شده و در بخش releases به دنبال نسخه OpenWRT خود باشید (به عنوان مثال نسخه نصب شده برای من OpenWrt 23.05.5 میباشد) ![image](https://github.com/user-attachments/assets/63406bb5-d36a-46f0-b976-fb2340760c98)
پس روی گزینه Packages 23.05 کلیک کرده ، در صفحه بعدی باز شده باید نوع سخت افزار خود را انتخاب کنید ( اگر بر روی یک کامپیوتر قدیمی در حال نصب هستید اولین گزینه یعنی  x86_64 انتخاب شماست)![image](https://github.com/user-attachments/assets/f29b8578-5a4f-40d6-9667-663d60bbb1ba)
در صفحه بعدی گزینه passwall_packages را انتخاب کیند ![image](https://github.com/user-attachments/assets/eb648527-0e93-49b0-82e8-b347318082c1)
در این صفحه آخرین ورژن آماده نصب Sing-Box را مشاهده میکنیم بر روی آن کلیک کرده تا فایل با فرمت IPK برای شما دانلود شود . ![image](https://github.com/user-attachments/assets/782e6e98-a39b-4308-96bf-5362f47dbbf0)

پس از دانلود آخرین ورژن از طریق صفحه وب به OpenWRT خود متصل شده در بخش System > Software گزینه Upload Package... را کلبیک کرده و فایل دانلود شده را انتخاب میکنیم 
![image](https://github.com/user-attachments/assets/86666608-94f1-4ced-9eba-43e72e1df421)
سپس کافی است گزینه Upload را زده تا آخرین نسخه اپلود و سپس شروع به نصب کنید
کمی منتظر میمانیم تا اپلود و نصب آن انجام و تکمیل شود 
برای اطمینان از اینکه اخرین نسحه به درستی نصب شده است تنها کافیست به سرور OpenWRT به صورت SSH متصل شده و sing-box version را بنویسیم ![image](https://github.com/user-attachments/assets/7788f4f4-0c9c-41ed-8f9e-c31c1a6a960e)

راه حل دوم اضافه کردن لیست Passwall به OPKG ( اپدیت اتوماتیک و نصب اخرین نسخه از طریق opkg و بخش software) :
برای این کار کافی است ابتدا ابزار مورد نیاز را نصب 
opkg update 
opkg install wget-ssl
 سپس کد های زیر را مرحله به مرحله کپی و وارد سرور OpenWRT کنید :
wget -O passwall.pub https://master.dl.sourceforge.net/project/openwrt-passwall-build/passwall.pub
opkg-key add passwall.pub

کل دستور زیر را کامل کپی و یکجا وارد سرور کنید
read release arch << EOF
$(. /etc/openwrt_release ; echo ${DISTRIB_RELEASE%.*} $DISTRIB_ARCH)
EOF
for feed in passwall_luci passwall_packages passwall2; do
  echo "src/gz $feed https://master.dl.sourceforge.net/project/openwrt-passwall-build/releases/packages-$release/$arch/$feed" >> /etc/opkg/customfeeds.conf
done

در نهایت 
opkg update
opkg install sing-box
این کار باعث میشود که با هر بار دستور opkg update اخرین نسخه های داخل لیست passwall نیز دریافت شده و از طریق آن میتوان آخرین نسخه را دریافت و نصب کرد 
نظر شخصی من این است که همیشه آخرین نسخه را بلافاصله بعد از انتشار نصب نکنیم ، ممکنه است اخرین نسخه دارای ایراد یا باگ ای باشد که اتصال شما و حتی خود سرور را درگیر و مختل کنید ( البته هیچ وقت جلوی کنجکاوی و یادگیری خود را نگیرید) فقط پیشنهاد من این است که روی سیستم یا سروری که تعدادی از افراد به طور مداوم در حال استفاده از ان هستند برای جلوگیری از اختلال و هر گونه مشکل دیگری از نسخه های کمی قدیمی تر استفاده کنیم) ( به همین دلیل من با نسخه 1.9.7) آموزش را ادامه خواهم داد.

راه اندازی کانفیگ برای Sing-Box : در تنظیم اولیه کانفیگ در مسیر زیر قرار دارد 
/etc/sing-box/

برای راحتی کار بهتر است وارد این مسیر شده برای اینکار فقط کافی است 
cd /etc/sing-box 
برای اینکه بخواهیم کانفیگ sing-box را تغییر یا ویرایش کنیم میتوان از ابزار nano استفاده کرد 
nano /etc/sing-box/config.json 
برای اولین بار که این فایل را باز میکنیم یک کانفیگ نمونه وجود دارد ، در زمان تألیف این آموزش شخصا چندین بار کانفیگ ای که بتواند بهترین خروجی را بدهد ( با توجه به اتصال پایدار ، سرعت و مقوله DNS Leak) بهترین گزینه را از نظر خودم داخل همین Repository گذاشتم 
( با توحه به انیکه ابزار sing-box دائما در حال اپدیت و تغییر است ، دو مدل کانفیگ را ایجاد کردم ، نسخه config197.json مرتبط با نسخه 1.9.7 و نسخه config1101 مرتبط با اخرین ورژن در حال حاضر  (نسخه 1.10.1 )میباشد 
شخصا سعی میکنم طبق اخرین نسخه ها تغییرات را اعمال کرده و نسخه به روز این کانفیگ را ارائه کنم
کانفیگ های ارائه شده داخل این آموزش فقط نیاز به اضافه کردن بخش outbound میباشد الباقی بخش ها بدون نیاز به دستکاری میباشند

از کجا کانفیگ های متانسب برداشت کنیم ؟
خیلی راحت اگر از پنل S-UI استفاده میکنید فقط کافی است هنگام خروجی گرفتن از گزینه JSON SubScription را انتخاب کنید و قسمت Outbound را برداشته و در کانفیگ نمونه جایگذاری میکنیم ![image](https://github.com/user-attachments/assets/8c529d2e-8e86-4c5a-b9b4-41f26802982b)

ترجیح بنده برای تغییر در کانفیک خود و استفاده از این کافنیگ نمونه استفاده از برنامه VS Code میباشد 
بعد از اعمال تغییرات برای صحت درستی و قابل اجرا بودن ان توسط sing-box کافی است sing-box check را وارد کرده در صورتی که هیچ پاسخی نشان نداد به معنی درست بودن کانفیگ میباشد 
اگر کانفیگ خود را به اسمی به غیر از config.json ذخیره کرده باشید به طور عادی sing-box ان را بررسی نمیکند در این صورت باید به شکل زیر کد را وارد کنید 
sing-box check -c yourconfig.json 
گزینه c- برای ان مواقعی است که کانفیگ شما در یک مسیر یا directory دیگر یا با نام به غیز از config.json ذخیره شده است 
دلخواه : برای اینکه کانفیگ مرتب و خواناتری داشته باشیم میتوانیم از sing-box format -c yourconfig.json -w نیز استفاده کنیم 
گزینه c- برای ان مواقعی است که کانفیگ شما در یک مسیر یا directory دیگر یا با نام به غیز از config.json ذخیره شده است 
گزینه w- برای اعمال تغییرات توسط ابزار فرمت sing-box بر روی کانفیگ ارائه شده توسط شماست

بعد از چک کردن کانفیگ و اطلاع از درست بودن آن برای تست کردن به درستی اجرا شدن ان sing-box run -c yourconfig.json را وارد میکنیم 
برای اولین اجرا فایل های مورد نیاز مانند در بخش routing (مثل geosite-ir سایت های ایرانی) یا پنل metacube را دانلود میکند و ممکن است کمی زمانبر باشد ( فقط برای اولین اجرا).
چرا ما این موارد را نیاز داریم ؟ 
مواردی مانند routing به ما کمک میکند که سایت های داخلی مانند صفحه پرداخت ها ، سایت یا برنامه بانک ها و ... بدون نیاز به خاموش کردن VPN قابل دسترس باشند ، مخصوصا در سناریو ما که در حال اجرای VPN بر روی کل شبکه هستیم پس به این خاطر نیاز است بسیار شمرده و حساب شده پیش ببریم 
موارد دیگر مانند پنل metacube برای این است که ما بدون نیاز به وارد شدن به محیط SSH سرور OpenWRT بتوانیم وضعیت sing-box را مشاهده ، log برنامه را بررسی و در صورت نیاز در بخش proxies سرور خود را تغییر دهیم ( در صورتی که کانفیگ های شما دارای چند سرور میباشد) 

برای اینکه مطمئن باشیم که sing-box به درستی اجرا و در حال کار میباشد باید در بخش log خود گزینه 

+0000 2024-11-17 13:20:57 INFO sing-box started (0.211s)
حتما مشاهده شود 
![image](https://github.com/user-attachments/assets/54406c72-73bb-44f6-b62d-3de83aebbc84)

حال اگر دیدیم بدون مشکل در حال اچراست میتوانیم تنظیمات مربوطه داخل پنل OprnWRT را برای مسیریابی اینترنت شما از طریق sing-box را اعمال کنیم 
در بخش inbound در کانفیگ sing-box گزینه ای به اسم "interface_name" وجود دارد در کانفیگ نمونه اینجا به اسم "sing-tun" اسم گذاری شده است .
برای تنظیم استفاده از sing-box در OpenWRT ابتدا به بخش Network > Interface مراجعه کرده ![image](https://github.com/user-attachments/assets/7091b7bb-1726-43ea-b431-56b4080dcd9a)
در این بخش گزینه ![image](https://github.com/user-attachments/assets/991302fd-bc45-4bd4-b966-9eb76afcc0e5) را انتخاب کنید 
در صفحه باز شده :
Name : نام دلخواه به عنوان مثال SingBoxTUN
Protocol : گزینه Unmanaged را انتخاب کنید 
Device : Ethernet Adaptor با اسم "sing-tun" یا هر اسمی که در بخش Inbound به آن داده اید
مثال : ![image](https://github.com/user-attachments/assets/e0d8a271-2471-43ef-8da8-8f6f73719b4d)


سپس در مرحله بعد به بخش Network > Firewall مراجعه کرده
![image](https://github.com/user-attachments/assets/59b579c5-2017-4500-9557-340c80332c64)
در پایین صفحه در قسمت Zone بر روی گزینه ![image](https://github.com/user-attachments/assets/a53ede73-5ab4-439c-ad08-5fedb33646ef) کلیک کرده.
در صفحه باز شده :
Name : نام دلخواه به عنوان مثال SingBox
Input : Accept 
OutPut : Accept
Forward : Accept
Masquerading : تیک خورده باشد 
Covered networks : مساوی نام Interface ای که در بخش قبلی ایجاد کردیم 
Allow forward from source zones: Lan 
مثال: 
![image](https://github.com/user-attachments/assets/67d0499c-2710-42ad-a1fa-63b387aff819)
در نهایت باید به شکل زیر باشد : 
![image](https://github.com/user-attachments/assets/609806c9-529d-408e-a646-cc47924699b5)


تا به اینجا ما توانستیم ابزار Sing-box  و تنظیمات مربوطه به آن و OpenWRT را راه اندازی کرده در این مرحله به سراغ دستگاه های خود مانند تلفن و لب تاب میرویم 
داخل اندروید : 
ابتدا وارد تنظیمات شده سپس بخش Network and Internet ![image](https://github.com/user-attachments/assets/22b639f3-1a2d-4792-bae3-cf624237d87e)
بر روی ایکون تنظیمات کلیک کرده 
![image](https://github.com/user-attachments/assets/1c97790a-588b-45fd-b0a5-dc4ba0741ca9)
در این صفحه باید گزینه ای با نام IP Address وجود داشته باشد آنرا به خاطر بسپارید زیرا در مرحله بعد به آن نیاز پیدا خواهیم کرد 

حال بر روی ایکون مداد کلیک کرده 
![image](https://github.com/user-attachments/assets/3c6c1b9a-6c20-407c-b7c8-9d0ba4ce5337)
در صفحه باز شده بر روی گزینه Advance Option کلیک کرده و گزینه IP Settings را از حالت DHCP به حالت Static تغییر دهید و تنظیمات نهایی را وارد کنید :
IP address : آی پی گوشی شما که در بخش قبل پیدا کردیم 
Gateway : آی پی سرور OpenWRT
Network Prefix length : تنظیم آن کاری نداریم 
DNS 1 : آی پی سرور OpenWRT
مثال : 
![image](https://github.com/user-attachments/assets/f0e39e29-0149-442b-a6bd-6f8f3b17a6e1)

برای اینکه تنظیمات اعمال شود یک بار وای فای گوشی را قطع و مجددا وصل مینماییم 
حال خواهیم دید که همه آپ ها و سایت ها بدون نیاز به VPN کار خواهند کرد !!
تبریک شما توانستید تنها با چند مرحله تمام شبکه داخلی خود را متصل به اینترنت بدون محدودیت کنید 















