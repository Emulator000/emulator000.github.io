---
authors: ["emulator000"]
date: "2022-09-22"
share: true
tags: [monolith, monoliti, team, carico cognitivo, microservizi, separazione, progetto]
title: "La separazione di un monolite"
layout: post
languageCode: "it-IT"
toc: true
---

## Procedure standard per la separazione di un progetto

Partiamo prima con un presupposto importante, qual è la dimensione giusta di un team? La risposta a questa domanda è come in tutte le cose: dipende.

La composizione di un team di sviluppo dipende da molti fattori, i componenti del team dovrebbero essere bilanciati in termini di seniority, capacity, specializzazione (back-end, front-end), interessi e preferenze personali, predisposizione analitica e cooperatività con gli altri membri del team.

Per aiutare in questo processo, ci viene in soccorso la [Conway’s Law](https://www.atlassian.com/blog/teamwork/what-is-conways-law-acmi#:~:text=Conway's%20Law%20states%20that%20%E2%80%9COrganizations,%E2%80%9CThe%20Mythical%20Man%20Month.%E2%80%9D).
Dati questi elementi, mediamente un team di sviluppo dovrebbe avere una dimensione di massimo 9 persone e per far sì che la separazione di un progetto abbia senso, bisognerebbe dapprima capire il [carico cognitivo](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC5696680/) attuale del team. Il carico cognitivo può essere definito come “la dimensione dell’effort totale utilizzato nella memoria lavorativa”.

Supponendo un team di sviluppatori con esperienza, questo molto semplicemente avrà un carico cognitivo maggiore rispetto ad un team di sviluppatori con meno esperienza.

Va quindi da se che molto sinteticamente, la dimensione del progetto dovrebbe avere una dimensione minore del carico cognitivo del team.

### Come separare un software

Per un’analisi primordiale a livello di responsabilità di un progetto, si possono utilizzare software che vanno a fare analisi statica direttamente nel codice del progetto, andando ad analizzare il repository e mappando graficamente tutti gli hot point e flame graph dei collegamenti più comuni, uno dei più famosi è senz'altro [Code Scene](https://codescene.com/), questo strumento farà anche un’analisi del coupling temporale e permetterà di visualizzare immediatamente e visivamente le macro-aree attraverso degli [Hot Spot](https://docs.enterprise.codescene.io/versions/4.0.16/guides/technical/hotspots.html).

Un altro tassello importante in questa analisi è l’accoppiamento degli eventi di un software, nel caso il progetto utilizzi il pattern Event Sourcing, sarà possibile fare coupling anche di tali eventi e quali sono le dipendenze tra essi e quindi anche capire quali sono quelli più o meno utilizzati.

Attraverso le metriche collezionate sarà possibile inoltre, unire altre informazioni utili a decretare ulteriormente quali parti il nostro progetto sta realmente utilizzando, in produzione, fornendoci dei parametri realistici e fedeli.

Ci possono venire in aiuto anche l’aggiunta di tracciamenti ad-hoc nei vari punti salienti e pubblici dell’applicativo, oltre che il contesto di dominio aziendale più globale.

Prima di disaccoppiare effettivamente un progetto, bisognerà tenere anche in considerazione il carico cognitivo dell’ulteriore frammento, e che questo, sia idoneo e adatto al nuovo sotto-team di sviluppatori nonché al nuovo carico cognitivo di esso.
