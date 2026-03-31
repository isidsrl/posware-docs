# Gestione ruoli e permessi

Questo documento descrive l'editor dei ruoli custom, la gestione dei permessi e l'assegnazione ruoli agli utenti in StoreServer.

---

## 1. Panoramica

StoreServer include un sistema di controllo accessi basato su ruoli (**RBAC**). Oltre ai ruoli predefiniti di sistema, è possibile creare **ruoli personalizzati** con un sottoinsieme di permessi selezionati.

Le funzionalità principali sono:

- **Visualizzazione ruoli** — elenco di tutti i ruoli (protetti e custom) con i rispettivi permessi
- **Creazione ruoli custom** — definizione di nuovi ruoli con permessi specifici
- **Modifica e cancellazione** — gestione dei ruoli custom (i ruoli protetti non sono modificabili)
- **Assegnazione ruoli utenti** — associazione di uno o più ruoli a ciascun utente

---

## 2. Ruoli predefiniti (protetti)

I ruoli protetti sono creati dal sistema e **non possono essere modificati né eliminati**.

| Ruolo | Descrizione | Livello |
|---|---|---|
| **SystemAdmin** | Accesso completo a tutte le funzionalità | Elevato |
| **Technician** | Supporto tecnico — equivalente a SystemAdmin | Elevato |
| **StoreManager** | Gestione punto vendita — tutto tranne moduli sistema, manutenzione e gestione ruoli | Elevato |
| **RoleManager** | Gestione ruoli e permessi, visualizzazione utenti | Elevato |
| **UserAccessManager** | Assegnazione ruoli agli utenti (solo ruoli non elevati) | Standard |
| **ShiftManager** | Dashboard, statistiche, dispositivi POS | Standard |
| **BillingOperator** | Fatturazione, fatture elettroniche, note credito, clienti | Standard |
| **ReportViewer** | Dashboard e statistiche (sola lettura) | Standard |
| **PriceCheckerAdmin** | Modulo PPC — visualizzazione e gestione | Standard |

!!! info "Ruoli elevati vs standard"
    I ruoli **elevati** (SystemAdmin, Technician, StoreManager, RoleManager) hanno accesso a funzionalità di amministrazione del sistema. I ruoli **standard** hanno accesso limitato alle funzionalità operative.

---

## 3. Permessi disponibili per i ruoli custom

### Permessi assegnabili

Quando si crea un ruolo custom, è possibile selezionare tra tutti i permessi dell'applicazione. Tuttavia, alcuni permessi sono **riservati** ai ruoli protetti e non possono essere assegnati a ruoli custom.

### Permessi riservati (blacklist)

I permessi che appartengono **esclusivamente** ai ruoli elevati sono automaticamente bloccati per i ruoli custom. Questi includono:

| Permesso | Descrizione |
|---|---|
| `store.users.create` | Creazione utenti |
| `store.users.update` | Modifica utenti |
| `store.users.delete` | Eliminazione utenti |
| `store.system.settings.view` | Visualizzazione impostazioni di sistema |
| `store.system.settings.manage` | Gestione impostazioni di sistema |
| `store.system.modules.manage` | Gestione moduli |
| `store.system.maintenance` | Manutenzione sistema |
| `store.roles.manage` | Gestione ruoli |
| `store.devices.pos.manage` | Gestione dispositivi POS |
| `store.celiac.view` | Visualizzazione celiachia |
| `store.data.management.view` | Visualizzazione gestione dati |
| `store.data.management.manage` | Gestione dati |
| `store.notifications.manage` | Gestione notifiche |

!!! warning "Sicurezza"
    La blacklist è calcolata automaticamente dal sistema. Non è possibile aggirare questa protezione: anche modificando un ruolo custom esistente, l'intero set di permessi viene rivalidato.

### Permessi assegnabili ai ruoli custom

I seguenti permessi sono disponibili per i ruoli custom:

| Permesso | Descrizione |
|---|---|
| `store.dashboard.view` | Visualizzazione dashboard |
| `store.billing.documents.view` | Visualizzazione documenti di fatturazione |
| `store.billing.invoices.create` | Creazione fatture |
| `store.billing.invoices.delete` | Eliminazione fatture |
| `store.billing.creditnotes.create` | Creazione note di credito |
| `store.billing.customers.view` | Visualizzazione clienti |
| `store.billing.customers.manage` | Gestione clienti |
| `store.billing.settings.manage` | Gestione impostazioni fatturazione |
| `store.statistics.view` | Visualizzazione statistiche |
| `store.devices.pos.view` | Visualizzazione dispositivi POS |
| `store.users.view` | Visualizzazione utenti |
| `store.ppc.view` | Visualizzazione modulo PPC |
| `store.ppc.manage` | Gestione modulo PPC |
| `store.notifications.view` | Visualizzazione notifiche |
| `store.roles.view` | Visualizzazione ruoli |
| `store.user-roles.view` | Visualizzazione assegnazione ruoli |
| `store.user-roles.manage` | Gestione assegnazione ruoli |

---

## 4. Gestione ruoli

### Accesso

La sezione è accessibile dal menu laterale **Sistema → Ruoli utente**.

!!! note "Permessi richiesti"
    - **Visualizzazione**: permesso `store.roles.view`
    - **Creazione/Modifica/Eliminazione**: permesso `store.roles.manage`

### Elenco ruoli

La pagina mostra una tabella con tutti i ruoli del sistema:

