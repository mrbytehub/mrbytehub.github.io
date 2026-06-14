---
layout: post
title: "Analisi: CVE-2026-47729 e CVE-2026-50012 in Squid Caching Proxy"
date: 2026-06-12
categories: security
tags: [cve, squid, proxy, rce, dos, information-disclosure]
---

# Rilevate Molteplici Vulnerabilità Critiche in Squid Caching Proxy

**Impatto:** Impatto sistemico Alto (65.0), con potenziale esecuzione di codice arbitrario, Denial of Service e divulgazione di informazioni sensibili.

**Vettore:** Network (Rete)

### 1. Sintesi Tecnica
L'Agenzia per la Cybersicurezza Nazionale (ACN) e lo CSIRT Italia hanno pubblicato un alert relativo a due nuove vulnerabilità di sicurezza che colpiscono Squid, il noto software open source utilizzato diffusamente come caching proxy. I difetti critici sono identificati come segue:

* **CVE-2026-47729** e **CVE-2026-50012**: Consentono impatti multipli e severi a seconda della configurazione del servizio.

Le tipologie di attacco associate a tali vulnerabilità includono:
* **Arbitrary Code Execution (RCE):** Un attaccante potrebbe eseguire codice non autorizzato con i privilegi del processo Squid.
* **Denial of Service (DoS):** Sfruttamento di anomalie per causare il crash del servizio, interrompendo la navigazione della rete interna.
* **Information Disclosure / Leakage:** Accesso non autorizzato ad informazioni sensibili e riservate transitanti o residenti sulle istanze affette.

L'effettivo sfruttamento delle vulnerabilità è strettamente subordinato alla specifica configurazione del modulo proxy indicata nei bollettini ufficiali di sicurezza del vendor.

### 2. Check di Perimetro (Come capire se sei colpito)
Per determinare l'esposizione della propria infrastruttura aziendale, occorre verificare i seguenti elementi:

* **Versioni Vulnerabili:** Squid, in tutte le versioni precedenti alla release **7.6**.
* **Verifica della Versione in uso:** Eseguire l'ispezione della versione binaria attualmente attiva sul server.
* **Analisi dei Log:** Ispezionare `/var/log/squid/access.log` e `/var/log/squid/cache.log` alla ricerca di arresti anomali ripetuti del demone o richieste HTTP malformate insolite.

### 3. Strategia di Remediation
Le azioni di mitigazione e remediation devono seguire un ordine cronologico prioritario:

1. **Aggiornamento Software:** Si raccomanda di aggiornare tempestivamente Squid all'ultima release stabile disponibile (**versione 7.6 o successive**), distribuita ufficialmente dal vendor.
2. **Restrizione degli Accessi:** Qualora l'aggiornamento non fosse immediatamente applicabile, limitare l'accesso alla porta del proxy (tipicamente `3128`) esclusivamente a subnet fidate mediante regole di firewalling (iptables/nftables) o ACL interne.
3. **Validazione della Configurazione:** Verificare i parametri del file `squid.conf` per assicurarsi di disabilitare funzionalità sperimentali o non necessarie che potrebbero attivare le condizioni di vulnerabilità descritte nel bollettino.

### 4. Distribuzioni Enterprise Tracker
* **Debian:** [Security Tracker CVE-2026-47729](https://security-tracker.debian.org/tracker/CVE-2026-47729) | [Security Tracker CVE-2026-50012](https://security-tracker.debian.org/tracker/CVE-2026-50012)
* **Ubuntu:** [USN / CVE Tracker CVE-2026-47729](https://ubuntu.com/security/CVE-2026-47729) | [USN / CVE Tracker CVE-2026-50012](https://ubuntu.com/security/CVE-2026-50012)
* **Red Hat:** [Red Hat CVE Portal CVE-2026-47729](https://access.redhat.com/security/cve/CVE-2026-47729) | [Red Hat CVE Portal CVE-2026-50012](https://access.redhat.com/security/cve/CVE-2026-50012)

### 5. Fonti e Referenze (Sitografia)
* **Fonte Primaria:** [[ACN] Rilevate vulnerabilità in Squid](https://www.acn.gov.it/portale/web/guest/-/rilevate-vulnerabilita-in-squid)
* **Analisi Correlata:** [[OSS-Security] Open Source Security Mailing List Reference](https://seclists.org/oss-sec/2026/q2/896)

### 6. Indicatori di Compromissione (IoC)
| Tipo | Valore | Note |
| :--- | :--- | :--- |
| Vulnerabilità | CVE-2026-47729 | Identificatore difetto Squid (RCE/DoS/Info Disclosure) |
| Vulnerabilità | CVE-2026-50012 | Identificatore difetto Squid (RCE/DoS/Info Disclosure) |

**Comando rapido di verifica / Query di Hunting:**
Per verificare lo stato del servizio o ispezionare i log alla ricerca di anomalie strutturate, eseguire il seguente comando da terminale:

```bash
squid -v 2>&1 | grep -i "version"
```