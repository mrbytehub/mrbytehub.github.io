---
layout: post
title: "Analisi: Vulnerabilità multiple in Squid Proxy (CVE-2026-47729, CVE-2026-50012)"
date: 2026-06-12
categories: security
tags: [linux,cve]
---

# Vulnerabilità multiple in Squid Caching Proxy (CVE-2026-47729, CVE-2026-50012)

**Impatto:** Alto - Rischio di esecuzione di codice arbitrario, Denial of Service (DoS) e divulgazione di informazioni sensibili.
**Vettore:** Network

### 1. Sintesi Tecnica
Sono state identificate due vulnerabilità che colpiscono Squid, il noto software open source ampiamente utilizzato come caching proxy server. L'analisi tecnica evidenzia i seguenti vettori di minaccia legati alla gestione interna dei protocolli e dei moduli di memoria:
* **CVE-2026-50012 (Heap-based Buffer Overflow):** Dinamica causata da un difetto di validazione impropria dell'input (Improper Input Validation) all'interno del modulo *Cache Digest Handler*. Un server remoto considerato attendibile o un utente malintenzionato in grado di camuffarsi da tale può sfruttare questa falla inviando risposte malevole appositamente configurate ai messaggi di richiesta `cache_digest`. Questo innesca un overflow nella memoria heap, con conseguente arresto anomalo del servizio (Denial of Service) o potenziale esecuzione di codice arbitrario (Arbitrary Code Execution).
* **CVE-2026-47729 (Out-of-bounds Read):** Vulnerabilità di lettura fuori dai limiti individuata all'interno del modulo *FTP Gateway Handler*. Un utente malintenzionato remoto può inviare input non validi o malformati durante le transazioni di gateway FTP gestite dal proxy. Questo comportamento induce Squid a leggere aree di memoria al di fuori dei buffer allocati, provocando la divulgazione di informazioni riservate (Information Disclosure) memorizzate nella RAM del sistema o il crash del processo.

### 2. Check di Perimetro ed Esposizione
Un sistema che esegue Squid deve considerarsi esposto o vulnerabile qualora soddisfi i seguenti criteri:
* **Versioni software affette:** Presenza di installazioni di Squid antecedenti alla versione **7.6** (inclusi i rami stabili precedenti 4.x, 5.x e 6.x).
* **Opzioni di compilazione specifiche (per CVE-2026-50012):** L'istanza deve essere stata esplicitamente configurata e compilata con l'opzione `--enable-cache-digests`. Se questa opzione non è attiva nell'eseguibile in uso, l'attacco basato sulla heap non può andare a buon fine.
* **Esposizione dei servizi:** Porte del proxy Squid raggiungibili da reti non fidate o configurazione di regole di peering (`cache_peer`) che consentono lo scambio di cache digest con server remoti esterni arbitrari.

### 3. Strategia di Remediation e Hardening
La risoluzione e la mitigazione del rischio richiedono l'adozione delle seguenti misure protettive sui sistemi:
* **Aggiornamento immediato:** Eseguire l'upgrade di Squid alla versione **7.6** o successive, oppure applicare tempestivamente le patch di sicurezza rilasciate dai manutentori della specifica distribuzione Linux in uso (Debian, Ubuntu, Red Hat, Amazon Linux).
* **Workaround e Hardening di Configurazione:**
  * Qualora l'aggiornamento non sia immediatamente applicabile, disabilitare lo scambio dei cache digest. Modificare il file `squid.conf` rimuovendo o commentando i nodi di peering non strettamente necessari, oppure ricompilare Squid escludendo l'opzione `--enable-cache-digests`.
  * Limitare o disattivare il modulo gateway FTP configurando in modo restrittivo le ACL di rete per bloccare richieste FTP non autorizzate dirette al proxy.
  * Implementare regole di firewalling per limitare l'accesso alle porte di ascolto di Squid soltanto a indirizzi IP strettamente autorizzati e fidati.

### 4. Riferimenti Ufficiali e Bollettini Vendor
* **Riferimento Ufficiale:** [MITRE CVE-2026-47729](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-47729)
* **Riferimento Ufficiale:** [MITRE CVE-2026-50012](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-50012)
* **Debian Security Tracker:** [Squid Source Package Vulnerabilities](https://security-tracker.debian.org/tracker/CVE-2026-47729)
* **Amazon Linux Security Center:** [Advisory CVE-2026-50012](https://explore.alas.aws.amazon.com/CVE-2026-50012.html)

### 5. Fonti e Referenze
* **Fonte Primaria:** [ACN Agenzia per la Cybersicurezza Nazionale - Rilevate vulnerabilità in Squid](https://www.acn.gov.it/portale/w/rilevate-vulnerabilita-in-squid)
* **Approfondimento:** [OSS-Security Mailing List - Squid Security Releases](https://seclists.org/oss-sec/2026/q2/896)

### 6. Indicatori di Compromissione (IoC)
Nessun indicatore di compromissione (IoC) rilevato nelle fonti analizzate.

### 7. Rilevamento e Tracce nei Sistemi
Per intercettare tentativi di abuso o comportamenti anomali generati dalle vulnerabilità descritte, controllare gli elementi nativi del sistema:
* **Verifica dei Log di Errore (`cache.log`):**
  * Ispezionare il log di diagnostica di Squid (generalmente posizionato in `/var/log/squid/cache.log`) alla ricerca di arresti improvvisi registrati come `Segmentation fault`, `SIGSEGV` o errori associati a violazioni della memoria heap durante l'elaborazione dei digest provenienti dai `cache_peer`.
* **Monitoraggio dei Log di Accesso (`access.log`):**
  * Analizzare le richieste HTTP e FTP nel file `/var/log/squid/access.log`. Identificare stringhe di comando insolitamente lunghe, payloads strutturati con caratteri di escape non standard o sequenze ripetitive mirate a saturare i buffer dei moduli FTP.
* **Ispezione dei Binari e dei Processi:**
  * Controllare le opzioni con cui è stato compilato l'eseguibile attivo lanciando il comando `squid -v`. Controllare se l'output contiene la stringa `--enable-cache-digests` per determinare l'effettiva presenza del modulo affetto da overflow.