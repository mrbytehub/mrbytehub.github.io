---
layout: post
title: "Analisi: Vulnerabilità multiple in Squid Proxy (CVE-2026-47729, CVE-2026-50012)"
date: 2026-06-12
categories: security
tags: [squid, proxy, cache, cve-2026-47729, cve-2026-50012, acn, open-source]
---

# Vulnerabilità Multiple di Sicurezza Critiche in Squid Caching Proxy

**Impatto:** Alto - Rischio concreto di Denial of Service (DoS), Information Disclosure e potenziale esecuzione di codice arbitrario (Arbitrary Code Execution).
**Vettore:** Network

### 1. Sintesi Tecnica
L'Agenzia per la Cybersicurezza Nazionale (ACN) ha rilasciato un bollettino di allerta relativo a due nuove vulnerabilità individuate in Squid, il celebre software open-source impiegato come proxy di caching e web gateway. Le falle di sicurezza interessano tutte le versioni precedenti alla release 7.6. Di seguito viene presentata l'analisi dettagliata della root cause e dei meccanismi di sfruttamento:

* **CVE-2026-47729 (Out-of-bounds Read nel modulo FTP Gateway):** Questa vulnerabilità è riconducibile a un bug di tipo 'Improper Validation of Syntactic Correctness of Input'. Quando Squid opera come gateway per il protocollo FTP, la mancata corretta validazione sintattica della risposta proveniente da un server FTP remoto non conforme (o sotto il controllo di un attaccante) innesca una lettura fuori dai limiti della memoria (Out-of-Bounds Read). Questo consente a un client autorizzato di intercettare frammenti casuali di memoria contenenti transazioni non correlate gestite dal proxy in quel momento.
* **CVE-2026-50012 (Heap-based Buffer Overflow nei Cache Digests):** La falla risiede in un difetto di validazione dell'input ('Improper Input Validation') durante la gestione delle risposte ai messaggi di richiesta 'cache_digest'. Un server remoto malevolo può inviare risposte appositamente modificate per mandare in overflow la memoria heap del processo di Squid. Le conseguenze variano dal crash immediato del servizio (Denial of Service) al potenziale sfruttamento per l'esecuzione di codice non autorizzato. Questa vulnerabilità è circoscritta esclusivamente alle istanze di Squid compilate con l'opzione esplicita `--enable-cache-digests`.

### 2. Check di Perimetro (Come capire se sei colpito)
Per identificare la presenza della vulnerabilità all'interno del proprio perimetro infrastrutturale, gli amministratori di sistema devono verificare la versione in uso e i flag di configurazione attivi:

* **Verifica della Versione Lineare:** Qualsiasi istanza operativa di Squid con versione inferiore alla 7.6 (es. rami 4.x, 5.x, 6.x e versioni 7.x fino alla 7.5) è da considerarsi esposta alle minacce descritte.
* **Ispezione dei Flag di Compilazione:** Eseguire il comando `squid -v` sulla macchina host per verificare la stringa di compilazione. Se nell'output è presente l'argomento `--enable-cache-digests`, il sistema risulta esposto alla vulnerabilità critica CVE-2026-50012.
* **Analisi dell'Infrastruttura di Rete:** Mappare l'utilizzo del proxy per le connessioni in uscita che utilizzano lo schema `ftp://` per determinare se la funzionalità di FTP gateway sia attivamente sollecitata da utenti o applicazioni interne.

### 3. Strategia di Remediation
La mitigazione completa ed efficace richiede l'applicazione tempestiva delle patch ufficiali del fornitore o l'adozione di misure compensative temporanee:

* **Aggiornamento del Software (Risoluzione Definitiva):** Aggiornare immediatamente i pacchetti o i binari di Squid alla versione **7.6** o successive, introdotte per risolvere strutturalmente entrambe le vulnerabilità.
* **Rimozione del Supporto Cache Digests (Workaround CVE-2026-50012):** Qualora l'aggiornamento immediato non fosse praticabile, è indispensabile ricompilare il software escludendo l'opzione `--enable-cache-digests` o disattivare l'uso dei digest nel file di configurazione `squid.conf`, limitando lo scambio di tabelle di cache con peer remoti non fidati.
* **Restrizione degli Accessi FTP Gateway (Mitigazione CVE-2026-47729):** Isolare le funzionalità FTP disabilitando o limitando l'uso delle direttive di inoltro verso server FTP esterni non censiti o insicuri tramite apposite regole di Controllo Accessi (ACL) nel file `squid.conf`.

