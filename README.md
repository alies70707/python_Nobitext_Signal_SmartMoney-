# Smart Money Trading Engine (Nobitex)

موتور تحلیل و سیگنال‌دهی **Smart Money Concepts (سبک ICT/RTM)** برای بازارهای
صرافی **نوبیتکس**، نوشته‌شده با پایتون. این پروژه بازارها را رتبه‌بندی می‌کند،
ستاپ‌های معاملاتی (Order Block، FVG، Liquidity Sweep، MSS و …) را شناسایی کرده
و سیگنال خرید/فروش همراه با حد ضرر و اهداف (TP) خروجی می‌دهد. همچنین شامل یک
موتور بک‌تست ساده است.

> ⚠️ این ابزار صرفاً جهت اهداف آموزشی و تحلیلی است و هیچ توصیه مالی ارائه
> نمی‌دهد. معامله‌کردن ریسک دارد و مسئولیت تصمیمات با خود کاربر است.

---

## ویژگی‌ها

- **رتبه‌بندی بازارها**: انتخاب Top-N بازار بر اساس حجم ۲۴ساعته، نقدینگی،
  اسپرد و فعالیت (min-max normalization).
- **تحلیل چند‌تایم‌فریمی**: بایاس کلی از تایم‌فریم‌های روزانه (1D) و ۴ساعته (4H)
  و ورود از تایم‌فریم پایین‌تر (5m/15m/1H).
- **تشخیص مفاهیم Smart Money**:
  - Market Structure Shift (MSS)
  - Liquidity Sweep
  - Displacement (با تایید حجم)
  - Fair Value Gap (FVG)
  - Order Block (OB)
  - Premium / Discount Zone
- **امتیازدهی هوشمند**: Smart Money Score (۰ تا ۲۴) و سطح اطمینان (Confidence).
  - سیگنال فقط زمانی صادر می‌شود که امتیاز و نسبت ریسک/پاداش از حد نصاب
    پیکربندی بیشتر باشند.
- **مدیریت ریسک**: درصد ریسک در معامله، حداکثر ضرر روزانه، حداکثر پوزیشن‌های
  باز و حداکثر قرارگیری همبسته.
- **بک‌تست**: اجرای استراتژی روی داده‌های تاریخی و گزارش عملکرد
  (Win Rate، Profit Factor، Drawdown، Sharpe).
- **حالت مانیتورینگ**: اجرای پیوستهٔ اسکن با فاصله زمانی قابل تنظیم.

---

## معماری

```
main.py                 ورودی CLI و قالب‌بندی خروجی (بدون منطق استراتژی)
config.py               تنظیمات از متغیرهای محیطی / .env
exchange/
  nobitex_client.py     کلاینت REST نوبیتکس (stats, orderbook, trades, candles)
  market_data.py        رتبه‌بندی بازارها و ساخت snapshot
data/
  candle_manager.py     مدیریت کندل‌های چندتایم‌فریم
  historical_data.py    دانلود داده‌های تاریخی (صفحه‌بندی خودکار)
strategy/               هسته استراتژی (signal_engine + ماژول‌های کمکی)
risk/                   مدیریت ریسک و اندازه پوزیشن
backtest/               موتور بک‌تست و محاسبه عملکرد
models/                 مدل‌های داده (Candle, Setup, Signal)
utils/                  لاگر و اعتبارسنج‌ها
tests/                  ۶۶ تست واحد (پوشش منطق استراتژی، بدون نیاز به شبکه)
```

---

## نصب

```bash
git clone https://github.com/alies70707/python_Nobitext_Signal_SmartMoney-.git
cd python_Nobitext_Signal_SmartMoney-
pip install -r requirements.txt
```

پیش‌نیاز: Python 3.10+

---

## پیکربندی

تنظیمات از متغیرهای محیطی (و فایل `.env` در صورت وجود) خوانده می‌شوند.
هیچ کلید/رمزی در کد سخت‌کد نشده است.

