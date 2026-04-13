import os
import json
import requests
from datetime import datetime

WEBHOOK_URL = os.environ["WECOM_WEBHOOK_URL"]
ANTHROPIC_API_KEY = os.environ["ANTHROPIC_API_KEY"]

def fetch_news():
    today = datetime.now().strftime("%Y-%m-%d")
    prompt = f"""Today is {today}. Search the web for today's top news across three categories: tech, finance, and world affairs. Return exactly 3 items per category (9 total).

Respond ONLY in this JSON format with no extra text:
[
  {{"category": "tech", "title": "...", "summary": "One sentence summary."}},
  {{"category": "finance", "title": "...", "summary": "One sentence summary."}},
  {{"category": "world", "title": "...", "summary": "One sentence summary."}}
]
Categories must be: tech / finance / world"""

    response = requests.post(
        "https://api.anthropic.com/v1/messages",
        headers={
            "x-api-key": ANTHROPIC_API_KEY,
            "anthropic-version": "2023-06-01",
            "content-type": "application/json",
        },
        json={
            "model": "claude-sonnet-4-20250514",
            "max_tokens": 1500,
            "tools": [{"type": "web_search_20250305", "name": "web_search"}],
            "messages": [{"role": "user", "content": prompt}],
        },
    )
    response.raise_for_status()
    data = response.json()

    text = "".join(b["text"] for b in data["content"] if b.get("type") == "text")
    text = text.replace("```json", "").replace("```", "").strip()
    start, end = text.index("["), text.rindex("]")
    return json.loads(text[start : end + 1])


def build_message(news_items):
    today = datetime.now().strftime("%Y-%m-%d %A")
    category_labels = {"tech": "🔬 Tech", "finance": "💹 Finance", "world": "🌍 World"}
    grouped = {"tech": [], "finance": [], "world": []}

    for item in news_items:
        cat = item.get("category", "world")
        if cat in grouped:
            grouped[cat].append(item)

    lines = [f"**📰 Daily News — {today}**\n"]
    for cat, label in category_labels.items():
        items = grouped.get(cat, [])
        if not items:
            continue
        lines.append(f"**{label}**")
        for item in items:
            lines.append(f"> **{item['title']}**")
            lines.append(f"> {item['summary']}")
            lines.append("")

    lines.append("_Powered by Claude AI · Auto-delivered every morning_")
    return "\n".join(lines)


def send_to_wecom(text):
    payload = {"msgtype": "markdown", "markdown": {"content": text}}
    response = requests.post(WEBHOOK_URL, json=payload)
    response.raise_for_status()
    result = response.json()
    if result.get("errcode") != 0:
        raise Exception(f"WeCom error: {result}")
    print("✅ News pushed successfully!")


if __name__ == "__main__":
    print("Fetching today's news...")
    news = fetch_news()
    print(f"Got {len(news)} news items.")
    message = build_message(news)
    send_to_wecom(message)