| Colonna | Descrizione |
|---|---|
| Nome | Nome del ruolo |
| Descrizione | Descrizione testuale |
| Tipo | **Protetto** (ruolo di sistema) o **Custom** (ruolo personalizzato) |
| Permessi | Numero di permessi associati |
| Azioni | Modifica ed elimina (solo per ruoli custom) |

### Creare un ruolo custom

1. Accedere a **Sistema → Ruoli utente**
2. Cliccare **Nuovo ruolo**
3. Compilare il form:
    - **Nome** — obbligatorio, massimo 50 caratteri, deve essere univoco
    - **Descrizione** — opzionale, massimo 500 caratteri
    - **Permessi** — selezionare almeno un permesso dalla checklist
4. Cliccare **Crea**

!!! tip "Permessi raggruppati"
    I permessi sono organizzati per area funzionale (es. `store.billing`, `store.devices`). I permessi riservati sono disabilitati e contrassegnati con un'icona di lucchetto.

### Modificare un ruolo custom

1. Nella lista ruoli, cliccare l'icona **matita** sul ruolo da modificare
2. Aggiornare nome, descrizione o permessi
3. Cliccare **Aggiorna**

!!! warning "Validazione completa"
    Ad ogni modifica, l'intero set di permessi viene rivalidato contro la blacklist. Non è possibile aggiungere gradualmente permessi riservati.

### Eliminare un ruolo custom

1. Nella lista ruoli, cliccare l'icona **cestino** sul ruolo da eliminare
2. Confermare l'eliminazione

!!! danger "Prerequisito"
    Un ruolo può essere eliminato solo se **nessun utente** è attualmente assegnato a quel ruolo. Rimuovere prima gli utenti dal ruolo.

---

## 5. Assegnazione ruoli agli utenti

### Accesso

La sezione è accessibile dal menu laterale **Sistema → Ruoli e permessi utenti**.

!!! note "Permessi richiesti"
    - **Visualizzazione**: permesso `store.user-roles.view`
    - **Modifica assegnazioni**: permesso `store.user-roles.manage`

### Elenco utenti

La pagina mostra tutti gli utenti visibili al ruolo corrente con i rispettivi ruoli assegnati come badge.

### Modificare i ruoli di un utente

1. Accedere a **Sistema → Ruoli e permessi utenti**
2. Cliccare l'icona **matita** sull'utente da modificare
3. Selezionare o deselezionare i ruoli desiderati
4. Cliccare **Salva**

---

## 6. Regole di sicurezza anti-escalation

Il sistema applica regole di sicurezza per impedire l'escalation dei privilegi.

### Chi può assegnare quali ruoli

| Caller | Può assegnare |
|---|---|
| **SystemAdmin / Technician** | Qualsiasi ruolo |
| **StoreManager + RoleManager** | Ruoli non elevati + UserAccessManager |
| **RoleManager** | Solo ruoli non elevati + UserAccessManager |
| **UserAccessManager** | Solo ruoli non elevati (ShiftManager, BillingOperator, ecc.) |
| **StoreManager** (senza RoleManager/UserAccessManager) | Nessuno |

### Restrizioni aggiuntive

- **Self-modification bloccata**: un utente non può modificare i propri ruoli (eccetto SystemAdmin e Technician)
- **Protezione utenti elevati**: un UserAccessManager non può modificare i ruoli di utenti che hanno ruoli elevati (SystemAdmin, Technician, StoreManager, RoleManager)
- **Visibilità filtrata**: un UserAccessManager vede nell'elenco solo gli utenti con ruoli non elevati

!!! example "Esempio pratico"
    Un utente con ruolo **UserAccessManager** può assegnare il ruolo **ShiftManager** a un operatore di cassa, ma **non può** assegnare il ruolo **StoreManager** né modificare i ruoli di un utente che è già StoreManager.

!!! info "StoreManager e gestione ruoli"
    Di default lo **StoreManager** non possiede i permessi di gestione ruoli (`store.roles.*`, `store.user-roles.*`). Qualora in una specifica installazione fosse necessario, un **SystemAdmin** o **Technician** può assegnare allo StoreManager anche il ruolo **RoleManager** (per gestione completa ruoli custom) o **UserAccessManager** (per sola assegnazione ruoli esistenti a utenti non elevati). Esempio: in un punto vendita dove il responsabile deve poter creare ruoli ad hoc per i propri operatori, un Technician assegna allo StoreManager il ruolo aggiuntivo RoleManager.

---

## 7. Voci di menu

Le voci del menu laterale nella sezione **Sistema** sono visibili solo agli utenti con i permessi appropriati:

| Voce menu | Permesso richiesto | Descrizione |
|---|---|---|
| Ruoli utente | `store.roles.view` | Gestione ruoli (creazione, modifica, eliminazione) |
| Ruoli e permessi utenti | `store.user-roles.view` | Assegnazione ruoli agli utenti |
| Gestione utenti | `store.users.view` | Gestione account utenti (creazione, modifica, reset password, 2FA) |

Se l'utente non possiede nessuno dei permessi sopra elencati, la sezione **Sistema** non è visibile nel menu laterale.

---

## 8. Audit

Tutte le operazioni sui ruoli vengono registrate nel log di audit (se abilitato):

| Azione | Evento audit |
|---|---|
| Creazione ruolo custom | `role.create` |
| Modifica ruolo custom | `role.update` |
| Eliminazione ruolo custom | `role.delete` |
| Assegnazione ruoli utente | `user.roles.assign` |

Per ogni evento vengono tracciati: utente che ha eseguito l'azione, dettagli del ruolo/permessi e timestamp.
