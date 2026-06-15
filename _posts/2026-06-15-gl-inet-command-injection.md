---
layout: post
title: "Analisi: GL.iNet GL-MT3000 Command Injection (CVE-2026-12186 e CVE-2026-12187)"
date: 2026-06-15
categories: security
tags: [cve,rce]
---

# Analisi dei Proof of Concept Pubblici per Command Injection su Dispositivi GL.iNet GL-MT3000

**Impatto:** Critico - Consente l'esecuzione di comandi arbitrari da remoto a utenti con privilegi limitati.
**Vettore:** Network

### 1. Sintesi Tecnica
Sono stati rilasciati pubblicamente Proof of Concept (PoC) relativi a due vulnerabilità di sicurezza di gravità elevata che colpiscono i dispositivi router GL.iNet GL-MT3000. Entrambe le falle espongono il sistema ad attacchi di command injection:

* **CVE-2026-12186:** La falla risiede nella funzione `replace_country` all'interno della libreria `/usr/lib/oui-httpd/rpc/tor`, appartenente al componente *Tor Proxy Service Configuration Handler*. Sfruttando questa debolezza, un utente malintenzionato remoto già autenticato con privilegi limitati può iniettare ed eseguire comandi di sistema non autorizzati.
* **CVE-2026-12187:** La vulnerabilità interessa il file eseguibile `/usr/bin/one_click_upgrade` integrato nel componente *Online Firmware Upgrade Handler*. Un utente malintenzionato remoto provvisto di credenziali a basso privilegio può manipolare i parametri di input legati alla funzionalità di aggiornamento firmware online per forzare l'esecuzione di comandi arbitrari a livello di sistema operativo.

### 2. Check di Perimetro ed Esposizione
Un sistema è esposto e potenzialmente vulnerabile se soddisfa i seguenti criteri tecnici direttamente verificabili sul dispositivo:
* Il modello dell'hardware in uso è il router **GL.iNet GL-MT3000**.
* Il firmware installato sul dispositivo è precedente alla **versione 4.7** (es. versione 4.4.5 o antecedenti).
* L'interfaccia di amministrazione web del router (`oui-httpd`) è raggiungibile via rete (esposta sulla LAN o erroneamente aperta sull'interfaccia WAN).
* Sono presenti credenziali di accesso note, deboli o predefinite che consentono a un utente malintenzionato di ottenere una sessione autenticata, seppur con permessi minimi.

### 3. Strategia di Remediation e Hardening
Le azioni pratiche per mitigare e risolvere definitivamente le minacce sui dispositivi consistono in:
* **Aggiornamento immediato del firmware:** Aggiornare il software dei dispositivi affetti alla versione 4.7 o successive (è disponibile la release ufficiale stabile mt3000-4.8.1 o successive).
* **Restrizione degli accessi di gestione:** Limitare l'esposizione della dashboard web disabilitando tassativamente la funzione di gestione remota dall'interfaccia WAN.
* **Disattivazione dei servizi non necessari:** Disabilitare il modulo Tor Proxy e bloccare i meccanismi di upgrade automatico non supervisionati qualora non siano strettamente indispensabili per l'operatività del dispositivo.
* **Irrobustimento delle credenziali:** Sostituire le password di fabbrica o deboli di tutti gli account registrati sul sistema per prevenire l'accesso iniziale.

### 4. Riferimenti Ufficiali e Bollettini Vendor
* **Riferimento Ufficiale:** [MITRE CVE-2026-12186](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-12186)
* **Riferimento Ufficiale:** [NVD NIST CVE-2026-12186](https://nvd.nist.gov/vuln/detail/CVE-2026-12186)
* **Riferimento Ufficiale:** [MITRE CVE-2026-12187](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2026-12187)
* **Riferimento Ufficiale:** [NVD NIST CVE-2026-12187](https://nvd.nist.gov/vuln/detail/CVE-2026-12187)
* **GL.iNet:** [GL-MT3000 Firmware Release Advisory](https://fw.gl-inet.com/firmware/mt3000/release/mt3000-4.8.1-0819-1755615825.tar)

### 5. Fonti e Referenze
* **Fonte Primaria:** [ACN - PoC pubblici per le CVE-2026-12186 e CVE-2026-12187](https://www.acn.gov.it/portale/web/guest/-/poc-pubblici-per-le-cve-2026-12186-e-cve-2026-12187)
* **Approfondimento:** [CSIRT Toscana - Dettaglio Bollettino AL01/260615/CSIRT-ITA](https://csirt.regione.toscana.it/poc-pubblici-per-le-cve-2026-12186-e-cve-2026-12187-al01-260615-csirt-ita/)
* **Approfondimento:** [GitHub Repository StrTzz123 IoT Vulnerabilities](https://github.com/StrTzz123/iot_vul/tree/main/GL-iNet/MT3000/4.4.5/upgrade_online_url)

### 6. Indicatori di Compromissione (IoC)
Nessun indicatore di compromissione (IoC) rilevato nelle fonti analizzate.

### 7. Rilevamento e Tracce nei Sistemi
Per identificare tentativi di exploitation o attività sospette all'interno del sistema operativo OpenWrt del router, monitorare i seguenti elementi:
* **Analisi dei Log di Sistema:** Ispezionare i log generati dal demone web `oui-httpd` (tramite comando `logread` o esaminando i file in `/var/log/messages`). Ricercare richieste HTTP o chiamate RPC dirette all'endpoint `/rpc/tor` contenenti stringhe anomale destinate alla funzione `replace_country`.
* **Tracciamento dei parametri di Upgrade:** Ispezionare le invocazioni dello script `/usr/bin/one_click_upgrade`. Monitorare i parametri passati alla funzionalità di aggiornamento firmware online, isolando la presenza di metacaratteri di concatenazione di comandi shell (es. `;`, `&&`, `|`, `$()`, o delimitatori in backtick `` ` ``).
* **Comportamenti anomali dei Processi:** Monitorare la struttura ad albero dei processi di sistema. Costituisce un indicatore di compromissione certo la presenza di processi figli come shell interattive o interpreti di comandi (`/bin/sh`, `/bin/ash`) originati dal processo HTTP principale (`oui-httpd`) o abbinati a utility di rete insolite (`curl`, `wget`, `nc`).