---
layout: post
title: "Analisi: Vulnerabilità multiple in Squid Caching Proxy"
date: 2026-06-12
categories: security
tags: [squid, vulnerability, dos, information-disclosure, rce, proxy]
---

# Rilevate molteplici vulnerabilità in Squid Caching Proxy

**Impatto:** Grave compromissione della disponibilità del servizio, potenziale divulgazione di informazioni sensibili ed esecuzione di codice arbitrario (Impatto Sistemico Alto: 65.0).

**Vettore:** Network (Rete / Richieste Proxy strutturate ad hoc)

### 1. Sintesi Tecnica

In base ai recenti bollettini di sicurezza pubblicati dall'Agenzia per la Cybersicurezza Nazionale (ACN) e dal CSIRT Italia, sono state identificate due nuove vulnerabilità significative all'interno del software open source Squid, ampiamente utilizzato come caching proxy a livello aziendale.

I vettori d'attacco legati a queste falle espongono i sistemi alle seguenti tipologie di minaccia:
* **Information Disclosure:** Un utente malintenzionato potrebbe essere in grado di accedere a dettagli riservati e dati sensibili transitanti o memorizzati nel proxy.
* **Denial of Service (DoS):** Sfruttando la vulnerabilità, un attaccante remoto può causare il crash o il blocco dei processi di Squid, compromettendo totalmente la disponibilità della connettività o del caching per i client interni.
* **Arbitrary Code Execution:** Una delle componenti affette potrebbe consentire l'esecuzione di codice non autorizzato sulla macchina server ospitante, qualora si verifichino specifiche condizioni di configurazione.

L'impatto complessivo è classificato come **Alto** a causa del ruolo centrale che i server proxy ricoprono nelle infrastrutture di rete enterprise come punti di ispezione e smistamento del traffico.

### 2. Check di Perimetro (Come capire se sei colpito)

Per determinare l'esposizione della propria infrastruttura a queste specifiche falle, è necessario effettuare le seguenti verifiche interne:

* **Controllo della Versione:** Verificare la versione di Squid installata sui propri apparati operativi o server Linux. Risultano vulnerabili tutte le release storiche e correnti **precedenti alla versione 7.6**.
* **Verifica della Configurazione:** Le vulnerabilità si manifestano esclusivamente nel caso in cui i prodotti elencati siano configurati secondo le specifiche modalità descritte all'interno del bollettino ufficiale del vendor (es. moduli di parsing o regole di ACL attive).
* **Ispezione dei Log:** Monitorare i file di log di Squid (tipicamente posizionati in `/var/log/squid/access.log` e `/var/log/squid/cache.log`) per rilevare eventuali anomalie sistematiche, crash inaspettati del demone `squid` o richieste HTTP strutturate in modo anomalo.

### 3. Strategia di Remediation

Si raccomanda l'applicazione immediata delle seguenti contromisure in ordine cronologico:

1. **Aggiornamento Software:** Procedere tempestivamente all'aggiornamento di Squid portando il software all'ultima release stabile rilasciata dal vendor (**versione 7.6 o successive**), oppure applicare i pacchetti di sicurezza distribuiti dal proprio fornitore di sistema operativo (backporting delle patch).
2. **Review delle Configurazioni:** Qualora l'aggiornamento non fosse immediatamente applicabile, consultare il bollettino tecnico del vendor per individuare le direttive di configurazione insicure e disabilitarle temporaneamente (es. restrizioni sulle funzionalità di caching avanzate o specifici helper di autenticazione).
3. **Hardening di Rete:** Limitare l'accesso alle porte di ascolto del proxy (es. porta predefinita `3128`) consentendolo unicamente ai segmenti di rete interna o agli host autorizzati tramite regole di firewalling (iptables/nftables) o Network Security Groups.

### 4. Distribuzioni Enterprise Tracker

* **Debian:** [Security Tracker CVE-Squid](https://security-tracker.debian.org/tracker/source-package/squid)
* **Ubuntu:** [USN / CVE Tracker Squid](https://ubuntu.com/security/cves?q=squid)
* **Red Hat:** [Red Hat CVE Portal](https://access.redhat.com/security/security-updates/#/cve)

### 5. Fonti e Referenze (Sitografia)

* **Fonte Primaria:** [[ACN] Rilevate vulnerabilità in Squid](https://www.acn.gov.it/portale/web/guest/-/rilevate-vulnerabilita-in-squid)
* **Analisi Correlata:** [[CSIRT Toscana] Rilevate vulnerabilità in Squid (AL04/260612/CSIRT-ITA)](https://csirt.regione.toscana.it/rilevate-vulnerabilita-in-squid-al04-260612-csirt-ita/)

### 6. Indicatori di Compromissione (IoC)

| Tipo | Valore | Note |
| :--- | :--- | :--- |
| Versione Software | Squid < 7.6 | Versioni del software affette da falle multiple |

**Comando rapido di verifica / Query di Hunting:**

Per verificare rapidamente la versione di Squid attualmente in esecuzione sul sistema locale Linux, utilizzare il seguente comando da terminale:

```bash
squid -v | grep -i "version"