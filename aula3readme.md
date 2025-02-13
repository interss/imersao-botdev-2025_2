@@ -1,9 +1,13 @@
const axios = require("axios");
const crypto = require("crypto");

const SYMBOL = "BTCUSDT";
const QUANTITY = "0.001";
const PERIOD = 14;

const API_URL = "https://api.binance.com";//"https://testnet.binance.vision";
const API_URL = "https://testnet.binance.vision";//"https://api.binance.com"
const API_KEY = "XXXXX";
const SECRET_KEY = "XXXXXX";

function averages(prices, period, startIndex) {
    let gains = 0, losses = 0;
@@ -41,6 +45,34 @@ function RSI(prices, period){
    return 100 - (100 / (1 + rs));
}

async function newOrder(symbol, quantity, side){
    const order = { symbol, quantity, side };
    order.type = "MARKET";
    order.timestamp = Date.now();
    const signature = crypto
        .createHmac("sha256", SECRET_KEY)
        .update(new URLSearchParams(order).toString())
        .digest("hex");
    order.signature = signature;
    try{
        const {data} = await axios.post(
            API_URL + "/api/v3/order", 
            new URLSearchParams(order).toString(), 
            {
                headers: { "X-MBX-APIKEY": API_KEY }
            }
        )
        console.log(data);
    }
    catch(err){
        console.error(err.response.data);
    }
}
let isOpened = false;

async function start() {
@@ -54,13 +86,16 @@ async function start() {
    const prices = data.map(k => parseFloat(k[4]));
    const rsi = RSI(prices, PERIOD);
    console.log("RSI: " + rsi);
    console.log("Já comprei? " + isOpened);

    if (rsi < 30 && isOpened === false) {
        console.log("sobrevendido, hora de comprar");
        isOpened = true;
        newOrder(SYMBOL, QUANTITY, "BUY");
    }
    else if (rsi > 70 && isOpened === true) {
        console.log("sobrecomprado, hora de vender");
        newOrder(SYMBOL, QUANTITY, "SELL");
        isOpened = false;
    }
    else
