# jayaraj
AI-Driven Customer Engagement Strategy for Astrotalk
Backend
1. Get the list of any 20 stocks along with their open price from polygon API and assign them a "refreshInterval" lying between 1-5 secs unique for each stock. Store this in a file in the backend.

2. The prices of each of these stocks should be updated with a random value every refreshInterval seconds, make sure that each stock is to be updated at it's own interval.


// backend/app.js

const express = require('express');
const axios = require('axios');
const fs = require('fs');

const app = express();
const PORT = 3001;

app.get('/initialize', async (req, res) => {
  try {
    const response = await axios.get('https://api.polygon.io/v2/snapshot/locale/us/markets/stocks/tickers?apiKey=YOUR_POLYGON_API_KEY');
    const stocks = response.data.tickers.slice(0, 20).map(stock => ({
      symbol: stock.ticker,
      openPrice: stock.day.open,
      refreshInterval: Math.floor(Math.random() * 5) + 1,
      currentPrice: stock.day.open, // Initial current price
    }));

    fs.writeFileSync('stocks.json', JSON.stringify(stocks));
    res.status(200).json({ message: 'Stocks initialized successfully' });
  } catch (error) {
    console.error('Error initializing stocks:', error.message);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});

app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});

3.Update Stock Prices:

Create a background process to update stock prices based on their refresh intervals.
// backend/updatePrices.js

const fs = require('fs');

const stocks = JSON.parse(fs.readFileSync('stocks.json', 'utf8'));

setInterval(() => {
  stocks.forEach(stock => {
    stock.currentPrice = stock.currentPrice + (Math.random() - 0.5); // Simulate price change
  });

  fs.writeFileSync('stocks.json', JSON.stringify(stocks));
}, 1000); // Update every second

4.Expose API to Fetch Stock Prices:

Create an API endpoint to fetch stock prices.

// backend/app.js

app.get('/stocks', (req, res) => {
  try {
    const stocks = JSON.parse(fs.readFileSync('stocks.json', 'utf8'));
    res.status(200).json(stocks);
  } catch (error) {
    console.error('Error fetching stocks:', error.message);
    res.status(500).json({ error: 'Internal Server Error' });
  }
});



Frontend (React)
1.Setup React Project:

Create a new React project using npx create-react-app frontend.
Navigate to the frontend directory.
2.Fetch N Stocks from Backend:

Use axios to fetch N stocks from the backend.

// frontend/src/App.js

import React, { useEffect, useState } from 'react';
import axios from 'axios';

function App() {
  const [stocks, setStocks] = useState([]);

  useEffect(() => {
    const fetchStocks = async () => {
      try {
        const response = await axios.get('http://localhost:3001/stocks');
        setStocks(response.data.slice(0, n));
      } catch (error) {
        console.error('Error fetching stocks:', error.message);
      }
    };

    fetchStocks();
  }, []);

  return (
    <div>
      <h1>Stocks List</h1>
      <ul>
        {stocks.map(stock => (
          <li key={stock.symbol}>
            {stock.symbol}: {stock.currentPrice.toFixed(2)}
          </li>
        ))}
      </ul>
    </div>
  );
}

export default App;


3.Implement Short Polling System:

Use setInterval to poll the backend every second for updated stock prices.

// frontend/src/App.js

useEffect(() => {
  const fetchStocks = async () => {
    try {
      const response = await axios.get('http://localhost:3001/stocks');
      setStocks(response.data.slice(0, n));
    } catch (error) {
      console.error('Error fetching stocks:', error.message);
    }
  };

  const intervalId = setInterval(fetchStocks, 1000);

  return () => clearInterval(intervalId);
}, [n]);


4.Run the Application:

Start both the backend and frontend servers.

# Terminal 1 (Backend)
cd backend
node app.js

# Terminal 2 (Backend - in a new terminal)
node updatePrices.js

# Terminal 3 (Frontend)
cd frontend
npm start
