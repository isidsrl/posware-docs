# Autenticazione e Accesso

Questo documento descrive le modalità di autenticazione, gestione della sessione, ruoli e policy di sicurezza di StoreServer.

---

## 1. Panoramica

StoreServer utilizza un sistema di autenticazione locale basato su **email e password** con credenziali memorizzate nel database interno.

La sessione è gestita tramite **token JWT**:

- **Access token** — valido 15 minuti, inviato nell'header `Authorization`
- **Refresh token** — valido 7 giorni, memorizzato in cookie `httpOnly` con rotazione automatica

È disponibile l'autenticazione a **due fattori (2FA)** tramite app TOTP (Google Authenticator, Microsoft Authenticator, ecc.).

> **Modalità future (non ancora attive):** SSO/OIDC e social login sono previste in roadmap ma non disponibili nella versione corrente.

---

## 2. Accesso al sistema

### URL di accesso

L'applicazione è raggiungibile sulla porta **6851**. Il form di login si trova alla radice dell'applicazione.

### Credenziali

| Campo | Descrizione |
|---|---|
| Email | Indirizzo email dell'account |
| Password | Password associata all'account |

### Politica password

Le password devono rispettare i seguenti requisiti minimi:

- Almeno **8 caratteri**
- Almeno una **lettera maiuscola**
- Almeno una **lettera minuscola**
- Almeno una **cifra numerica**
- Almeno un **carattere speciale** (es. `!`, `@`, `#`, `$`)

### Flusso di accesso

```
Inserimento credenziali → Verifica → (2FA se abilitata) → Access token + Refresh token → (Cambio password obbligatorio se richiesto) → Sessione attiva
```

### Sessioni concorrenti

Ogni utente può avere un massimo di **3 sessioni attive contemporaneamente**. Al superamento del limite, la sessione più vecchia viene revocata automaticamente.

---

## 3. Autenticazione a due fattori (2FA)

### Cos'è e perché usarla

La 2FA aggiunge un secondo livello di verifica oltre alla password. Anche se le credenziali venissero compromesse, l'accesso non è possibile senza il codice temporaneo generato dall'app sul dispositivo dell'utente.

### Prerequisiti

Un'app authenticator installata su smartphone o tablet:

- Google Authenticator
- Microsoft Authenticator
- Authy
- Qualsiasi app compatibile con lo standard TOTP (RFC 6238)

### Come abilitare la 2FA

1. Accedere alla propria area utente → **Sicurezza**
2. Selezionare **Abilita autenticazione a due fattori**
3. Inquadrare il **QR code** con l'app authenticator
4. Inserire il **codice a 6 cifre** generato dall'app per confermare l'associazione
5. **Salvare i codici di recupero** mostrati a schermo in un luogo sicuro

> **Importante:** i codici di recupero vengono mostrati una sola volta. Conservarli in modo sicuro (es. gestore di password, documento stampato).

### Come funziona al login

Dopo aver inserito email e password, verrà richiesto di:

1. Aprire l'app authenticator
2. Inserire il **codice TOTP a 6 cifre** relativo all'account StoreServer
3. Confermare

Il codice è valido per circa 30 secondi; l'app genera automaticamente un nuovo codice alla scadenza.

### Codici di recupero

Vengono generati **10 codici monouso** al momento dell'attivazione della 2FA. Ogni codice può essere utilizzato una sola volta in caso di impossibilità di accedere all'app authenticator (es. dispositivo smarrito).

Dopo l'uso, il codice viene invalidato automaticamente.

### Come disabilitare la 2FA

1. Accedere alla propria area utente → **Sicurezza**
2. Selezionare **Disabilita autenticazione a due fattori**
3. Inserire la password corrente per confermare

---

## 4. Gestione sessione e token

### Durata dei token

| Token | Durata | Note |
|---|---|---|
| Access token | 15 minuti | Rinnovato automaticamente tramite refresh token |
| Refresh token | 7 giorni | Cookie `httpOnly`, rotazione ad ogni rinnovo |

Il rinnovo dell'access token avviene in modo trasparente per l'utente durante l'utilizzo normale dell'applicazione.

### Logout

Il logout revoca il refresh token corrente e termina la sessione. L'access token residuo (max 15 min) non può essere revocato anticipatamente, ma perde utilità all'uscita.

### Sessioni concorrenti

Ogni utente può mantenere attive fino a **3 sessioni contemporanee** (es. browser diversi, dispositivi diversi). Al tentativo di apertura di una quarta sessione, quella più vecchia viene revocata automaticamente.

