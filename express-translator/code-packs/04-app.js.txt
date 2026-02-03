const form = document.getElementById("translateForm");
const textInput = document.getElementById("textInput");
const fromLang = document.getElementById("fromLang");
const toLang = document.getElementById("toLang");
const resultText = document.getElementById("resultText");
const statusText = document.getElementById("statusText");
const historyList = document.getElementById("historyList");
const translateBtn = document.getElementById("translateBtn");

function renderHistory(items) {
  historyList.innerHTML = "";
  items.forEach((item) => {
    const li = document.createElement("li");
    li.textContent = `[${item.from} → ${item.to}] "${item.input}" → "${item.output}"`;
    historyList.appendChild(li);
  });
}

form.addEventListener("submit", async (e) => {
  e.preventDefault();

  const text = textInput.value.trim();
  const from = fromLang.value;
  const to = toLang.value;

  if (!text) {
    statusText.textContent = "⚠️ Please type a message first.";
    resultText.textContent = "";
    return;
  }

  // UI: loading state
  translateBtn.disabled = true;
  statusText.textContent = "⏳ Translating… (Agent Linguist at work)";
  resultText.textContent = "";

  try {
    const url = `/api/translate?text=${encodeURIComponent(text)}&from=${encodeURIComponent(from)}&to=${encodeURIComponent(to)}`;
    const resp = await fetch(url);
    const data = await resp.json();

    if (!resp.ok) {
      statusText.textContent = "❌ Translation failed.";
      resultText.textContent = data.error || "Unknown error";
      return;
    }

    statusText.textContent = "✅ Done!";
    resultText.textContent = data.translatedText;

    if (Array.isArray(data.history)) {
      renderHistory(data.history);
    }
  } catch (err) {
    statusText.textContent = "❌ Network/server error.";
    resultText.textContent = err.message;
  } finally {
    translateBtn.disabled = false;
  }
});