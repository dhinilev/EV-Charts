# EV Charts

Candlestick charts for NIFTY / SENSEX indices, index option premiums and NSE stocks, using the HDFC Sky Open API.
*An Erulenkattil Veettil Group · by Valerian*

## What it does

- **Index** – NIFTY 50, NIFTY BANK, SENSEX, FIN NIFTY, MIDCAP SELECT, BANKEX, INDIA VIX
- **Options** – pick underlying → expiry → strike (◀ ▶ steppers, ATM button) → CE / PE and chart the premium
- **Stocks** – search any NSE stock; the last 10 stay as quick chips
- Timeframes 1m, 3m, 5m, 15m, 30m, 1h, D, W (HDFC Sky serves 1-minute and daily; the rest are built on the phone)
- EMA 9, EMA 21, session VWAP, previous-day high / low / close lines, volume
- Live refresh every 20 s while the market is open
- Pinch to zoom, drag to scroll, long-press for crosshair, double-tap to jump to the latest candle
- **Buyers vs sellers (v1.1):** every candle body is split into buyer lots (green, from the bottom) and seller lots
  (red, from the top). A pane under the chart shows buyer bars ▲, seller bars ▼ and the cumulative delta (CVD) line.
  The lots box shows `B · S · Δ` for the latest candle, or for any candle under the crosshair, plus the live
  bid/ask pressure of the order book.
- **OI strip (v1.1):** a thin coloured strip under the candles. On options it shows writers adding, writers exiting,
  buyers adding or buyers exiting. On index charts it shows short buildup, short covering, long buildup or long
  unwinding of the near-month future. It works on past candles too.
- Read-only: EV Charts never places orders

## One-time setup on HDFC Sky

1. Log in at **developer.hdfcsky.com** → **My Apps** → **+ Create**.
2. **Redirect URL:** `https://evcharts.app/callback` (any https URL works, but it must be identical in the app's Settings).
3. **APIs in this app:** keep *Trading API* selected.
4. **Primary Static IP:** the form makes this mandatory (SEBI rule). See *Static IP note* below.
5. Click **Create**, then **activate** the app in My Apps (new apps start deactivated). Copy the **API Key** and **API Secret**.

## In the app

1. Open EV Charts → ⚙ Settings → paste the API Key, API Secret and Redirect URL → **Save keys**.
2. The HDFC Sky login page opens → Client ID → OTP. You are returned to the app automatically.
3. The contract list (~17 MB) downloads once a day so new expiries and strikes appear.
4. Log in again each trading day (HDFC Sky sessions are daily).

## Build

Upload this folder to the GitHub repo. The workflow in `.github/workflows/build-apk.yml` builds on every push to
`main`; download **EV-Charts-apk** from the run's Artifacts. The APK is signed with `app/ev-charts.keystore` so each new
build installs over the previous one. Keep the repo private, because this key file is stored in it.

## How buyers and sellers are separated

The app listens to HDFC Sky's live tick feed. Each new trade is classified against the order book it hit:

- traded at or above the best ask → a buyer was in a hurry → **buyer lots**
- traded at or below the best bid → a seller was in a hurry → **seller lots**
- in between → the price direction decides (up = buyer, down = seller)

Index charts take the split from the near-month **NIFTY / SENSEX future**, because the index itself does not trade.
The split is collected only while the app is open and logged in, from the moment you open it each day; earlier
candles show normally. What was collected is saved, so closing and reopening the app the same day keeps it.

## Static IP note

SEBI's API rules make HDFC Sky ask for a static IP. A phone on mobile data or home Wi-Fi does not have one. If HDFC Sky
enforces the IP on market-data calls as well as orders, chart requests will fail with an HTTP error, which the app
shows on the chart. In that case the requests have to go through a small server or VPN that has a static IP.

## Files

| Path | What |
|---|---|
| `app/src/main/assets/www/` | Chart screen (HTML/JS), chart engine `chart.js`, indicators `data.js` |
| `app/src/main/java/.../HdfcSky.kt` | Login token exchange, candles, LTP |
| `app/src/main/java/.../ContractMaster.kt` | Contract-list download and lookup |
| `app/src/main/java/.../LoginActivity.kt` | HDFC Sky login page and redirect capture |
| `app/src/main/java/.../SecureStore.kt` | Keystore-encrypted storage for keys and session |

Opening `app/src/main/assets/www/index.html` in a desktop browser runs the UI with demo data.