---

## 5. Ruoli e permessi

Ogni utente ha uno o più ruoli che determinano le funzionalità accessibili. Un utente può avere più ruoli contemporaneamente.

| Ruolo | Descrizione | Accesso |
|---|---|---|
| **SystemAdmin** | Amministratore di sistema | Completo — tutte le funzionalità incluse impostazioni di sistema e manutenzione |
| **Technician** | Tecnico di sistema | Completo — equivalente a SystemAdmin per le funzionalità operative |
| **StoreManager** | Direttore punto vendita | Tutto tranne moduli di sistema, impostazioni avanzate, manutenzione e gestione ruoli |
| **ShiftManager** | Responsabile turno | Dashboard, statistiche, visualizzazione e gestione dispositivi POS |
| **BillingOperator** | Operatore fatturazione | Fatturazione, fatture elettroniche, note credito, clienti, statistiche |
| **ReportViewer** | Visualizzatore report | Dashboard e statistiche (sola lettura) |
| **PriceCheckerAdmin** | Amministratore verifica prezzi | Modulo PPC (Price Checker) — visualizzazione e gestione |

---

## 6. Gestione utenti (amministratori)

### Accesso alla pagina

La pagina di gestione utenti è accessibile dal menu laterale **Sistema → Gestione utenti**.

!!! note "Permesso richiesto"
    Per accedere alla pagina è necessario il permesso `store.users.view`.

### Elenco utenti

La pagina mostra una tabella con tutti gli utenti del punto vendita:

| Colonna | Descrizione |
|---|---|
| Nome | Nome dell'utente |
| Cognome | Cognome dell'utente |
| Email | Indirizzo email (usato come credenziale di accesso) |
| Stato | Attivo o disattivato |
| 2FA | Indica se l'autenticazione a due fattori è abilitata |
| Azioni | Modifica, reset password, disabilita 2FA |

### Creazione utente

Per creare un nuovo utente:

1. Accedere a **Sistema → Gestione utenti**
2. Cliccare **Nuovo utente**
3. Compilare il form:
    - **Email** — obbligatoria, deve essere univoca
    - **Nome** — obbligatorio
    - **Cognome** — obbligatorio
4. Cliccare **Crea**

Il sistema:

- Genera automaticamente una **password sicura** di 16 caratteri (con maiuscole, minuscole, cifre e caratteri speciali)
- Assegna i ruoli di default: **ShiftManager**, **BillingOperator**, **ReportViewer**
- Attiva il flag **cambio password obbligatorio** al primo accesso

Al completamento, viene mostrata una modale con la **password generata** e un pulsante per copiarla negli appunti.

!!! warning "Password mostrata una sola volta"
    La password viene visualizzata **esclusivamente** nella modale di conferma. Una volta chiusa, non è più recuperabile. Comunicare la password all'utente in modo sicuro (di persona o tramite canale protetto). **Non inviare mai la password via email non cifrata.**

### Modifica utente

Per modificare un utente esistente:

1. Nella lista utenti, cliccare l'icona **matita** sull'utente da modificare
2. Aggiornare i campi desiderati:
    - **Email**
    - **Nome**
    - **Cognome**
    - **Stato** — attivo o disattivato
3. Cliccare **Salva**

### Reset password amministratore

Un amministratore può reimpostare la password di un utente senza conoscere la password corrente.

1. Nella lista utenti, cliccare l'icona **chiave** sull'utente
2. Nel dialog che appare, verificare (e se necessario deselezionare) l'opzione **"Richiedi cambio password al prossimo accesso"** (attiva per default)
3. Cliccare **Reset password**

Il sistema:

- Genera una **nuova password sicura** di 16 caratteri
- Attiva o disattiva il flag cambio password in base alla scelta dell'admin
- Mostra la nuova password in una modale (una sola volta)

!!! tip "Quando disattivare il cambio password obbligatorio"
    Disattivare l'opzione quando la password viene consegnata direttamente a voce all'utente presente, evitando un passaggio aggiuntivo non necessario.

!!! warning "Password mostrata una sola volta"
    Come per la creazione, la password è visibile solo nella modale. Comunicarla all'utente in modo sicuro.

### Disabilitazione 2FA amministratore

Un amministratore può disabilitare l'autenticazione a due fattori di un utente senza il codice OTP dell'utente.

1. Nella lista utenti, cliccare l'icona **scudo** sull'utente con 2FA attiva
2. Confermare l'operazione

Il sistema:

