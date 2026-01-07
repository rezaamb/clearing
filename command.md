0) چک سریع فضای درایو C و بزرگ‌ترین پوشه‌ها

بررسی فضای خالی و مصرف‌شده‌ی C

```bash
Get-PSDrive C | Select-Object Name,@{n="Free(GB)";e={[math]::Round($_.Free/1GB,2)}},@{n="Used(GB)";e={[math]::Round(($_.Used)/1GB,2)}}
```

چه می‌کند؟

میزان Free و Used درایو C را بر حسب گیگابایت نشان می‌دهد

بدون هیچ تغییری، فقط گزارش

بزرگ‌ترین پوشه‌های ریشه C

```bash
Get-ChildItem C:\ -Force -Directory |
  ForEach-Object {
    $s=(Get-ChildItem $_.FullName -Recurse -Force -File -Attributes !ReparsePoint -ErrorAction SilentlyContinue | Measure-Object Length -Sum).Sum
    [pscustomobject]@{Folder=$_.FullName; GB=[math]::Round(($s/1GB),2)}
  } | Sort-Object GB -Descending | Select-Object -First 15 | Format-Table -AutoSize
```

چه می‌کند؟

پوشه‌های سطح اول C:\ را بررسی می‌کند

حجم واقعی هر پوشه (جمع فایل‌ها) را حساب می‌کند

۱۵ پوشه‌ی بزرگ‌تر را نمایش می‌دهد

لینک‌های سیستمی (ReparsePoint) نادیده گرفته می‌شوند

1) پاکسازی‌های امن (هیچ دیتای مهمی حذف نمی‌شود)

خالی کردن Recycle Bin
```bash
Clear-RecycleBin -Force
```
چه می‌کند؟

سطل زباله ویندوز را کاملاً پاک می‌کند

پاک‌سازی Tempهای عمومی ویندوز

```bash
Remove-Item "$env:TEMP\*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "C:\Windows\Temp\*" -Recurse -Force -ErrorAction SilentlyContinue
```

چه می‌کند؟

فایل‌های موقتی کاربر و سیستم را حذف می‌کند

اگر فایلی در حال استفاده باشد، بدون خطا رد می‌شود


پاک‌سازی Cacheهای سبک کاربری

```bash
Remove-Item "C:\Users\r.babanezhad\AppData\Local\Temp\wsl-crashes\*" -Recurse -Force -ErrorAction SilentlyContinue
Remove-Item "C:\Users\r.babanezhad\AppData\Local\Microsoft\Terminal Server Client\Cache\*" -Force -ErrorAction SilentlyContinue
```

چه می‌کند؟

لاگ‌های کرش WSL

کش ریموت دسکتاپ (RDP Bitmap Cache)

کاملاً امن

2) ریز کردن پروفایل کاربر r.babanezhad
   
بزرگ‌ترین پوشه‌های داخل پروفایل

```bash

$u="C:\Users\r.babanezhad"
Get-ChildItem $u -Force -Directory |
  ForEach-Object {
    $s=(Get-ChildItem $_.FullName -Recurse -Force -File -Attributes !ReparsePoint -ErrorAction SilentlyContinue | Measure-Object Length -Sum).Sum
    [pscustomobject]@{Folder=$_.FullName; GB=[math]::Round(($s/1GB),2)}
  } | Sort-Object GB -Descending | Select-Object -First 20 | Format-Table -AutoSize
```
چه می‌کند؟

پوشه‌های اصلی داخل پروفایل کاربر را بررسی می‌کند

نشان می‌دهد کدام فولدر بیشترین فضا را گرفته (Downloads، AppData، Desktop و …)

شکار فایل‌های حجیم داخل پروفایل



```bash
$u="C:\Users\r.babanezhad"
Get-ChildItem $u -Recurse -Force -File -Attributes !ReparsePoint -ErrorAction SilentlyContinue |
  Sort-Object Length -Descending |
  Select-Object -First 30 FullName,@{n="GB";e={[math]::Round($_.Length/1GB,2)}} |
  Format-Table -AutoSize
```

چه می‌کند؟

۳۰ فایل بزرگ‌تر داخل پروفایل را لیست می‌کند

هدف: پیدا کردن ISO، ZIP، LOG، DB، CACHEهای حجیم



3) کامپکت کردن docker_data.vhdx (بیشترین آزادسازی فضا)

   
آماده‌سازی و خاموش کردن WSL


```bash
$path = "C:\Users\r.babanezhad\AppData\Local\Docker\wsl\disk\docker_data.vhdx"
wsl --shutdown | Out-Null
```

چه می‌کند؟

میاد WSL و Docker را کامل خاموش می‌کند

شرط لازم برای Compact

نمایش اندازه قبل
```bash
"BEFORE:"; Get-Item $path | Select-Object FullName,@{n="GB";e={[math]::Round($_.Length/1GB,2)}},LastWriteTime
```

Compact امن با DiskPart

```bash
$dp = @"
select vdisk file="$path"
attach vdisk readonly
compact vdisk
detach vdisk
exit
"@
$script = Join-Path $env:TEMP "compact-docker-vhdx.txt"
$dp | Set-Content -Path $script -Encoding ASCII
diskpart /s $script
```

چه می‌کند؟

فضای آزاد داخل VHDX را فشرده می‌کند

هیچ ایمیج یا کانتینری حذف نمی‌شود

امن‌ترین روش رسمی مایکروسافت

نمایش اندازه بعد

```bash
"AFTER:"; Get-Item $path | Select-Object FullName,@{n="GB";e={[math]::Round($_.Length/1GB,2)}},LastWriteTime
```

4) برگرداندن وضعیت و بررسی سلامت Docker
```bash
wsl -l -v
docker context ls
docker info --format "{{.Name}} | {{.Driver}} | {{.DockerRootDir}}"
```

چه می‌کند؟

وضعیت WSL

کانتکست Docker

مسیر دیتای Docker