| متغیر               | پیش‌فرض           | توضیح                                        |
|---------------------|-------------------|----------------------------------------------|
| `NOBITEX_API_KEY`   | (خالی)            | اختیاری؛ فقط برای endpointهای عمومی لازم نیست |
| `NOBITEX_API_URL`   | apiv2.nobitex.ir  | آدرس پایه API                               |
| `DEFAULT_TIMEFRAME` | 15m               | تایم‌فریم ورود پیش‌فرض                       |
| `SCAN_INTERVAL`     | 60                | فاصله (ثانیه) بین چرخه‌های مانیتورینگ       |
| `WATCHLIST`         | (خالی)            | لیست نمادها با ویرگول، مثلاً `btc-rls,eth-rls` |
| `RISK_PER_TRADE`    | 0.005             | درصد ریسک در هر معامله                       |
| `MAX_DAILY_LOSS`    | 0.02              | حداکثر ضرر روزانه                            |
| `MAX_OPEN_POSITIONS`| 3                 | حداکثر پوزیشن‌های باز                        |
| `MIN_SMART_MONEY_SCORE` | 16          | حد نصاب امتیاز برای صدور سیگنال              |
| `MIN_RISK_REWARD`   | 2.0               | حد نصاب نسبت ریسک/پاداش                       |
| `ATR_MULTIPLIER`    | 0.2               | ضریب بافر استاپ‌لاس نسبت به ATR              |
| `INITIAL_CAPITAL`   | 100000000         | سرمایه اولیه بک‌تست (IRR)                    |
| `FEE_RATE`          | 0.0005            | کارمزد                                       |
| `SLIPPAGE`          | 0.0002            | لغزش                                         |

نمونه `.env`:
```env
DEFAULT_TIMEFRAME=15m
SCAN_INTERVAL=60
WATCHLIST=btc-rls,eth-rls
MIN_SMART_MONEY_SCORE=16
```

---

## استفاده

```bash
# تحلیل نماد پیش‌فرض (BTCIRT)
python main.py

# تحلیل یک نماد و تایم‌فریم مشخص
python main.py --symbol BTCIRT --timeframe 15m

# رتبه‌بندی و تحلیل ۱۰ بازار برتر
python main.py --scan

# اسکن یک لیست نظارتی (watchlist)
python main.py --symbols btc-rls,eth-rls,xrp-rls

# مانیتورینگ پیوسته (همراه با --scan یا --symbols)
python main.py --scan --monitor
python main.py --symbols btc-rls,eth-rls --monitor

# بک‌تست
python main.py --backtest
python main.py --backtest --symbol BTCIRT --timeframe 15m --verbose
```

خروجی در حالت اسکن یک داشبورد تک‌صفحه‌ای شامل نماد، جهت، امتیاز، نقطه ورود،
استاپ‌لاس و TP1 و نسبت R:R نمایش می‌دهد. با `--verbose` جزئیات هر سیگنال
(توضیح ستاپ و تفکیک امتیاز) چاپ می‌شود.

---

## پایپ‌لاین استراتژی (بدون نگاه به آینده)

۱. بایاس کلی (HTF) از Daily + 4H
۲. شناسایی نقدینگی تا کندل MSS
۳. تشخیص MSS (نیازمند sweep + displacement + شکست ساختار)
۴. تشخیص و امتیازدهی FVG حوالی MSS
۵. تشخیص Order Block پیش از MSS
۶. طبقه‌بندی Premium/Discount نقطه ورود
۷. محاسبه ورود، استاپ‌لاس و اهداف (بر پایه نقدینگی)
۸. اعتبارسنجی نسبت ریسک/پاداش
۹. امتیازدهی Smart Money و سطح اطمینان
۱۰. صدور سیگنال فقط در صورت عبور از تمام دروازه‌ها

---

## تست

```bash
pytest tests/ -q
```

تمامی تست‌ها به صورت آفلاین (بدون نیاز به شبکه) اجرا می‌شوند و منطق استراتژی
را پوشش می‌دهند.

---

## نکات فنی

- کلاینت نوبیتکس دفاعی نوشته شده: timeout، retry با backoff، مدیریت
  rate-limit (۴۲۹/۵xx) و عدم crash در صورت خطای یک درخواست.
- کلید API (در صورت تنظیم) هرگز در لاگ‌ها نشت نمی‌کند (SecretFilter).
- نمادهای حاوی زیرخط در بیس (مثل `1k_shib-rls`) به فرم معتبر API
  (`1K_SHIBIRT`) نرمال‌سازی می‌شوند.

---

## سلب مسئولیت

این نرم‌افزار ابزاری تحلیلی است و هیچ مسئولیتی در قبال زیان‌های مالی ناشی از
استفاده از آن ندارد. پیش از استفاده در محیط واقعی، حتماً با بک‌تست و بررسی
دستی اعتبارسنجی کنید.
