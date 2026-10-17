# Ринковий радар

Статичний односторінковий віджет для моніторингу USDT-M perpetual futures Gate.io. Інтерфейс українською; список контрактів зберігається у localStorage браузера.

## Запуск

Відкрийте `index.html` у сучасному браузері з доступом до публічного Gate.io API v4 (`https://api.gateio.ws/api/v4`). Backend і API keys не потрібні. Можна додавати та видаляти до 30 контрактів; дозволений формат — `BTC_USDT` або `BTCUSDT`. Дані оновлюються щохвилини.

## Використані публічні endpoints

- `GET /futures/usdt/tickers` — last price, 24h change, quote volume, funding rate, mark price та index price.
- `GET /futures/usdt/contracts` — funding interval (секунди) для контрактів.
- `GET /futures/usdt/candlesticks` — закриті свічки `4h` та `15m`, обсяг свічки в контрактах.
- `GET /futures/usdt/contract_stats` — OI в контрактах і quote currency, зміна OI, L/S account/taker та top-trader ratios, направлені long/short liquidation amounts.
- `GET /futures/usdt/funding_rate` — до 12 історичних funding rates; кеш на 15 хвилин для зниження кількості запитів.
- `GET https://api.dexscreener.com/token-pairs/v1/{chainId}/{tokenAddress}` — token pairs/liquidity/FDV/market cap, якщо ці поля повернуто; запит лише за мережею та точною адресою.
- `GET https://api.geckoterminal.com/api/v2/networks/{network}/tokens/{tokenAddress}/pools?page=1` — pools/reserve/FDV/market cap, якщо ці поля повернуто; кеш результату на 5 хвилин.

Ендпоїнти та поля перевірено за офіційним Gate.io API v4 JavaScript/Python SDK-довідником. Ліквідації беруться з `long_liq_*` і `short_liq_*` полів `contract_stats`; напрямок не виводиться зі знака чи кольору. Якщо Gate.io не повертає поле, віджет показує N/A; помилки API позначаються окремо та не переривають завантаження інших контрактів.

## Індикатори та статуси

- RSI, MACD, EMA20, price extension і volume anomaly розраховуються з реальних закритих Gate.io свічок. Перегрів аналізується на 4H, ведмежий розворот і точка входу — окремо на 15m; незакриті свічки відкидаються, тому поточна свічка не repaint-иться в оцінці.
- SQUEEZE EVIDENCE, BEARISH CONFIRMATION та SHORT ENTRY QUALITY — окремі оцінки 0–100. Недоступні фактори виключаються з знаменника; кількість факторів із даними показується біля кожної оцінки.
- SHORT ENTRY QUALITY = 35% Squeeze Evidence + 45% Bearish Confirmation + 20% якості точки входу. Статус «⚠️ ВХІД НА ПРОДАЖ» вимагає одночасно загальну оцінку ≥65, squeeze ≥45, confirmation ≥65 і point quality ≥50. Високий squeeze score сам по собі не створює SHORT-статус.
- Squeeze thresholds: позитивний funding; funding вище історичного середнього на одне стандартне відхилення (мінімум три спостереження); OI зростає або abnormal OI growth ≥3%; ratios ≥1.5; 4H price extension ≥3%; 4H volume ≥1.5× середнього; RSI ≥70; short liquidations більші за long і становлять щонайменше 0.1% OI quote value.
- Sell-volume підтвердження на 15m: обсяг ведмежої свічки більше попереднього щонайменше на 15%. Точка входу оцінює close нижче локального рівня та відстань до рівня не більше 1.5 ATR.
- Статуси та їхні пороги пояснені у віджеті. Squeeze score оцінює потенційний перегрів LONG; він не є короткою позицією чи рекомендацією.

## Додаткові market data та contract lookup

Token lookup вручну використовує лише публічні безключові endpoints DexScreener і GeckoTerminal для Ethereum, Polygon, Arbitrum, Optimism, Base, BNB Chain та Solana. Пошук не виконується за символом/назвою: потрібні мережа й адреса токена. Віджет зіставляє адресу з поверненими парами, показує джерело, ліквідність, FDV і market cap лише якщо API повернув відповідні поля. Це перевірка provider listing, а не аудит контракту чи гарантія безпеки.

Circulating supply та unlock schedule через ці endpoint-и не підтверджуються і відображаються як N/A. На цьому етапі CoinGlass, Coinalyze, Laevitas, TradingView data API, CoinGecko, CoinMarketCap, Tokenomist та CryptoRank не підключені: потрібні API credentials/умови доступу не налаштовані або не перевірені для цього статичного клієнта. Платні API, ключі чи секрети не додаються.

WHALE / EXCHANGE FLOW показує N/A: інтегрованого безключового джерела великих переказів з адресами, мережею, напрямком та timestamp немає. Futures liquidation statistics Gate.io — біржова метрика, не on-chain transfer. Віджет не виводить «продаж» із transfer-to-exchange і не вигадує whale data.

## Обмеження

Дані Gate.io — лише публічні біржові дані та можуть бути недоступні для окремого контракту. Gate.io може не повертати деякі поля/метрики; вони відображаються як N/A, а не нуль або випадкове значення. При недоступності API показується API ERROR та час останнього успішного оновлення. Віджет є статичним GitHub Pages проєктом і не надає інвестиційних рекомендацій.
