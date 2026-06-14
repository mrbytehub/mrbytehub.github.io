---
layout: null
---
<!DOCTYPE html>
<html lang="it">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Dashboard Sicurezza</title>
    <style>
        body { background-color: #0f141c !important; color: #e6edf3 !important; font-family: sans-serif; padding: 20px; }
        .container { max-width: 900px; margin: 0 auto; background: #161b22; padding: 30px; border-radius: 8px; border: 1px solid #30363d; }
        table { width: 100%; border-collapse: collapse; margin-top: 20px; color: #e6edf3; }
        th { background: #21262d; padding: 12px; border: 1px solid #30363d; }
        td { padding: 12px; border: 1px solid #30363d; }
        a { color: #58a6ff; }
    </style>
</head>
<body>
    <div class="container">
        <h1>🛡️ Dashboard</h1>
        <table>
            <thead>
                <tr><th>Data</th><th>Minaccia</th><th>Tag</th></tr>
            </thead>
            <tbody>
                {% for post in site.posts %}
                  {% if post.categories contains 'security' %}
                    <tr>
                        <td>{{ post.date | date: "%Y-%m-%d" }}</td>
                        <td><a href="{{ post.url | relative_url }}">{{ post.title }}</a></td>
                        <td>{{ post.tags | join: ", " }}</td>
                    </tr>
                  {% endif %}
                {% endfor %}
            </tbody>
        </table>
    </div>
</body>
</html>
