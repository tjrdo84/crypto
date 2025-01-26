import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Input } from "@/components/ui/input";
import { Slider } from "@/components/ui/slider";
import { Select, SelectTrigger, SelectContent, SelectItem } from "@/components/ui/select";
import { motion } from "framer-motion";
import axios from "axios";

const exchanges = [
  { name: "Binance", url: "https://api.binance.com/api/v3/ticker/price?symbol=" },
  { name: "KuCoin", url: "https://api.kucoin.com/api/v1/market/stats?symbol=" },
  { name: "Coinbase", url: "https://api.coinbase.com/v2/prices/" },
  { name: "Gate.io", url: "https://api.gate.io/api/v4/spot/tickers?currency_pair=" },
  { name: "Bitget", url: "https://api.bitget.com/api/spot/v1/market/tickers" },
];

export default function CryptoProfitApp() {
  const [investment, setInvestment] = useState(0);
  const [selectedCrypto, setSelectedCrypto] = useState("BTCUSDT");
  const [selectedFiat, setSelectedFiat] = useState("USD");
  const [priceMultiplier, setPriceMultiplier] = useState(1);
  const [prices, setPrices] = useState({});

  useEffect(() => {
    const fetchPrices = async () => {
      const cryptoPrices = {};

      for (const exchange of exchanges) {
        try {
          let response;

          if (exchange.name === "Binance") {
            response = await axios.get(`${exchange.url}${selectedCrypto}`);
            cryptoPrices[exchange.name] = parseFloat(response.data.price);
          } else if (exchange.name === "KuCoin") {
            response = await axios.get(`${exchange.url}${selectedCrypto}`);
            cryptoPrices[exchange.name] = parseFloat(response.data.data.last);
          } else if (exchange.name === "Coinbase") {
            response = await axios.get(`${exchange.url}${selectedCrypto}/spot`);
            cryptoPrices[exchange.name] = parseFloat(response.data.data.amount);
          } else if (exchange.name === "Gate.io") {
            response = await axios.get(`${exchange.url}${selectedCrypto}`);
            cryptoPrices[exchange.name] = parseFloat(response.data[0].last);
          } else if (exchange.name === "Bitget") {
            response = await axios.get(exchange.url);
            const pairData = response.data.data.find((d) => d.symbol === selectedCrypto);
            cryptoPrices[exchange.name] = pairData ? parseFloat(pairData.last) : null;
          }
        } catch (error) {
          console.error(`Error fetching data from ${exchange.name}:`, error);
          cryptoPrices[exchange.name] = null;
        }
      }

      setPrices(cryptoPrices);
    };

    fetchPrices();
  }, [selectedCrypto]);

  const calculateAveragePrice = () => {
    const validPrices = Object.values(prices).filter((price) => price !== null);
    return validPrices.length > 0
      ? validPrices.reduce((sum, price) => sum + price, 0) / validPrices.length
      : 0;
  };

  const adjustedPrice = calculateAveragePrice() * priceMultiplier;
  const coinsReceived = investment / calculateAveragePrice();
  const currentValue = coinsReceived * adjustedPrice;

  return (
    <div className="bg-gradient-to-br from-yellow-200 to-blue-200 min-h-screen flex items-center justify-center p-4">
      <Card className="w-full max-w-3xl shadow-2xl rounded-2xl p-6 bg-white">
        <CardContent>
          <h1 className="text-3xl font-bold text-center mb-4 text-blue-800">Crypto Profit Simulator</h1>
          <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
            <div>
              <label className="block text-lg font-medium text-blue-800">Investment Amount ({selectedFiat})</label>
              <Input
                type="number"
                className="mt-2 w-full rounded-lg border-blue-400 focus:ring-2 focus:ring-blue-300"
                placeholder="Enter amount"
                value={investment}
                onChange={(e) => setInvestment(e.target.value)}
              />
              <label className="block text-lg font-medium mt-4 text-blue-800">Select Cryptocurrency</label>
              <Select onValueChange={(value) => setSelectedCrypto(value)} value={selectedCrypto}>
                <SelectTrigger className="mt-2 w-full rounded-lg border-blue-400 focus:ring-2 focus:ring-blue-300">
                  {selectedCrypto}
                </SelectTrigger>
                <SelectContent>
                  {Object.keys(prices).map((crypto) => (
                    <SelectItem key={crypto} value={crypto}>
                      {crypto}
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
              <label className="block text-lg font-medium mt-4 text-blue-800">Select Fiat Currency</label>
              <Select onValueChange={(value) => setSelectedFiat(value)} value={selectedFiat}>
                <SelectTrigger className="mt-2 w-full rounded-lg border-blue-400 focus:ring-2 focus:ring-blue-300">
                  {selectedFiat}
                </SelectTrigger>
                <SelectContent>
                  {["USD", "EUR", "GBP", "JPY", "AUD"].map((fiat) => (
                    <SelectItem key={fiat} value={fiat}>
                      {fiat}
                    </SelectItem>
                  ))}
                </SelectContent>
              </Select>
            </div>
            <div>
              <label className="block text-lg font-medium text-blue-800">Adjust Price Multiplier</label>
              <Slider
                className="mt-4"
                defaultValue={[1]}
                min={0.1}
                max={5}
                step={0.1}
                onValueChange={(value) => setPriceMultiplier(value[0])}
              />
              <motion.div
                className="mt-6 p-4 bg-blue-50 rounded-xl shadow-md"
                initial={{ opacity: 0, y: 10 }}
                animate={{ opacity: 1, y: 0 }}
              >
                <h2 className="text-xl font-semibold text-blue-800">Simulation Results</h2>
                <p className="mt-2 text-blue-700">Coins Received: {coinsReceived.toFixed(4)}</p>
                <p className="mt-2 text-blue-700">Adjusted Price: {adjustedPrice.toFixed(2)} {selectedFiat}</p>
                <p className="mt-2 text-blue-700 font-bold">Current Value: {currentValue.toFixed(2)} {selectedFiat}</p>
              </motion.div>
            </div>
          </div>
        </CardContent>
      </Card>
    </div>
  );
}
