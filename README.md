# Guida alla Creazione di Scenari Obiettivo

Questo documento spiega come creare e configurare nuovi obiettivi annuali per il gioco. Gli obiettivi vengono definiti nell'array `objectiveDeck` all'interno del file `spa_game.html`.

## Struttura di un Oggetto Obiettivo

Ogni obiettivo è un oggetto JavaScript con la seguente struttura di base:

```javascript
{
    id: "identificativo_unico", // Un ID univoco per l'obiettivo
    description: "<strong>Obiettivo Annuale:</strong> Descrizione dell'obiettivo che apparirà al giocatore.",
    // La sezione 'conditions' definisce i requisiti per il punteggio.
    conditions: [
        {
            type: 'tipo_controllo', // 'resource_check' o 'stat_check'
            // parametri specifici in base al tipo
        }
    ],
    // 'scoringTiers' definisce il punteggio graduato.
    scoringTiers: [
        { target: 100, points: 300, outcomeId: 'gold' },
        { target: 50,  points: 150, outcomeId: 'silver' },
        { target: 0,   points: 0,   outcomeId: 'failure' }
    ],
    // 'outcomes' definisce i messaggi e le immagini per ogni risultato.
    outcomes: {
        'gold': {
            title: "Titolo per il risultato Oro",
            narrative: "Narrativa per il risultato Oro.",
            image: "img/percorso_immagine_oro.png"
        },
        'silver': { /* ... */ },
        'failure': { /* ... */ }
    }
}
```

### Dettagli delle Proprietà

-   `id`: Una stringa univoca che identifica l'obiettivo. Non deve ripetersi.
-   `description`: Il testo HTML che descrive la missione al giocatore all'inizio dell'anno.
-   `conditions`: Un array di una o più condizioni che devono essere verificate. Il sistema attuale supporta una logica di tipo **AND** (tutte le condizioni devono essere vere).
-   `scoringTiers`: Un array che definisce i livelli di successo. Il sistema controlla il `target` dall'alto verso il basso e assegna i `points` del primo livello raggiunto. L'ordine è importante.
-   `outcomes`: Un oggetto che collega un `outcomeId` (es. 'gold') a un titolo, una narrativa e un'immagine che verranno mostrati al giocatore alla fine dell'anno.

---

## Tipi di Obiettivo

Di seguito sono riportati esempi per i tre tipi di obiettivo richiesti.

### 1. Raccogli almeno X unità di risorsa Y

Questo obiettivo controlla la quantità di una risorsa che il giocatore possiede alla fine dell'anno.

-   **`conditions.type`**: `resource_check`
-   **`conditions.resource`**: Il nome della risorsa da controllare (es. `'meat'`, `'grain'`).

**Esempio: Accumula almeno 100 unità di Carne (`meat`).**

```javascript
{
    id: "maintain_meat",
    description: "<strong>Obiettivo Annuale:</strong> L'inverno si preannuncia rigido. Accumula più carne possibile per assicurare il benessere della tua gente.",
    conditions: [
        {
            type: 'resource_check',
            resource: 'meat'
        }
    ],
    scoringTiers: [
        // Se il giocatore ha 100 o più 'meat', ottiene 300 punti.
        { target: { meat: 100 }, points: 300, outcomeId: 'gold' },
        // Se ha tra 50 e 99 'meat', ottiene 150 punti.
        { target: { meat: 50 },  points: 150, outcomeId: 'silver' },
        // Altrimenti, 0 punti.
        { target: { meat: 0 },   points: 0,   outcomeId: 'failure' }
    ],
    outcomes: {
        'gold': {
            title: "Dispense Reali",
            narrative: "Le tue scorte di carne sono abbondanti! Hai guadagnato 300 punti.",
            image: "img/meat_gold.png"
        },
        'silver': {
            title: "Inverno Tranquillo",
            narrative: "Hai abbastanza carne per superare l'inverno. Hai guadagnato 150 punti.",
            image: "img/meat_silver.png"
        },
        'failure': {
            title: "Razioni Limitate",
            narrative: "Le tue scorte di carne sono basse. Hai guadagnato 0 punti.",
            image: "img/meat_failure.png"
        }
    }
}
```

---

### 2. Vendi X unità di risorsa Y

Questo obiettivo non controlla la quantità finale di una risorsa, ma quante unità ne sono state **vendute** durante l'anno. Per fare ciò, è necessario:
1.  Usare un `stat_check`.
2.  Aggiungere un'azione `trackSale` al pulsante di vendita corrispondente.

-   **`conditions.type`**: `stat_check`
-   **`conditions.stat`**: Il nome della statistica da tracciare (es. `'fish_sold_this_year'`). Questa statistica deve essere azzerata all'inizio di ogni anno (il sistema lo fa in automatico).

