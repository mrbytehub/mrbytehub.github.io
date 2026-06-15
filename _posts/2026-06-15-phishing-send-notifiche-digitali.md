---
layout: post
title: "Analisi: Campagna di Phishing SEND - Servizio Notifiche Digitali"
date: 2026-06-15
categories: security
tags: [phishing,smishing]
---

# Campagna di Phishing a Tema "SEND - Servizio Notifiche Digitali"

**Impatto:** Medio (40.0) - Sottrazione di dati personali, anagrafici e credenziali di pagamento (carte di credito).
**Vettore:** SMS (Smishing) e Web.

### 1. Sintesi Tecnica
È stata rilevata una campagna illecita di smishing finalizzata alla frode finanziaria e al furto di identità, orchestrata mediante la contraffazione delle interfacce istituzionali di "SEND – Servizio Notifiche Digitali" e "pagoPA". La dinamica dell'attacco si sviluppa secondo le seguenti fasi:

* **Iniezione del Vettore:** La vittima riceve un SMS ingannevole inviato da un mittente alfanumerico generico (es. "Comune") che notifica una falsa infrazione del Codice della Strada associata alla targa del destinatario, includendo un collegamento ipertestuale esterno.
* **Fase di Evasione ed Escation:** Il collegamento reindirizza a una landing page malevola che richiede l'inserimento del numero di targa per verificare presunti insoluti della Polizia Stradale.
* **Ingegneria Sociale Mirata:** Una volta inserita la targa, il sistema mostra dettagli tecnici verosimili e precisi (velocità rilevata, tolleranza, limite di velocità, importo ridotto entro 5 giorni pari a 44,40 euro) strutturati per creare urgenza psicologica nell'utente.
* **Raccolta dati PII:** Il portale richiede l'inserimento completo di dati anagrafici (nome, cognome, indirizzo, CAP, provincia, e-mail, numero di telefono).
* **Esfiltrazione Finanziaria:** L'utente viene infine reindirizzato a una simulazione di interfaccia di pagamento pagoPA/SEND dove viene indotto a inserire i dati sensibili della propria carta di credito, completando la compromissione dello strumento finanziario.

### 2. Check di Perimetro ed Esposizione
La minaccia non sfrutta vulnerabilità software di tipo RCE o difetti logici applicativi lato server su asset aziendali, ma si basa sull'esposizione del fattore umano e dei dispositivi mobili aziendali/personali. I fattori di rischio per l'infrastruttura aziendale includono:
* Mancanza di politiche restrittive o filtri DNS sui dispositivi mobili aziendali (MDM/COPE/BYOD).
* Assenza di blocchi preventivi sui sistemi di Web Proxy aziendali verso domini di recente registrazione (Lookalike o Typosquatting) privi di reputazione consolidata.

### 3. Strategia di Remediation e Hardening
Per mitigare la minaccia a livello infrastrutturale e di endpoint, implementare le seguenti misure:
* **Inibizione di Rete:** Configurare i DNS aziendali, i firewall e i sistemi di Secure Web Gateway (SWG) per bloccare la risoluzione e il traffico verso il dominio identificato e i relativi sotto-URL.
* **MDM & MTD Policy:** Implementare regole di Mobile Threat Defense (MTD) tramite le soluzioni di Mobile Device Management (MDM) per identificare e bloccare la ricezione o l'apertura di URL malevoli da SMS.
* **Restrizioni sui Gateway Email:** Integrare regole di content filtering sui gateway di posta aziendali per intercettare comunicazioni analoghe qualora la campagna si estendesse al canale e-mail.
* **Incident Response Finanziario:** In caso di avvenuta interazione e inserimento dati da parte di un utente, procedere all'immediato blocco della carta di pagamento tramite l'istituto emittente e avviare le procedure di monitoraggio per furto di identità sulle credenziali aziendali (qualora sia stata usata l'e-mail corporate).

### 4. Riferimenti Ufficiali e Bollettini Vendor
Trattandosi di una campagna di phishing basata su tecniche di ingegneria sociale e non del logoramento di un bug software, non è associata ad alcuna CVE (Common Vulnerabilities and Exposures) del MITRE o del NIST.

* **Riferimento Ufficiale CSIRT Italia:** [Alert AL04/260615/CSIRT-ITA](https://www.acn.gov.it/portale/w/campagna-di-phishing-a-tema-send-servizio-notifiche-digitali-)

### 5. Fonti e Referenze
* **Fonte Primaria:** [ACN Bollettino - Campagna di phishing a tema "SEND – Servizio Notifiche Digitali"](https://www.acn.gov.it/portale/w/campagna-di-phishing-a-tema-send-servizio-notifiche-digitali-)

### 6. Indicatori di Compromissione (IoC)

| Tipo | Valore | Note |
| :--- | :--- | :--- |
| url | https://notificheadigitali[.]org/it | Pagina web di atterraggio della frode |
| domain | notificheadigitali[.]org | Dominio contraffatto utilizzato per simulare la piattaforma SEND |

### 7. Rilevamento e Tracce nei Sistemi
Per identificare potenziali interazioni interne con la minaccia, analizzare i log dei sistemi di sicurezza aziendali come segue:

* **Log dei Server DNS Aziendali:** Verificare la presenza di query di risoluzione verso il dominio `notificheadigitali.org`.
* **Log del Web Proxy / Next-Generation Firewall (NGFW):** Ricercare all'interno delle stringhe HTTP Request o nei log di connessione HTTPS (se presente ispezione SSL/TLS) record associati a:
  * `notificheadigitali.org`
  * `https://notificheadigitali.org/it`
* **Eventi Windows (Sysmon) sui Client Enterprise:** Qualora i client o i notebook aziendali abbiano navigato sul sito tramite tethering o reti aziendali, verificare la presenza del seguente evento nel registro di Sysmon:
  * **Event ID 22 (DNS Query):** Verificare se il campo `QueryName` corrisponde a `notificheadigitali.org`.