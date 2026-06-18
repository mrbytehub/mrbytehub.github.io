---
layout: post
title: "Analisi: CVE-2026-42530 e CVE-2026-42055"
date: 2026-06-18
categories: security
tags:
  - web
  - cve
  - rce
  - dos
---

# Vulnerabilità Critiche in NGINX Open Source e Plus (CVE-2026-42530, CVE-2026-42055)

**Impatto:** Denial of Service (DoS) critico con potenziale esecuzione di codice remoto (RCE) tramite exploit di memoria (Use-After-Free e Heap Buffer Overflow).
**Vettore:** Network (attacco remoto non autenticato via protocolli HTTP/3 e HTTP/2 / gRPC).

### 1. Sintesi Tecnica
F5 ha rilasciato aggiornamenti di sicurezza per correggere due gravi falle logiche e di gestione della memoria all'interno dei processi worker di NGINX Open Source e NGINX Plus:

* **CVE-2026-42530 (Use-After-Free):** Risiede nel modulo `ngx_http_v3_module`. Un attaccante remoto non autenticato può inviare una sessione HTTP/3 (QUIC) appositamente contraffatta per forzare la riapertura di uno stream dell'encoder QPACK. Questa operazione induce il worker process a fare riferimento a un'area di memoria precedentemente liberata, innescando una corruzione della memoria (CWE-416) che causa il crash immediato del processo e, in assenza di protezioni attive come l'ASLR, l'esecuzione di codice arbitrario.
* **CVE-2026-42055 (Heap-based Buffer Overflow):** Colpisce i moduli di reverse proxy `ngx_http_proxy_v2_module` e `ngx_http_grpc_module`. Si verifica quando NGINX è configurato per inoltrare il traffico verso un server upstream tramite HTTP/2 o gRPC. Se l'applicazione riceve intestazioni (header) eccezionalmente grandi nel momento in cui viene stabilita la connessione upstream, si verifica un overflow nel segmento Heap (CWE-122). Ciò provoca l'interruzione del worker process o la potenziale esecuzione di codice remoto (RCE).

### 2. Check di Perimetro ed Esposizione
Un sistema è esposto ed effettivamente vulnerabile se rispetta i seguenti criteri specifici:

* **Versioni Software Affette:**
    * *NGINX Open Source (Mainline):* Versioni da 1.31.0 a 1.31.1 (per CVE-2026-42530) e da 1.13.10 a 1.31.1 (per CVE-2026-42055).
    * *NGINX Open Source (Stable):* Versioni precedenti alla 1.30.3 (per CVE-2026-42055).
    * *NGINX Plus:* Versioni R33 fino a R36 (prima della patch P6) e branch 37.x (dalla 37.0.0 alla 37.0.1).
* **Condizioni di Configurazione (CVE-2026-42530):** Il modulo HTTP/3 QUIC deve essere attivamente configurato all'interno dei blocchi di `server {}`. Esempio sintattico esposto:
    ```nginx
    listen 443 quic;
    http3 on;
    ```
