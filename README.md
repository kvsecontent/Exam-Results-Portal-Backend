// server.js
const express = require('express');
const cors = require('cors');
const axios = require('axios');
require('dotenv').config();

const app = express();
const PORT = process.env.PORT || 3000;

// CORS Configuration
const corsOptions = {
  origin: [
    'https://yourusername.github.io', // Your GitHub Pages URL
    'http://localhost:3000', // Local development
    'https://your-custom-domain.com' // Optional custom domain
  ],
  methods: ['GET'],
  allowedHeaders: ['Content-Type']
};

app.use(cors(corsOptions));

// Exam Results Endpoint
app.get('/api/exam-results', async (req, res) => {
  try {
    // Google Sheets API request
    const response = await axios.get(`https://sheets.googleapis.com/v4/spreadsheets/${process.env.GOOGLE_SHEET_ID}/values/Sheet1!A1:Z100`, {
      params: {
        key: process.env.GOOGLE_SHEETS_API_KEY
      }
    });

    // Process and sanitize data
    const rawData = response.data.values || [];
    const headers = rawData[0] || [];
    const results = rawData.slice(1).map(row => 
      headers.reduce((obj, header, index) => {
        obj[header] = row[index] || '';
        return obj;
      }, {})
    );

    // Send processed results
    res.json({
      status: 'success',
      data: results
    });
  } catch (error) {
    console.error('Error fetching exam results:', error.response ? error.response.data : error.message);
    res.status(500).json({
      status: 'error',
      message: 'Unable to fetch exam results'
    });
  }
});

// Health Check Endpoint
app.get('/health', (req, res) => {
  res.status(200).json({ status: 'healthy' });
});

// Start Server
app.listen(PORT, () => {
  console.log(`Backend proxy server running on port ${PORT}`);
});
