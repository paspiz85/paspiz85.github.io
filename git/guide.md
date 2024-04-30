# La mia guida all'utilizzo di Git
<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e0/Git-logo.svg/512px-Git-logo.svg.png" alt="logo git">
</p>

Git è un sistema di controllo delle versioni distribuito ampiamente utilizzato per gestire progetti software. Questa guida ti fornirà le informazioni di base per utilizzare al meglio Git nel tuo progetto. Nota: questa è solo una breve introduzione, per ulteriori dettagli consulta la [documentazione ufficiale di Git](https://git-scm.com/doc).

## Indice
1. [Installazione e configurazione iniziale](#installazione-e-configurazione-iniziale)
2. [Inizializzazione e configurazione di un repository](#inizializzazione-e-configurazione-di-un-repository)
3. [Repository locali e remoti](#repository-locali-e-remoti)
4. [Gestione delle modifiche](#gestione-delle-modifiche)
   1. [Modifiche staged e unstaged](#modifiche-staged-e-unstaged)
   2. [Commit e log delle modifiche](#commit-e-log-delle-modifiche)
   3. [Esclusione dal versioning](#esclusione-dal-versioning)
   4. [Stash](#stash)
5. [Branching](#branching)
6. [Merge e rebase](#merge-e-rebase)
   1. [Cherry-Pick](#cherry-pick)
   2. [Squash](#squash)
   3. [Pull request](#pull-request)
7. [Tagging](#tagging)
8. [Altre funzioni utili](#altre-funzioni-utili)
9. [Link utili](#link-utili)

## Installazione e configurazione iniziale

Per iniziare, assicurati di avere Git installato sul tuo sistema. Puoi scaricarlo da [git-scm.com](https://git-scm.com/).

Dopo l'installazione, la configurazione può essere fatta a vari livelli:

1. **Configurazione di sistema**, valida per tutti gli utenti e solitamente memorizzata in ```/etc/gitconfig``` (su sistemi Unix-like) o ```C:\Program Files\Git\etc\gitconfig``` (su Windows).
2. **Configurazione utente**, valida solo per l'utente corrente e solitamente memorizzata in ```~/.gitconfig``` o ```~/.config/git/config```.
3. **Configurazione di repository**, valida solo per il repository corrente e memorizzata nella sottocartella ```.git/config```.

Le configurazioni ad un livello più specifico sovrascrivono quelle ad un livello più generico.

Le configurazioni di sistema possono essere visualizzate con:
```bash
git config --system -l
```
Un esempio di output:
```bash
http.sslcainfo=C:/Program Files/Git/mingw64/ssl/certs/ca-bundle.crt
http.sslbackend=openssl
diff.astextplain.textconv=astextplain
core.autocrlf=true
core.fscache=true
core.symlinks=false
core.editor=nano.exe
credential.helper=manager
```

Le configurazioni utente possono essere visualizzate con:
```bash
git config --global -l
```
Per il corretto funzionamento di Git, è importante a questo livello aggiungere il tuo utente e indirizzo email per identificare le modifiche nei commit:
```bash
git config --global user.name "Il_Tuo_Nome"
git config --global user.email "tua@email.com"
```

[Torna all'indice](#indice)

## Inizializzazione e configurazione di un repository

Nel caso di di un progetto esistente è possibile scaricare il repository con il comando:
```bash
git clone URL_del_repository
```
Sarà creata un cartella con all'interno tutti i file del repository, posizionarsi in tale cartella per iniziare a lavorare.

Se invece è necessario creare da zero un nuovo progetto bisogna posizionarsi nella cartella del progetto e digitare:
```bash
git init
```
Sarà poi necessario aggiungere un repository remoto con:
```bash
git remote add origin URL_del_repository
```

Le configurazioni di repository possono essere visualizzate con:
```bash
git config -l
```

[Torna all'indice](#indice)

## Repository locali e remoti

Git è un sistema distruibuto poichè ogni repository locale contiene una copia completa di tutta la cronologia del progetto. I repository locali vengono aggiornati utilizzando uno o più repository remoti tramite i comandi fetch (lettura) e push (scrittura). Ogni repository remoto viene identificato con un alias a cui è associato un URL (in realtà due poichè si possono utilizzare due riferimenti diversi fetch e push). Le informazioni sui repository remoti si ottengono con:
```bash
git remote -v
```

Ciascun branch locale può essere associato ad un branch remoto. Per visualizzare tutti i branch locali e le relative associazioni si puù usare il comando:
```bash
git branch -vv
```
Per aggiungere o modificare una associazione si usa il seguente comando:
```bash
git branch --set-upstream-to=remote_alias/remote_branch_name local_branch_name
```

Il comando **git fetch** scarica le ultime modifiche dal repository remoto specificato (o da tutti i repository se non ne viene specificato uno). Una alternativa è il comando **git pull** analogo a fetch ma in aggiunta integra (merge) eventuali modifiche remote nel branch locale corrente.

Il comando **git push** è la controparte di pull e serve ad inviare al repository remoto le modifiche del branch locale corrente. La risoluzione di eventuali conflitti deve essere risolta obbligatoriamente in locale, perciò se ci sono modifiche remote non integrate l'operazione viene bloccata: sarà necessario un ```git pull``` e rifare ```git push```.

[Torna all'indice](#indice)

## Gestione delle modifiche

Per chi inizia ad usare Git per la prima volta, può risultare difficoltoso capire tutti gli step necessari ad apportare una modifica (visibile agli altri utenti). Il concetto chiave è capire lo stato in cui si può trovare una modifica, i possibili stati infatti sono:
- *unstaged*: è lo stato in cui viene fatta una modifica ad un file senza comunicare alcunché a Git, in generale i nuovi file detti non tracciati non vengono considerati in questo stato ma vengono  trattati allo stesso modo delle modifiche unstaged;
- *staged*: è lo stato in cui una modifica ad un file viene dichiara pronta, serve a proteggere le modifiche volontare da quelle accidentali (che rimarranno unstaged);
- *committed*: è lo stato in cui si trova una modifica confermata che può coivolgere uno o più file, tale modifica viene anche chiamata commit e rappresenta una unità atomica della cronologia del progetto. I commit vanno fatti sempre in locale e solo successivamente possono essere resi pubblici con ```git push```;
- *pushed*: non è un vero e proprio stato ma serve ad indicare la differenza tra un commit locale e un commit remoto (pubblicato tramite ```git push```). Solo questi ultimi infatti saranno visibili agli altri utenti.

### Modifiche staged e unstaged
Per visualizzare tutte le modifiche locali (non ancora committed):
```bash
git status
```

Per passare le modifiche di uno o più file da unstaged a staged si usa add (oppure rm per le cancellazioni di file/cartelle):
```bash
git add elenco_di_uno_o_più_file_o_cartelle
```

Se c'è stata una modifica una modifica accidentale è possibile annullarla con ```checkout --``` oppure ```restore```:
```bash
git checkout -- elenco_di_uno_o_più_file_o_cartelle
```
Se tale modifica fosse già passata nello stato staged è possibile farla ritornare nello stato unstaged con:
```bash
git restore --staged elenco_di_uno_o_più_file_o_cartelle
```
Successivamente sarà possibile annullarla del tutto anche dallo stato staged.

Per rimuovere tutti i file e le cartelle non tracciati si usa:
```bash
git clean -d -f
```

### Commit e log delle modifiche
Quando il nostro lavoro ci sembra consistente possiamo procedere a confermare tutte le modifiche fatte con la generazione di un commit: da notare che non serve che tutte le modifiche siano nello stato staged, è possibile mantenere modifiche unstaged che però saranno ignorate. Ad ogni commit va associato un messaggio che dovrebbe descrivere la modifica:
```bash
git commit -m "Descrizione del commit"
```
Il messaggio è _OBBLIGATORIO_, se non specificato va inserito tramite l'editor di testo configurato in Git. Al termine il commit sarà identificato univocamente con un UUID ed inserito nella cronologia insieme alle informazioni dell'autore (nome ed e-mail).

Nel caso fosse sfuggito qualcosa è possibile fare un secondo commit con l'opzione --amend:
```bash
git commit --amend
```
Il primo commit verrà sostituito da un commit cumulativo.

*ATTENZIONE*: l'operazione di push è fondamentale per pubblicare un commit ed il comando commit non fornisce un meccanismo per automatizzare tale operazione. Tuttavia Git permette la definizione di alias per automatizzare questi due comandi:
```bash
git config --global alias.commit-push '!f() { git commit -m "$1" && git push; }; f'
```
Ora sarà possibile eseguire:
```bash
git commit-push "Descrizione del commit"
```

Per visualizzare la cronologia di tutti i commit (del branch corrente):
```bash
git log
```

### Esclusione dal versioning
Nel caso in cui sia necessario escludere file/cartelle dal versionamento si usa l'apposito file di testo **.gitignore**.

### Stash
In realtà esisterebbe anche un ulteriore stato chiamato *stashed* in cui modifiche staged ed unstaged posso essere messe in una sorta di stack che (LIFO) utile per accantonare modifiche da utilizzare in un secondo momento. Per mettere da parte tutte le modifiche il comando è:
```bash
git stash
```
Per ripristinare le ultime modifiche messe da parte:
```bash
git stash pop
```

[Torna all'indice](#indice)

## Branching

I branch consentono lo sviluppo parallelo di funzionalità senza interferire con il branch principale. Alcuni comandi utili:

- Creare un nuovo branch dal branch corrente:
  ```bash
  git branch nome_branch
  ```

- Passare a un branch esistente:
  ```bash
  git checkout nome_branch
  ```

- Creare e passare a un nuovo branch dal branch corrente:
  ```bash
  git checkout -b nome_branch
  ```

- Creare un branch da uno specifico commit:
  ```bash
  git branch nome_branch commit_hash
  git checkout nome_branch
  git push --set-upstream remote_alias nome_branch
  ```

- Visualizzare l'elenco di tutti i branch locali con i relativi riferimenti remoti:
  ```bash
  git branch -vv
  ```

- Visualizzare l'elenco di tutti i commit di tutti i branch:
  ```bash
  git show-branch
  ```

- Copiare file/cartelle da uno specifico branch:
  ```bash
  git checkout nome_branch elenco_di_uno_o_più_file_o_cartelle
  ```

- Copiare tutti i file e le cartelle da uno specifico branch:
  ```bash
  git checkout nome_branch .
  ```

- Cancellare un branch locale:
  ```bash
  git branch -D nome_branch
  ```

- Sincronizzare la cancellazione dei branch remoti:
  ```bash
  git fetch --prune origin
  git branch -vv | grep ': gone]' | awk '{print $1}' | xargs git branch -d
  ```

[Torna all'indice](#indice)

## Merge e rebase

Dopo aver lavorato su un branch, prima o poi sarà necessario unire le modifiche al branch principale:

```bash
git checkout branch-principale
git merge branch-da-unire
```

Può succedere che qualcun'altro abbia apportato modifiche al branch principale ed in tal caso durante il merge si potrebbero verificare dei conflitti: uno o più file potrebbero essere stati modificati in entrambi i branch. In molti casi Git riesce a risolvere automaticamente i conflitti (se le modifiche sono su porzioni differenti del file), ma negli altri sarà nostro compito decidere come risolverli. La risoluzione dei conflitti potrebbe essere piuttosto lunga e laboriosa perciò si consigliato dei tool grafici (ad esempio Git GUI), per concludere il merge bisogna che tutti i conflitti siano stati marcati come *resolved* ed effettuare una operazione di ```git commit```. Nel caso si voglia rinunciare al merge:
```bash
git merge --abort
```

Se, prima di effettuare un merge, ci accorgiamo di modifiche al branch principale dovremmo aggiornare il nostro branch con tali modifiche:
```bash
git rebase origin/branch-principale
```

### Cherry-Pick:
Per copiare un singolo commit nel branch corrente:
```bash
git cherry-pick commit_hash
```
Verrà generato un nuovo commit (con identificativo diverso) nel branch corrente che conterrà esattamente le stesse modifiche del commit originale.

### Squash
Lo squash è un'operazione che consente di combinare o "schizzare" diversi commit in un singolo commit. Questa tecnica è spesso utilizzata per semplificare la cronologia del repository, unendo una serie di piccoli commit in uno più significativo prima di inviarli al repository remoto. Utilizzo di Squash:

1. **Identificare i Commit da Squashare:**
   - Esegui `git log` per visualizzare la cronologia dei commit e individua i commit che desideri combinare. Ogni commit avrà un hash univoco associato.

2. **Eseguire il Comando di Squash:**
   - Utilizza il comando `git rebase -i HEAD~n`, dove `n` è il numero di commit indietro rispetto a HEAD che desideri includere nel processo di rebase interattivo.

3. **Modificare l'Editor Interattivo:**
   - Verrà aperto un editor di testo con una lista di commit. Cambia `pick` in `squash` o `s` per i commit che vuoi unire.

4. **Risolvere i Conflitti se Necessario:**
   - Se ci sono conflitti durante il processo di rebase interattivo, risolvi ciascun conflitto seguendo le istruzioni.

5. **Inserire un Nuovo Messaggio di Commit:**
   - Dopo lo squash, verrà richiesto di inserire un nuovo messaggio di commit che rappresenti le modifiche combinate. Salva e chiudi l'editor.

6. **Forzare il Push se Necessario:**
   - Se hai già inviato i commit al repository remoto, potrebbe essere necessario eseguire un push forzato (`git push -f`) per sovrascrivere la cronologia esistente.

Considerazioni importanti:
- Lo squash è utile per mantenere la cronologia del repository pulita e comprensibile.
- È consigliabile utilizzare lo squash solo su commit locali non ancora inviati al repository remoto per evitare problemi di compatibilità.
- Puoi combinare anche commit di branch diversi se desideri consolidare le modifiche correlate.

Ricorda sempre di fare attenzione quando utilizzi il rebase interattivo e lo squash, specialmente quando si tratta di commit condivisi con altri membri del team.

Una alternativa allo squash è creare un nuovo branch che accorpa tutte le modifiche di un altro branch in un unico commit, basta individuare uno specifico commit o branch remoto (origin/nome_branch):
```bash
git checkout branch_da_copiare
git rebase commit_iniziale
git checkout commit_iniziale
git checkout -b nuovo_branch
git checkout branch_da_copiare .
git commit -m "Descrizione della modifica"
```

### Pull request

Il sistema delle pull request (PR) non è nativo di Git ma rapprenta uno strumento comune nelle piattaforme di hosting di repository Git come GitHub, GitLab e Bitbucket. Sono utilizzate per agevolare il processo di revisione del codice e l'integrazione delle modifiche nelle branch principali di un progetto. Un esempio di workflow con questo strumento è il seguente:

1. **Creazione di una Branch:**
   - Un collaboratore crea una nuova branch nel repository per implementare le modifiche proposte.

2. **Sviluppo delle Modifiche:**
   - Il collaboratore effettua le modifiche richieste nella sua branch locale.

3. **Push della Branch Remota:**
   - La branch contenente le modifiche viene caricata sul repository remoto.

4. **Creazione della Pull Request:**
   - Il collaboratore crea una pull request per richiedere l'integrazione delle sue modifiche nella branch principale del progetto.

5. **Revisione del Codice:**
   - Altri membri del team revisionano il codice proposto nella pull request, fornendo feedback attraverso commenti.

6. **Modifiche e Discussioni Aggiuntive:**
   - Il creatore della pull request apporta eventuali modifiche in risposta ai feedback ricevuti.

7. **Merge della Pull Request:**
   - Dopo l'approvazione, un membro autorizzato esegue il merge delle modifiche nella branch principale.

8. **Chiusura della Pull Request:**
   - Dopo il merge, la pull request può essere chiusa, indicando il completamento del processo.

I principali vantaggi delle Pull Request sono:

- *Collaborazione Efficiente:* Le pull request agevolano la collaborazione e la revisione del codice in un ambiente controllato.
  
- *Tracciabilità delle Modifiche:* Ogni pull request offre una tracciabilità chiara delle modifiche proposte e delle discussioni correlate.

- *Miglioramento della Qualità del Codice:* La revisione del codice attraverso pull request aiuta a garantire che solo le modifiche di alta qualità vengano integrate nella branch principale.

Le pull request sono uno strumento essenziale per la gestione del codice sorgente in progetti collaborativi.

Può essere utile creare un alias per la creazione di una Pull Request:
```bash
git config --global alias.pr '!start $(git config --get remote.origin.url | sed 's/\.git$//')/compare/${1:-master}...$(git symbolic-ref --short HEAD)'
```

[Torna all'indice](#indice)

## Tagging
I tag sono etichette che vengono utilizzate per contrassegnare punti specifici nella cronologia di un repository. Un tag è una referenza fissa ed immutabili che punta ad un commit particolare e può essere utilizzato per identificare versioni stabili, rilasci software o punti di riferimento importanti nel tuo progetto.

- Creare un nuovo tag:
  ```bash
  git tag -a nome-del-tag -m "Descrizione del tag"
  ```

- Passare a un tag esistente:
  ```bash
  git checkout tags/nome-del-tag
  ```

- Visualizzazione dei tag esistenti:
  ```bash
  git tag -l

- Cancellare uno o più tag in remoto:
  ```bash
  git tag -l "nome-del-tag" | xargs git push origin --delete
  ```

- Sincronizzare la cancellazione dei tag remoti:
  ```bash
  git fetch --prune origin "+refs/tags/*:refs/tags/*"
  ```

[Torna all'indice](#indice)

## Altre funzioni utili

- Aggiornare le credenziali del password manager di Windows:
  ```bash
  cmdkey /delete:git:https://git.example.com
  ```

[Torna all'indice](#indice)

## Link utili

- [Git & Tricks da Mia Mamma Usa Linux](https://www.miamammausalinux.org/2023/10/git-tricks-pillole-di-source-code-management-parte-1-lambiente/)

[Torna all'indice](#indice)

