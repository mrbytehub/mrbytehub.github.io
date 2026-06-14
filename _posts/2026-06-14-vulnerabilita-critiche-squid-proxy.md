---
layout: post
title: "Analisi: Vulnerabilità critiche in Squid Proxy (CVE-2026-47729, CVE-2026-50012)"
date: 2026-06-14
categories: security
tags: [cve, squid, proxy, rce, dos]
---

# Vulnerabilità critiche in Squid Proxy

**Impatto:** Critico (Alto). Rischio di esecuzione di codice arbitrario (RCE), Denial of Service (DoS) e Disclosure di informazioni sensibili.
**Vettore:** Network (servizio proxy esposto o accessibile internamente).

### 1. Sintesi Tecnica
Il CSIRT Italia ha segnalato la presenza di vulnerabilità critiche nel software di caching proxy open source Squid. Le falle interessano le versioni precedenti alla 7.6. La natura delle vulnerabilità consente a un attaccante remoto, in determinate condizioni di configurazione, di compromettere l'integrità e la disponibilità del servizio, potendo arrivare all'esecuzione di codice arbitrario sul sistema host.

### 2. Check di Perimetro (Come capire se sei colpito)
* **Versioni Software:** Verificare se la versione installata di Squid è antecedente alla 7.6.
* **Comando di verifica (Linux):**
    ```bash
    squid -v
    ```
* **Controllo Log:** Monitorare i log di sistema (`/var/log/squid/cache.log` o `/var/log/squid/access.log`) per attività anomale, richieste HTTP malformate o tentativi di crash improvvisi del servizio.

### 3. Strategia di Remediation
* **Patching:** Aggiornare immediatamente Squid alla versione 7.6 o superiore tramite il repository della propria distribuzione o compilando dai sorgenti ufficiali.
* **Hardening Temporaneo:** Se l'aggiornamento immediato non è possibile, limitare l'accesso al proxy tramite firewall (ACL), restringendo le connessioni unicamente agli indirizzi IP autorizzati (White-listing).
* **Revisione Configurazioni:** Verificare le direttive di configurazione in `squid.conf` per limitare le funzionalità non necessarie che potrebbero essere sfruttate come vettore per le vulnerabilità citate.

### 4. Distribuzioni Enterprise Tracker
* **Debian:** [Security Tracker CVE-2026-47729](https://security-tracker.debian.org/tracker/CVE-2026-47729) / [CVE-2026-50012](https://security-tracker.debian.org/tracker/CVE-2026-50012)
* **Ubuntu:** [CVE-2026-47729 Vulnerability Tracker](https://ubuntu.com/security/CVE-2026-47729)
* **Red Hat:** [Red Hat CVE-2026-47729 Portal](https://access.redhat.com/security/cve/CVE-2026-47729)

### 5. Indicatori di Compromissione (IoC)
Al momento non sono stati rilasciati IoC specifici (IP C2 o hash di malware) poiché si tratta di vulnerabilità di tipo software. Si consiglia di monitorare il traffico in ingresso verso la porta del proxy (default 3128) per picchi anomali di richieste `GET`/`POST`.

**Comando rapido di verifica / Query di Hunting:**
```bash
# Verifica rapida della versione installata su sistemi Debian/Ubuntu
dpkg -l | grep squid

# Ricerca nel log di accessi sospetti (esempio: caratteri speciali o path traversal)
grep -E "\.\./|\.\.\\" /var/log/squid/access.log