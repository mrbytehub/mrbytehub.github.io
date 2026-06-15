---
layout: post
title: "Analisi: CVE-2026-48907"
date: 2026-06-15
categories: security
tags: [joomla, jce, cve, rce, exploit]
---

# Vulnerabilità RCE Unauthenticated nell'estensione Joomla Content Editor (JCE)

**Impatto:** Critico (CVSS 10.0) - Consente il pieno compromesso del server web ospitante mediante il caricamento e l'esecuzione di codice PHP arbitrario senza autenticazione.
**Vettore:** Network

### 1. Sintesi Tecnica
La vulnerabilità CVE-2026-48907 è una falla di tipo Remote Code Execution (RCE) derivante da una catena di tre debolezze logiche distinte presenti nella gestione dell'importazione dei profili del plugin Joomla Content Editor (JCE) per il CMS Joomla!:
* **Assenza di Autorizzazione:** L'endpoint "/index.php?option=com_jce&task=profiles.import" non implementa controlli di sessione amministrativa o privilegi di accesso, consentendo a utenti non autenticati di invocare l'azione di importazione dei profili dell'editor.
* **Mancata Validazione delle Estensioni:** La funzionalità di importazione riceve file XML di configurazione del profilo ma non valida in modo rigido la struttura o i dati allegati, permettendo la manipolazione delle direttive sui tipi di file ammessi.
* **Funzione di Upload Permissiva:** La chiamata interna al metodo "File::upload" viene effettuata con il parametro "$allow_unsafe = true", bypassando i controlli di sicurezza standard sulle estensioni pericolose.

Un attaccante può inviare una richiesta multipart confezionata per importare un profilo modificato ad hoc con politiche di caricamento arbitrarie (abilitando estensioni come ".php"). Successivamente, sfruttando l'endpoint RPC dello stesso plugin ("task=plugin.rpc&plugin=browser&method=upload"), l'attaccante può caricare una web shell ed eseguirla via HTTP per ottenere l'esecuzione di comandi sul sistema operativo.

### 2. Check di Perimetro ed Esposizione
Un sistema è esposto e vulnerabile se soddisfa i seguenti prerequisiti tecnici:
* È in esecuzione il CMS Joomla! con l'estensione Joomla Content Editor (JCE) attiva.
* La versione di JCE installata è compresa tra la v1.0.0 e la v2.9.99.4 inclusa.
* L'endpoint "index.php?option=com_jce" è raggiungibile pubblicamente senza restrizioni di rete o regole di filtraggio a livello perimetrale.

### 3. Strategia di Remediation e Hardening
* **Aggiornamento Software:** Aggiornare immediatamente l'estensione JCE alla versione ">= 2.9.99.6". Sebbene la vulnerabilità sia stata originariamente sanata nella versione 2.9.99.5, la release 2.9.99.6 introduce misure aggiuntive di hardening e audit del codice.
* **Hardening del Server Web:** Configurare il server web (Apache, Nginx) per impedire l'esecuzione di script PHP all'interno delle directory dedicate ai contenuti statici e ai file temporanei (es. "/images/", "/tmp/").
* **Filtraggio delle Richieste:** Applicare regole WAF o direttive di blocco a livello di web server per intercettare e scartare le richieste HTTP contenenti i parametri "option=com_jce" combinati con "task=profiles.import" non provenienti da indirizzi IP fidati o reti di gestione.

### 4. Riferimenti Ufficiali e Bollettini Vendor
* **Riferimento Ufficiale:** [MITRE CVE-2026-48907](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-48907)
* **Riferimento Ufficiale:** [NVD NIST CVE-2026-48907](https://nvd.nist.gov/vuln/detail/CVE-2026-48907)
* **Joomla Content Editor:** [JCE Editor Changelog](https://www.joomlacontenteditor.net/support/changelog/editor)

### 5. Fonti e Referenze
* **Fonte Primaria:** [ACN Agenzia per la Cybersicurezza Nazionale - Joomla JCE: sfruttamento attivo in rete della CVE-2026-48907](https://www.acn.gov.it/portale/w/joomla-jce-sfruttamento-attivo-in-rete-della-cve-2026-48907)
* **Approfondimento:** [YesWeHack - CVE-2026-48907: Unauthenticated RCE in the Joomla Content Editor extension](https://www.yeswehack.com/news/rce-joomla-content-editor-extension)
* **Approfondimento:** [mySites.guru - Find and Fix the JCE Profiles Hack](https://mysites.guru/blog/finding-every-site-running-a-vulnerable-jce/)

### 6. Indicatori di Compromissione (IoC)
| Tipo | Valore | Note |
| :--- | :--- | :--- |
| IP | 107.149.130[.]5 | IP sorgente rilevato in campagne di exploit automatizzate |
| File | m.php | Nome tipico della web shell PHP caricata nei casi analizzati |

### 7. Rilevamento e Tracce nei Sistemi
Per identificare tentativi di attacco o compromissioni riuscite, analizzare i log degli accessi del server web ("access.log") e il file system ricercando i seguenti pattern:

* **Log del Server Web (Richiesta di Creazione Profilo):**
  Presenza di richieste POST indirizzate alla funzione di importazione con codice di stato HTTP 200:
  "POST /index.php?option=com_jce&task=profiles.import"

* **Log del Server Web (Caricamento della Web Shell):**
  Identificazione di chiamate RPC successive per il caricamento di file tramite il plugin browser:
  "POST /index.php?option=com_jce&task=plugin.rpc&plugin=browser&method=upload"

* **Log del Server Web (Esecuzione della Shell):**
  Richieste HTTP GET verso script PHP posizionati in directory insolite o normalmente destinate ai soli media:
  "GET /images/m.php?cmd=" o "GET /tmp/"

* **Verifica sul File System:**
  * Monitorare la directory dei profili JCE per verificare la creazione di nuovi file di configurazione XML non autorizzati.
  * Effettuare una scansione delle cartelle scrivibili dal server (es. "/images/", "/tmp/") alla ricerca di script PHP creati di recente o con timestamp alterati.