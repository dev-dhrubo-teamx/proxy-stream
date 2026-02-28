# Proxy Stream Setup IDX 

 1. we install cloudflared

```
wget https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64
```
2.
```
chmod +x cloudflared-linux-amd64
```
3.
```
mv cloudflared-linux-amd64 cloudflared
chmod +x cloudflared
```
# Then Go to cloudflare tunnel dashbord and get debian manuall tunnel run code and run in idx 

# Now we make our Main Proxy 

Step 1 
```
mkdir hls-proxy
cd hls-proxy
```
step 2 
```
npm init -y
npm install express node-fetch cors
```
step 3
 Make Server Js File server.js
 ```
const express = require("express");
const cors = require("cors");
const { Readable } = require("stream");

const app = express();
app.use(cors());

app.get("/proxy", async (req, res) => {
  const url = req.query.url;
  if (!url) return res.status(400).send("Missing url");

  // 🔑 forward Range header
  const headers = {
    "User-Agent": "Mozilla/5.0",
  };

  if (req.headers.range) {
    headers.Range = req.headers.range;
  }

  try {
    const response = await fetch(url, { headers });

    // 🔑 forward status (200 / 206)
    res.status(response.status);

    // 🔑 forward important headers
    [
      "content-type",
      "content-length",
      "content-range",
      "accept-ranges",
    ].forEach(h => {
      const v = response.headers.get(h);
      if (v) res.setHeader(h, v);
    });

    // stream body
    Readable.fromWeb(response.body).pipe(res);

  } catch (err) {
    res.status(500).send(err.message);
  }
});

app.listen(3000, () => {
  console.log("🚀 Proxy with Range support running on http://localhost:3000");
});
```
Step 4 
Start Proxy Server 
```
node server.js
```
