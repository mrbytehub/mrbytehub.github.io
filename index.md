---
layout: null
title: "mrbytehub | Threat Intel Dashboard"
---

<style>
    * { box-sizing: border-box; }
    body { 
        background-color: #0f141c !important; 
        color: #e6edf3 !important; 
        font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Helvetica, Arial, sans-serif; 
        padding: 40px 20px; 
        margin: 0;
    }
    .container { 
        max-width: 900px; 
        margin: 0 auto; 
        background: #161b22; 
        padding: 30px; 
        border-radius: 8px; 
        border: 1px solid #30363d; 
        box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
    }
    h1 { color: #e6edf3; border-bottom: 2px solid #30363d; padding-bottom: 10px; margin-top: 0; }
    table { width: 100%; border-collapse: collapse; margin-top: 20px; table-layout: fixed; }
    th { background: #21262d; padding: 12px; border: 1px solid #30363d; text-align: left; color: #e6edf3; }
    td { padding: 12px; border: 1px solid #30363d; word-wrap: break-word; color: #e6edf3; }
    a { color: #58a6ff; text-decoration: none; }
    a:hover { text-decoration: underline; }
    .tag { background: #282e38; padding: 3px 8px; border-radius: 4px; font-size: 0.8rem; border: 1px solid #30363d; color: #8b949e; }
</style>

<div class="container">
    <h1>🛡️ Threat Intelligence Dashboard</h1>
    <a href="{{ '/feed.xml' | relative_url }}" target="_blank" style="font-size: 0.9em; color: #ff6600; text-decoration: none;">
       🌐 Abbonati al Feed RSS
    </a>
    <p>Analisi rapide e strutturate su vulnerabilità critiche (CVE) e campagne di minaccia attive.</p>

    <table>
        <thead>
            <tr>
                <th style="width: 20%;">Data</th>
                <th style="width: 55%;">Minaccia / Vulnerabilità</th>
                <th style="width: 25%;">Tag</th>
            </tr>
        </thead>
        <tbody>
            {% for post in site.posts %}
              {% if post.categories contains 'security' %}
                <tr>
                    <td><strong>{{ post.date | date: "%Y-%m-%d" }}</strong></td>
                    <td><a href="{{ post.url | relative_url }}">{{ post.title }}</a></td>
                    <td>
                        {% for tag in post.tags %}
                          <span class="tag">{{ tag }}</span>
                        {% endfor %}
                    </td>
                </tr>
              {% endif %}
            {% endfor %}
        </tbody>
    </table>
</div>
