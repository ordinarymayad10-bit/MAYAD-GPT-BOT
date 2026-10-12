
export default async (request) => {
  if (request.method !== "POST") {
    return new Response("Mayad AI Bot is running!");
  }

  const env = process.env;
  const url = new URL(request.url);

  if (url.searchParams.get("secret") !== env.WEBHOOK_SECRET) {
    return new Response("Forbidden", { status: 403 });
  }

  const update = await request.json();
  const msg = update.message;

  if (!msg?.chat?.id || !msg.text) {
    return new Response("OK");
  }

  const chatId = msg.chat.id;
  const text = msg.text.trim();

  async function send(method, body) {
    return fetch(
      `https://api.telegram.org/bot${env.TELEGRAM_TOKEN}/${method}`,
      {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(body)
      }
    );
  }

  async function reply(text) {
    return send("sendMessage", {
      chat_id: chatId,
      text
    });
  }

  if (text === "/start") {
    await reply(
      "🤖 Welcome to Mayad AI!\n\n" +
      "💬 Chat: /chat তোমার প্রশ্ন\n" +
      "🎨 Photo: /image ছবির বর্ণনা"
    );
    return new Response("OK");
  }

  if (text.startsWith("/image ")) {
    await reply("ছবি তৈরির API পরের ধাপে সংযুক্ত করতে হবে।");
    return new Response("OK");
  }

  const question = text.startsWith("/chat ")
    ? text.slice(6).trim()
    : text;

  if (!env.HF_TOKEN) {
    await reply("AI চালু করতে HF_TOKEN সেট করতে হবে।");
    return new Response("OK");
  }

  try {
    const response = await fetch(
      "https://router.huggingface.co/v1/chat/completions",
      {
        method: "POST",
        headers: {
          Authorization: `Bearer ${env.HF_TOKEN}`,
          "Content-Type": "application/json"
        },
        body: JSON.stringify({
          model: env.HF_CHAT_MODEL || "Qwen/Qwen2.5-7B-Instruct",
          messages: [
            { role: "system", content: "বাংলায় সাহায্যকারী AI হও।" },
            { role: "user", content: question }
          ],
          max_tokens: 400
        })
      }
    );

    const data = await response.json();

    if (!response.ok) {
      await reply("AI API কাজ করছে না। Token ও model পরীক্ষা করো।");
    } else {
      await reply(
        data.choices?.[0]?.message?.content || "উত্তর পাওয়া যায়নি।"
      );
    }
  } catch {
    await reply("সংযোগে সমস্যা হয়েছে। পরে আবার চেষ্টা করো।");
  }

  return new Response("OK");
};
