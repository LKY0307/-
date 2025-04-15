# -import { useState } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

const generateRandomPrice = (price) => {
  const change = (Math.random() * 0.1 - 0.05) * price;
  return Math.round((price + change) * 100) / 100;
};

const initialStockList = [
  { name: "台積電", code: "2330", price: 800, industry: "半導體" },
  { name: "統一超商", code: "2912", price: 310, industry: "零售業" },
  { name: "長榮", code: "2603", price: 150, industry: "貨運航運" },
  { name: "國泰金", code: "2882", price: 53, industry: "金融保險" },
  { name: "聯發科", code: "2454", price: 1200, industry: "IC設計" },
  { name: "群創", code: "3481", price: 13, industry: "面板" },
];

export default function App() {
  const [cash, setCash] = useState(10000);
  const [portfolio, setPortfolio] = useState([]);
  const [stockList, setStockList] = useState(initialStockList);

  const handleBuy = (stock, quantity) => {
    const totalCost = stock.price * quantity;
    if (cash >= totalCost && quantity > 0) {
      setCash(cash - totalCost);
      const existing = portfolio.find((item) => item.code === stock.code);
      if (existing) {
        existing.quantity += quantity;
        existing.totalCost += totalCost;
        setPortfolio([...portfolio]);
      } else {
        setPortfolio([
          ...portfolio,
          {
            name: stock.name,
            code: stock.code,
            price: stock.price,
            industry: stock.industry,
            quantity,
            totalCost,
          },
        ]);
      }
    } else {
      alert("金額不足或數量錯誤");
    }
  };

  const handleSell = (stock, quantity) => {
    const owned = portfolio.find((item) => item.code === stock.code);
    if (owned && quantity > 0 && quantity <= owned.quantity) {
      const saleValue = stock.price * quantity;
      setCash(cash + saleValue);
      owned.quantity -= quantity;
      owned.totalCost = Math.round((owned.totalCost * (owned.quantity / (owned.quantity + quantity))) * 100) / 100;
      const newPortfolio = owned.quantity > 0 ? [...portfolio] : portfolio.filter(item => item.code !== stock.code);
      setPortfolio(newPortfolio);
    } else {
      alert("數量錯誤或你沒有這麼多股票");
    }
  };

  const updatePrices = () => {
    setStockList(stockList.map(stock => ({
      ...stock,
      price: generateRandomPrice(stock.price)
    })));
  };

  return (
    <div className="p-4 max-w-3xl mx-auto">
      <h1 className="text-3xl font-bold mb-4">模擬股神 APP</h1>
      <p className="mb-2">目前模擬資金：<strong>${cash.toLocaleString()} 元</strong></p>
      <Button onClick={updatePrices} className="mb-4">📈 模擬更新今日股價</Button>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
        {stockList.map((stock) => (
          <Card key={stock.code}>
            <CardContent className="p-4">
              <div className="text-xl font-semibold">
                {stock.name} ({stock.code})
              </div>
              <div>產業：{stock.industry}</div>
              <div>股價：{stock.price} 元</div>
              <BuyStockForm stock={stock} onBuy={handleBuy} />
              <SellStockForm stock={stock} onSell={handleSell} />
            </CardContent>
          </Card>
        ))}
      </div>

      <div className="mt-8">
        <h2 className="text-2xl font-bold mb-2">我的投資組合</h2>
        {portfolio.length === 0 ? (
          <p>你還沒有買任何股票。</p>
        ) : (
          <ul className="space-y-2">
            {portfolio.map((stock) => (
              <li key={stock.code} className="border p-2 rounded-md">
                {stock.name} ({stock.code}) - {stock.quantity} 股，共花費 {stock.totalCost.toLocaleString()} 元（產業：{stock.industry}）
              </li>
            ))}
          </ul>
        )}
      </div>
    </div>
  );
}

function BuyStockForm({ stock, onBuy }) {
  const [quantity, setQuantity] = useState(0);

  return (
    <div className="mt-2 flex items-center gap-2">
      <Input
        type="number"
        value={quantity}
        min={0}
        onChange={(e) => setQuantity(Number(e.target.value))}
        className="w-20"
      />
      <Button onClick={() => onBuy(stock, quantity)}>買進</Button>
    </div>
  );
}

function SellStockForm({ stock, onSell }) {
  const [quantity, setQuantity] = useState(0);

  return (
    <div className="mt-2 flex items-center gap-2">
      <Input
        type="number"
        value={quantity}
        min={0}
        onChange={(e) => setQuantity(Number(e.target.value))}
        className="w-20"
      />
      <Button onClick={() => onSell(stock, quantity)} variant="secondary">賣出</Button>
    </div>
  );
}