- Disabilita la 2FA per l'utente
- Invalida la chiave authenticator associata
- L'utente dovrà riconfigurare la 2FA dal proprio profilo se necessario

!!! warning "Impatto sulla sicurezza"
    La disabilitazione della 2FA riduce il livello di protezione dell'account. Utilizzare questa funzione solo in caso di reale necessità (es. dispositivo smarrito, app authenticator non più accessibile).

### Disattivazione utente

La disattivazione di un utente è una **soft-delete**: l'account viene disabilitato (campo `IsActive = false`) ma i dati vengono conservati per scopi di audit. L'utente non può effettuare il login finché l'account è disattivato.

Per disattivare un utente, modificare il campo **Stato** da "Attivo" a "Disattivato" nella schermata di modifica utente.

---

## 7. Cambio password

### Cambio volontario

1. Accedere alla propria area utente → **Sicurezza**
2. Inserire la password corrente
3. Inserire la nuova password (rispettare la politica password)
4. Confermare la nuova password

### Cambio forzato al primo accesso

Dopo il login, se `forcePasswordChange = true`, il sistema reindirizza automaticamente a `/auth/change-password`.

- L'accesso a qualsiasi altra pagina è bloccato finché la password non è cambiata
- Il form richiede: **Nuova password** + **Conferma nuova password** (nessuna password attuale richiesta)
- Al salvataggio, il sistema aggiorna la password e reindirizza alla dashboard

---

## 8. Profilo utente

Ogni utente può aggiornare le proprie informazioni dall'area profilo.

### Dati aggiornabili

| Campo | Tipo |
|---|---|
| Nome | Testo |
| Cognome | Testo |
| Azienda | Testo |
| Ruolo aziendale | Testo |
| Telefono | Testo |
| Sito web | URL |
| Paese | Selezione |
| Indirizzo | Testo |

### Avatar

- Formati accettati: **JPG**, **PNG**, **WebP**
- Dimensione massima: **2 MB**

### Completamento profilo

Il sistema calcola una **percentuale di completamento** del profilo in base ai campi valorizzati, visibile nell'area utente.

---

## 9. Log di audit (amministratori)

### Cosa viene tracciato

Quando il log di audit è abilitato, vengono registrate le seguenti azioni:

| Azione | Descrizione |
|---|---|
| Login | Accesso al sistema (riuscito o fallito) |
| Logout | Uscita dal sistema |
| Cambio password | Modifica della password utente |
| Creazione utente | Nuovo account creato |
| Modifica utente | Aggiornamento dati o ruoli utente |
| Abilitazione 2FA | Attivazione autenticazione a due fattori |
| Disabilitazione 2FA | Disattivazione autenticazione a due fattori |
| Reset password admin | Password reimpostata da un amministratore (`user.password.admin-reset`) |
| Disabilitazione 2FA admin | 2FA disabilitata da un amministratore (`user.2fa.admin-disable`) |
| Cambio password forzato | Password cambiata dall'utente al primo accesso obbligatorio (`user.password.force-change`) |

Per ogni evento vengono registrati: utente, indirizzo IP, user agent, timestamp UTC, dettagli dell'azione.

### Abilitazione

Il log di audit è **disabilitato per default**. Per abilitarlo, modificare `appsettings.json`:

```json
{
  "AuditLog": {
    "Enabled": true
  }
}
```

---

## 10. Configurazione (amministratori di sistema)

Parametri configurabili in `appsettings.json`:

| Parametro | Default | Descrizione |
|---|---|---|
| `Jwt:ACCESS_TOKEN_EXPIRATION_MINUTES` | `15` | Durata in minuti dell'access token |
| `Jwt:REFRESH_TOKEN_EXPIRATION_DAYS` | `7` | Durata in giorni del refresh token |
| `AuditLog:Enabled` | `false` | Abilita la registrazione del log di audit |

### Account di default

Al primo avvio, il sistema crea automaticamente tre account predefiniti se non esistono:

| Account | Email | Password di default | Ruolo |
|---|---|---|---|
| Amministratore di sistema | `admin@posware.store` | `Vbhg4132!` | SystemAdmin |
| Supporto tecnico | `support@posware.store` | `Support4132!` | Technician |
| Direttore punto vendita | `manager@posware.store` | `Manager1!` | StoreManager |

!!! warning "Modificare le credenziali di default"
    Cambiare email e password dell'account "Direttore punto vendita" immediatamente dopo la prima installazione del sistema. La password di default è nota e rappresenta un rischio di sicurezza se lasciata invariata.
