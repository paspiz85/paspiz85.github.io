# La mia guida all'utilizzo di Git
<p align="center">
  <img src="https://upload.wikimedia.org/wikipedia/commons/thumb/e/e0/Git-logo.svg/512px-Git-logo.svg.png" alt="logo git">
</p>
Git è un sistema di controllo delle versioni distribuito ampiamente utilizzato per gestire progetti software. Questa guida ti fornirà le informazioni di base per utilizzare al meglio Git nel tuo progetto.

## Indice
1. [Installazione e configurazione iniziale](#installazione-e-configurazione-iniziale)
2. [Inizializzazione e configurazione di un repository](#inizializzazione-e-configurazione-di-un-repository)
3. [Repository locali e remoti](#repository-locali-e-remoti)
4. [Gestione delle modifiche](#gestione-delle-modifiche)
5. [Branching](#branching)
6. [Fusione di branch](#fusione-di-branch)
7. [Rimozione di file](#rimozione-di-file)
8. [GitHub e repository remoti](#github-e-repository-remoti)

## Installazione e configurazione iniziale

Per iniziare, assicurati di avere Git installato sul tuo sistema. Puoi scaricarlo da [git-scm.com](https://git-scm.com/).

Dopo l'installazione, la configurazione può essere fatta a vali livelli:

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
A questo livello vanno aggiunte il tuo nome utente e indirizzo email per identificare le modifiche nei commit:
```bash
git config --global user.name "IlTuoNome"
git config --global user.email "tuo@email.com"
```

[Torna all'indice](#indice)

## Inizializzazione e configurazione di un repository

Nel caso di di un progetto esistente è possibile scaricare il repository con il comando:
```bash
git clone <URL_del_repository>
```
Sarà creata un cartella con all'interno tutti i file del repository, posizionarsi in tale cartella per iniziare a lavorare.

Se invece è necessario creare da zero un nuovo progetto bisogna posizionarsi nella cartella del progetto e digitare:
```bash
git init
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
git branch --set-upstream-to=<remote_alias>/<remote_branch_name> <local_branch_name>
```

Il comando **git fetch** scarica le ultime modifiche dal repository remoto specificato (o da tutti i repository se non ne viene specificato uno). Una alternativa è il comando **git pull** analogo a fetch ma in aggiunta integra (merge) eventuali modifiche remote nel branch locale corrente.

Il comando **git push** è la controparte di pull e serve ad inviare al repository remoto le modifiche del branch locale corrente. La risoluzione di eventuali conflitti deve essere risolta obbligatoriamente in locale, perciò se ci sono modifiche remote non integrate l'operazione viene bloccata: sarà necessario un ```git pull``` e rifare ```git push```.

[Torna all'indice](#indice)

## Gestione delle modifiche

Per chi inizia ad usare Git per la prima volta, può risultare difficoltoso capire tutti gli step necessari ad apportare una modifica (visibile agli altri utenti). Il concetto chiave è capire lo stato in cui si può trovare una modifica, i possibili stati infatti sono:
- *unstaged*: è lo stato in cui viene fatta una modifica ad un file senza comunicare alcunché a Git (in generale i nuovi file non vengono considerati in questo stato ma posso essere trattati allo stesso modo delle modifiche unstaged);
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
git add <elenco di uno o più file/cartelle>
```

Se c'è stata una modifica una modifica accidentale è possibile annullarla con:
```bash
git restore <elenco di uno o più file/cartelle>
```
Se tale modifica fosse già passata nello stato staged è possibile farla ritornare nello stato unstaged con:
```bash
git restore --staged <elenco di uno o più file/cartelle>
```
Successivamente sarà possibile annullarla del tutto anche dallo stato staged.

### Commit
Quando il nostro lavoro ci sembra consistente possiamo procedere a confermare tutte le modifiche fatte con la generazione di un commit: da notare che non serve che tutte le modifiche siano nello stato staged, è possibile mantenere modifiche unstaged che però saranno ignorate. Ad ogni commit va associato un messaggio che dovrebbe descrivere la modifica:
```bash
git commit -m "Descrizione del commit"
```
Il messaggio è _OBBLIGATORIO_, se non specificato va inserito tramite l'editor di testo configurato in Git. 

Nel caso fosse sfuggito qualcosa è possibile fare un secondo commit con l'opzione --amend:
```bash
git commit --amend
```
Il primo commit verrà sostituito da un commit cumulativo.


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

## 5. Branching

I branch consentono lo sviluppo parallelo di funzionalità senza interferire con il branch principale. Alcuni comandi utili:

- Creare un nuovo branch:
  ```bash
  git branch nome-del-branch
  ```

- Passare a un branch esistente:
  ```bash
  git checkout nome-del-branch
  ```

- Creare e passare a un nuovo branch:
  ```bash
  git checkout -b nuovo-branch
  ```

## 6. Fusione di branch

Dopo aver lavorato su un branch e voler unire le modifiche al branch principale:

```bash
git checkout branch-principale
git merge branch-da-unire
```

## 7. Rimozione di file

Per rimuovere un file dal repository:

```bash
git rm nome-del-file
git commit -m "Rimosso il file"
```

## 8. GitHub e repository remoti

- Collegare il repository locale a un repository su GitHub:
  ```bash
  git remote add origin https://github.com/tuo-nome-utente/nome-repository.git
  ```

- Invia i tuoi commit al repository remoto:
  ```bash
  git push -u origin branch-principale
  ```

Questa è solo una breve introduzione a Git. Per ulteriori dettagli, consulta la documentazione ufficiale di [Git](https://git-scm.com/doc). Buon sviluppo!
