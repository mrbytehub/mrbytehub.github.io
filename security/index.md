---
layout: page
title: 🛡️ Threat Intelligence & Security Alerts
permalink: /security/
---

Benvenuto nella sezione di Security Operational Intel. Qui vengono raccolte le analisi rapide e strutturate su vulnerabilità critiche (CVE) e campagne di minaccia attive sul territorio italiano.

> 📝 **Nota metodologica:** Per garantire una reazione rapida e tempestiva a fronte di decine di bollettini quotidiani, i report in questa sezione vengono elaborati e formattati con il supporto di strumenti di Intelligenza Artificiale (Custom Gemini Gem) sulla base di fonti ufficiali (CERT-AGID, ACN, D3Lab, ecc.) e successivamente revisionati.

---

### Ultimi Alert e Analisi

<table style="width:100%; border-collapse: collapse;">
  <thead>
    <tr style="background-color: #2f3542; color: white;">
      <th style="padding: 10px; text-align: left;">Data</th>
      <th style="padding: 10px; text-align: left;">Minaccia / Vulnerabilità</th>
      <th style="padding: 10px; text-align: left;">Tag</th>
    </tr>
  </thead>
  <tbody>
    {% for post in site.posts %}
      {% if post.categories contains 'security' %}
        <tr style="border-bottom: 1px solid #ddd;">
          <td style="padding: 10px; white-space: nowrap;"><strong>{{ post.date | date: "%Y-%m-%d" }}</strong></td>
          <td style="padding: 10px;"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></td>
          <td style="padding: 10px;">
            {% for tag in post.tags %}
              <span style="background-color: #f1f2f6; padding: 2px 6px; border-radius: 4px; font-size: 0.85em;">{{ tag }}</span>
            {% endfor %}
          </td>
        </tr>
      {% endif %}
    {% endfor %}
  </tbody>
</table>

---
📥 Vuoi integrare i nostri Indicatori di Compromissione? Scarica la [Lista IoC Aggiornata]({{ '/security/iocs/ioc-current.txt' | relative_url }}).