* **Condizioni di Configurazione (CVE-2026-42055):** Devono coesistere contemporaneamente le seguenti tre direttive non di default nel file `nginx.conf`:
    1. Uso di `proxy_http_version 2;` o `grpc_pass ...;` per il proxying verso l'upstream.
    2. La direttiva `ignore_invalid_headers` impostata su `off;`.
    3. La direttiva `large_client_header_buffers` configurata con una dimensione per buffer superiore a 2 Megabyte (il default è 4 buffer da 8 Kilobyte, configurazione che blocca l'attacco originario).

### 3. Strategia di Remediation e Hardening
L'applicazione tempestiva dei rimedi segue una gerarchia di interventi software e di configurazione:

* **Aggiornamento di Sicurezza (Risoluzione Definitiva):**
    * Aggiornare NGINX Open Source alla versione **1.31.2** (Mainline) o **1.30.3** (Stable).
    * Aggiornare NGINX Plus alla versione **R36 P6** o **37.0.2.1** (LTS).
* **Mitigazione Temporanea per CVE-2026-42530:** Se l'aggiornamento non è applicabile immediatamente, disabilitare il supporto HTTP/3 rimuovendo la configurazione QUIC dai file di configurazione del sito:
    * Rimuovere o commentare la riga `http3 on;` e il parametro `quic` dalla direttiva `listen`.
    * Eseguire un reload sicuro del demone: `nginx -s reload`.
* **Mitigazione Temporanea per CVE-2026-42055:** Modificare la configurazione per neutralizzare l'overflow:
    * Rimuovere o impostare su `on` la direttiva `ignore_invalid_headers` (valore predefinito).
    * Ridurre la dimensione massima consentita in `large_client_header_buffers` a un valore strettamente inferiore a `2m`.

### 4. Riferimenti Ufficiali e Bollettini Vendor
* **Riferimento Ufficiale:** [MITRE CVE-2026-42530](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-42530)
* **Riferimento Ufficiale:** [MITRE CVE-2026-42055](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-42055)
* **F5 NGINX:** [K000161616: NGINX ngx_http_v3_module vulnerability CVE-2026-42530](https://my.f5.com/manage/s/article/K000161616)
* **F5 NGINX:** [K000161584: NGINX ngx_http_proxy_v2_module and ngx_http_grpc_module vulnerability CVE-2026-42055](https://my.f5.com/manage/s/article/K000161584)
* **NGINX:** [NGINX Security Advisories](https://nginx.org/en/security_advisories.html)

### 5. Fonti e Referenze
* **Fonte Primaria:** [Agenzia per la Cybersicurezza Nazionale (ACN) - Risolte vulnerabilità nei prodotti NGINX](https://www.acn.gov.it/portale/web/guest/-/risolte-vulnerabilita-nei-prodotti-nginix)
* **Approfondimento:** [The Hacker News - F5 Patches Two Critical NGINX Open Source Flaws Enabling Remote Code Execution](https://thehackernews.com/2026/06/f5-patches-two-critical-nginx-open.html)
* **Approfondimento:** [IONIX Threat Center - CVE-2026-42530 Advisory](https://www.ionix.io/threat-center/cve-2026-42530/)

### 6. Indicatori di Compromissione (IoC)
Nessun indicatore di compromissione (IoC) rilevato nelle fonti analizzate.

### 7. Rilevamento e Tracce nei Sistemi
L'analisi dei tentativi di exploit e delle anomalie deve concentrarsi sulla diagnostica di sistema e sui comportamenti dei processi:

* **Crash dei Processi Worker (Syslog / dmesg / Kernel Log):** L'effetto immediato dell'attivazione di entrambe le vulnerabilità è il crash per segmentazione del thread di esecuzione. Ispezionare i log di sistema (es. `/var/log/syslog`, `/var/log/messages` o tramite `dmesg`) alla ricerca di errori di tipo:
    ```text
    nginx[PID]: segfault at ... ip ... sp ... error 4 in nginx
    traps: nginx[PID] general protection fault
    ```
* **Log degli Errori di NGINX (`error.log`):** Il processo master di NGINX registrerà l'interruzione anomala e il successivo riavvio automatico del processo figlio danneggiato. Controllare la presenza ricorsiva di messaggi simili a:
    ```text
    [alert] PID#0: worker process PID exited on signal 11 (core dumped)
    ```
    *Nota: Un numero elevato di riavvii dei worker process in archi temporali ridotti indica un potenziale attacco DoS in corso.*
* **Monitoraggio dei Processi e Core Dump:** Verificare la generazione di file core dump all'interno delle directory di lavoro di NGINX (se abilitati tramite la direttiva `worker_rlimit_core`). L'analisi post-mortem del core dump tramite tool come `gdb` mostrerà il crash originato dalle funzioni interne dei moduli `ngx_http_v3_module` o `ngx_http_proxy_v2_module`.