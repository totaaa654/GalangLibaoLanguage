require("dotenv").config();

const express = require("express");

const app = express();
const PORT = process.env.PORT || 3000;

// Serve the website from the "public" folder
app.use(express.static("public"));

/**
 * Tiny in-memory history (shared while server is running)
 * We'll store the last 5 translations.
 */
let history = [];

app.get("/api/translate", async (req, res) => {
  try {
    const text = (req.query.text || "").trim();
    const from = (req.query.from || "en").trim();
    const to = (req.query.to || "tl").trim();

    // Basic validation
    if (!text) {
      return res.status(400).json({ error: "Missing 'text' query parameter." });
    }
    if (!from || !to) {
      return res.status(400).json({ error: "Missing 'from' or 'to' language code." });
    }

    // Build MyMemory API request
    const url = new URL("https://api.mymemory.translated.net/get");
    url.searchParams.set("q", text);
    url.searchParams.set("langpair", `${from}|${to}`);

    // Optional: provide email to increase free daily quota
    // Put this in .env as MYMEMORY_EMAIL=you@example.com
    if (process.env.MYMEMORY_EMAIL) {
      url.searchParams.set("de", process.env.MYMEMORY_EMAIL);
    }

    const apiResp = await fetch(url);
    const data = await apiResp.json();

    // MyMemory typically returns translated text here:
    const translatedText = data?.responseData?.translatedText;

    if (!translatedText) {
      return res.status(502).json({
        error: "Translation API did not return translatedText.",
        raw: data
      });
    }

    // Save to history (latest first)
    history.unshift({
      from,
      to,
      input: text,
      output: translatedText,
      time: new Date().toISOString()
    });
    history = history.slice(0, 5);

    return res.json({
      translatedText,
      history
    });
  } catch (err) {
    return res.status(500).json({
      error: "Server error while translating.",
      details: err.message
    });
  }
});

app.get("/health", (req, res) => {
  res.json({ status: "ok", message: "Server is alive ✅" });
});

app.listen(PORT, () => {
  console.log(`✅ Server running at http://localhost:${PORT}`);
});