### 4. Distribuzioni Enterprise Tracker
* **Debian:** [Security Tracker CVE-2026-47729](https://security-tracker.debian.org/tracker/CVE-2026-47729)
* **Ubuntu:** [USN / CVE Tracker CVE-2026-47729](https://ubuntu.com/security/cve/CVE-2026-47729)
* **Red Hat:** [Red Hat CVE Portal CVE-2026-47729](https://access.redhat.com/security/cve/CVE-2026-47729)

### 5. Fonti e Referenze (Sitografia)
* **Fonte Primaria:** [ACN - Rilevate vulnerabilità in Squid](https://www.acn.gov.it/portale/w/rilevate-vulnerabilita-in-squid)
* **Approfondimento:** [Openwall oss-security - Squid CVE-2026-47729 and CVE-2026-50012](https://seclists.org/oss-sec/2026/q2/896)

### 6. Indicatori di Compromissione (IoC)
Nessun indicatore di compromissione (IoC) rilevato nelle fonti analizzate.

**Strumenti di Verifica Tecnica e Hunting:**
Al fine di automatizzare le attività di audit e di identificazione della vulnerabilità sugli host aziendali, viene fornito uno script Bash ad ampio spettro. Il codice esamina in modo sicuro le caratteristiche dell'eseguibile locale, estrae la versione e segnala la presenza dei flag di compilazione pericolosi.

```bash
#!/bin/bash
# ==============================================================================
# Squid Proxy Vulnerability Auditor (CVE-2026-47729 & CVE-2026-50012)
# Descrizione: Analizza l'istanza locale di Squid per rilevare versioni 
#              vulnerabili e configurazioni a rischio compilazione.
# ==============================================================================

set -euo pipefail

# Definizione dei codici colore per l'output a terminale
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
NC='\033[0m'

echo "======================================================================"
echo " [!] Squid Proxy Security Auditor & Hardening Tool"
echo "======================================================================"

# Individuazione del binario di Squid installato nel sistema
SQUID_BIN=$(command -v squid || command -v squid3 || true)

if [ -z "$SQUID_BIN" ]; then
    echo -e "${GREEN}[+] Squid non è installato o non è presente nel PATH di questo sistema.${NC}"
    exit 0
fi

echo -e "[*] Individuato binario Squid in: ${SQUID_BIN}"

# Estrazione della stringa di versione principale
SQUID_VERSION_RAW=$($SQUID_BIN -v | head -n 1 | awk '{print $4}')
echo -e "[*] Versione software rilevata: ${SQUID_VERSION_RAW}"

# Parsing della versione (Major e Minor) per la verifica della soglia di sicurezza
VERSION_MAJOR=$(echo "$SQUID_VERSION_RAW" | cut -d'.' -f1)
VERSION_MINOR=$(echo "$SQUID_VERSION_RAW" | cut -d'.' -f2)

IS_VULNERABLE=0

if [ "$VERSION_MAJOR" -lt 7 ]; then
    IS_VULNERABLE=1
elif [ "$VERSION_MAJOR" -eq 7 ] && [ "$VERSION_MINOR" -lt 6 ]; then
    =1
fi

if [ "$IS_VULNERABLE" -eq 1 ]; then
    echo -e "${RED}[!] ALLERTA: La versione attuale è precedente alla 7.6 ed è suscettibile ad attacchi.${NC}"
else
    echo -e "${GREEN}[+] Successo: La versione installata risulta protetta (>= 7.6).${NC}"
fi

# Verifica del flag specifico per la vulnerabilità CVE-2026-50012
echo -e "\n[*] Ispezione dei parametri di compilazione del binario..."
COMPILATION_FLAGS=$($SQUID_BIN -v)

if echo "$COMPILATION_FLAGS" | grep -q -- "--enable-cache-digests"; then
    echo -e "${RED}[!] RISCHIO: Il binario include '--enable-cache-digests'. Il vettore CVE-2026-50012 è ATTIVO.${NC}"
else
    echo -e "${GREEN}[+] Ottimo: Il binario non supporta '--enable-cache-digests'. Mitigazione integrata.${NC}"
fi

# Controllo preliminare dello stato di stabilità nei log
CACHE_LOG="/var/log/squid/cache.log"
echo -e "\n[*] Analisi euristica del registro degli errori di sistema..."
if [ -f "$CACHE_LOG" ]; then
    if grep -Ei "assertion failed|signal 11|segmentation fault" "$CACHE_LOG" | tail -n 5; then
        echo -e "${YELLOW}[!] Rilevati potenziali eventi insoliti o crash recenti nel log degli errori.${NC}"
    else
        echo -e "${GREEN}[+] Nessuna anomalia strutturale macroscopica rilevata nel log degli errori.${NC}"
    fi
else
    echo -e "[*] File log standard cache.log non presente nel percorso predefinito. Analisi log saltata."
fi

echo "======================================================================"
echo -e "[*] Audit completato. Aggiornare i sistemi vulnerabili alla release 7.6"
echo "======================================================================"
```