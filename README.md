# Cloudflare Lab · آزمایشگاه کلادفلر

شبیه‌ساز سه‌بعدی و تعاملی که نشان می‌دهد درخواست کاربر در هر حالت SSL/TLS کلادفلر روی کدام پورت و با چه پروتکلی به سرور اوریجین می‌رسد. رفتار شبیه‌ساز با تست واقعی روی یک دامنه‌ی زنده تنظیم شده است، نه فقط با مستندات.

An interactive 3D simulator of how Cloudflare maps visitor HTTP/HTTPS ports to the origin server in each SSL/TLS mode. Its behavior is calibrated against live tests on a real zone, not only the documentation.

**Live demo:** `https://<your-username>.github.io/cloudflare-lab/`

---

## امکانات

- حالت‌های Off، Flexible، Full، Full (strict) و Strict (SSL-Only Origin Pull)
- پروکسی روشن (ابر نارنجی) و خاموش (DNS only)
- درخواست روی پورت‌های 80، 443، 8080، 8443 و هر پورت دلخواه
- Always Use HTTPS، Origin Rule برای تغییر پورت مقصد، و ریدایرکت سمت اوریجین
- نمایش قدم‌به‌قدم مسیر درخواست و پاسخ، با ریدایرکت‌ها و حلقه‌ی ریدایرکت
- سناریوهای خطا: 520، 521، 526، 400، timeout و خطای گواهی
- ناوبری کامل در صحنه: چرخش، جابه‌جایی، زوم، پرش به کاربر/لبه/اوریجین، دابل‌کلیک برای رفتن به هر نقطه، و حالت دنبال کردن بسته
- دوزبانه فارسی و English، با میان‌برهای صفحه‌کلید برای ضبط ویدیو

## ناوبری

| کار | ماوس | صفحه‌کلید | گوشی |
|---|---|---|---|
| چرخش | کشیدن | — | یک انگشت |
| جابه‌جایی | راست‌کلیک یا Shift + کشیدن | کلیدهای جهت | دو انگشت |
| زوم | اسکرول | `+` / `−` | دو انگشت (pinch) |
| رفتن به یک نقطه | دابل‌کلیک | — | — |
| دنبال کردن بسته | دکمه‌ی «دنبال کردن بسته» | `F` | دکمه |
| کل صحنه | دکمه‌ی ⌂ | `0` | دکمه |

## نتیجه‌های تست واقعی

| حالت | `http` :80 | `https` :443 | `https` :8443 | `http` :8080 |
|---|---|---|---|---|
| Off | HTTP → 80 | HTTP → 80 | HTTP → 8443 | HTTP → 8080 |
| Flexible | HTTP → 80 | HTTP → 80 | TLS → 8443 | HTTP → 8080 |
| Full | HTTP → 80 | TLS → 443 | TLS → 8443 | HTTP → 8080 |
| Full (strict) | HTTP → 80 | TLS + cert check → 443 | TLS + cert check → 8443 | HTTP → 8080 |
| + Always Use HTTPS | 301 → https:443 | — | — | 301 → https:443 (port dropped) |
| Unsupported port (3000, 9443), proxied | timeout | timeout | | |
| DNS only | direct to the same port | | | |

نکته‌هایی که با مستندات فرق داشت:

- در حالت Off، درخواست HTTPS به HTTP ریدایرکت **نشد**. لبه خودش HTTPS را جواب داد و با HTTP به پورت 80 اوریجین رفت. این حتی با هدرهای مرورگر Chrome هم همین‌طور بود.
- Always Use HTTPS روی پورت‌های HTTP غیراستاندارد هم ریدایرکت می‌کند، ولی Location شماره‌ی پورت را حذف می‌کند.
- تغییر حالت SSL ظرف حدود ۲ تا ۱۵ ثانیه روی لبه اعمال می‌شود.

خروجی خام تست‌ها در پوشه‌ی [`results/`](results/) است. اسم دامنه و IP سرور در آن‌ها جایگزین شده‌اند.

## اجرای تست روی دامنه‌ی خودتان

اسکریپت‌ها در پوشه‌ی [`tools/`](tools/) هستند و روی یک سرور لینوکسی با Python 3.7 به بالا، `curl` و `openssl` اجرا می‌شوند.

| فایل | کار |
|---|---|
| `cf-lab-run.py` | کل ماتریس حالت‌ها را خودکار تست می‌کند و در پایان تنظیمات را برمی‌گرداند |
| `cf-lab-followup.py` | با اوریجینی که هم TLS و هم متن ساده را می‌پذیرد، نشان می‌دهد کلادفلر دقیقاً چه چیزی فرستاده |
| `cf-origin-echo.py` و `cf-port-test.sh` | نسخه‌ی دستی: پاسخ‌گو روی سرور و تست‌کننده‌ی جداگانه |

توکن API لازم: فقط برای همان zone، با مجوزهای Zone Read، Zone Settings Edit و DNS Edit.

```
sudo ufw allow 80,443,8080,8443,2052,2053,3000,9443/tcp
read -rs CF_API_TOKEN
export CF_API_TOKEN
python3 tools/cf-lab-run.py --zone yourdomain.com --cert cert.pem --key key.pem
```

اسکریپت یک زیردامنه‌ی موقت (`cflab.yourdomain.com`) می‌سازد و در پایان پاکش می‌کند. اگر وسط کار قطع شد، با `--restore` همه‌چیز برمی‌گردد. حالت SSL روی کل zone اعمال می‌شود، پس اگر رکورد دیگری با ابر نارنجی دارید، اسکریپت اول از شما تأیید می‌گیرد. بعد از تست توکن را Revoke کنید و پورت‌های تست را ببندید.

## انتشار روی GitHub Pages

Settings ← Pages ← Source: Deploy from a branch ← Branch: `main` و پوشه‌ی `/ (root)` ← Save.

## منابع

- [Encryption modes · Cloudflare Docs](https://developers.cloudflare.com/ssl/origin-configuration/ssl-modes/)
- [Network ports · Cloudflare Docs](https://developers.cloudflare.com/fundamentals/reference/network-ports/)
- [Origin Rules · Cloudflare Docs](https://developers.cloudflare.com/rules/origin-rules/)