**Esempio: Vendi più Pesce (`fish`) che puoi.**

**Passo 1: Definire l'obiettivo.**

```javascript
{
    id: "sell_fish",
    description: "<strong>Obiettivo Annuale:</strong> I mercati del pesce sono vuoti. Vendi più pesce che puoi per ottenere un grande bonus.",
    conditions: [
        {
            type: 'stat_check',
            stat: 'fish_sold_this_year' // La statistica che tracceremo
        }
    ],
    scoringTiers: [
        { target: { fish_sold_this_year: 301 }, points: 500, outcomeId: 'gold' },
        { target: { fish_sold_this_year: 101 }, points: 200, outcomeId: 'silver' },
        { target: { fish_sold_this_year: 50 },  points: 150, outcomeId: 'bronze' },
        { target: { fish_sold_this_year: 0 },   points: 0,   outcomeId: 'failure' }
    ],
    outcomes: { /* ... */ }
}
```

**Passo 2: Modificare il pulsante di vendita.**

Nel database `gameLocations`, trova il pulsante che vende il pesce e aggiungi l'azione `trackSale`.

```javascript
const gameLocations = {
    "location_6": { // Villaggio di Pescatori
        // ...
        buttons: [
            // ...
            {
                id: "fish_sell",
                text: "Vendi Pesce",
                actions: [
                    { type: "removeResource", resource: "fish", amount: 30 },
                    { type: "addResource", resource: "shillings", amount: 20 },
                    // QUESTA AZIONE AGGIORNA LA STATISTICA
                    { type: "trackSale", resource: "fish", amount: 30 },
                    { type: "showMessage", title: "Pesce Venduto", message: "Hai venduto il pesce per 20 scellini." }
                ]
            }
        ]
    }
};
```

Il sistema ora sommerà la quantità di pesce venduto ogni volta che il giocatore preme quel pulsante e la confronterà con i `scoringTiers` a fine anno.

---

### 3. Obiettivo Combinato (Vendi X AND Mantieni Y)

Questo tipo di obiettivo richiede di soddisfare **due o più condizioni contemporaneamente**. Per farlo, basta aggiungere più oggetti all'array `conditions`. Il sistema verificherà che il giocatore soddisfi il `target` di **tutte** le condizioni per ottenere il punteggio.

**Esempio: Vendi almeno 30 Pesce (`fish`) E mantieni almeno 50 Grano (`grain`).**

```javascript
{
    id: "sell_fish_and_keep_grain",
    description: "<strong>Obiettivo Annuale:</strong> I mercanti richiedono pesce fresco, ma la gente ha bisogno di pane. Vendi pesce per profitto, ma assicurati di avere abbastanza grano nelle riserve.",
    // Array con due condizioni
    conditions: [
        {
            type: 'stat_check',
            stat: 'fish_sold_this_year'
        },
        {
            type: 'resource_check',
            resource: 'grain'
        }
    ],
    scoringTiers: [
        // Per ottenere 400 punti, il giocatore deve aver venduto ALMENO 60 pesce E avere ALMENO 100 grano.
        { target: { fish_sold_this_year: 60, grain: 100 }, points: 400, outcomeId: 'gold' },

        // Per 200 punti, deve aver venduto ALMENO 30 pesce E avere ALMENO 50 grano.
        { target: { fish_sold_this_year: 30, grain: 50 }, points: 200, outcomeId: 'silver' },

        { target: { fish_sold_this_year: 0, grain: 0 }, points: 0, outcomeId: 'failure' }
    ],
    outcomes: {
        'gold': {
            title: "Maestro del Commercio",
            narrative: "Un capolavoro di gestione! Hai massimizzato i profitti senza sacrificare il benessere del popolo. Ottieni 400 punti.",
            image: "img/gold_trade.png"
        },
        'silver': {
            title: "Abile Mercante",
            narrative: "Hai raggiunto un ottimo equilibrio tra vendita e gestione delle scorte. Ottieni 200 punti.",
            image: "img/silver_trade.png"
        },
        'failure': {
            title: "Gestione Difficile",
            narrative: "Non sei riuscito a bilanciare le due richieste. Nessun punto bonus quest'anno.",
            image: "img/failure_trade.png"
        }
    }
}
```

**Importante:** Per questo tipo di obiettivo, la proprietà `target` in `scoringTiers` non è più un numero, ma un **oggetto**. Le chiavi di questo oggetto devono corrispondere alla risorsa (`resource`) o statistica (`stat`) definita nelle `conditions`.

Assicurati sempre di aver configurato anche l'azione `trackSale` per la parte di vendita dell'obiettivo!
