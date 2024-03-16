# Architettura di Moonbetting

Sommario
=================

<!--ts-->
* [Introduzione](#introduzione)
  * [Progetto](#progetto)
  * [Azienda](#azienda)
  * [Obiettivi e motivazioni](#obiettivi-e-motivazioni)
* [Architettura a microservizi](#architettura-a-microservizi)
  * [Beckend](#backend)
  * [Microservizi Principali](#microservizi-principali)
  * [Feign](#feign)
  * [AMQP](#amqp)
  * [Frontend](#frontend)
  * [Database](#database)
  * [Eureka](#eureka)
  * [Gateway](#gateway)
* [Tecnologie utilizzate](#tecnologie-utilizzate)
  * [Panoramica](#panoramica)
  * [Client - Frontend/UI](#frontend-ui)
  * [Librerie e Plugin](#librerie-e-plugin)
  * [Altro](#altro)
  * [Servizi e strumenti esterni](#servizi-e-strumenti-esterni)
* [Installazione](#installazione)
  * [Script di pipeline](#script-di-pipeline)
* [Documentazione API](#documentazione-api)
* [Importazione progetto in IntelliJ](#importazione-progetto-in-intellij)

<!--te-->





# Introduzione

In questo capitolo vengono introdotti il progetto, gli obiettivi, contesto aziendale e la struttura del documento.

## Progetto
Il progetto **Moonbetting** vede la realizzazione di una parte Backend, sviluppata utilizzando il framework Java Spring, e di una parte Frontend sviluppata con il framework ReactJs. Tutto il codice sorgente dei microservizi e del Frontend è disponibile su [GitHub](https://github.com/guglielmonavarra). Da tale repository il codice sorgente viene prelevato, compilato e messo in produzione da una pipeline CI/CD.


## Azienda
Mondogaming è un’azienda che fornisce soluzioni software nel campo del betting e scommesse sportive, proiettato nel business B2B e B2C. Già presente in Italia, Africa, Sud America, Europa e Asia, con un piano di espansione nel resto del mondo.


## Obiettivi e motivazioni
L’obiettivo principale è stato quello di realizzare una nuova piattaforma di gioco certificata sia in Italia che Regno Unito. 




# Architettura a microservizi

Questa tipologia di architettura attua una divisione a livello funzionale. Infatti si mira alla realizzazione di una singola applicazione composta da **N** servizi sviluppati in maniera indipendente secondo il principio della singola responsabilità. La comunicazione tra i microservizi avviene utilizzando il  protocollo HTTP tramite API RESTful.

I vantaggi principali di questo tipo di architettura sono: 

 - applicazioni altamente scalabili: man mano che la domanda per determinati servizi aumenta, i microservizi possono essere distribuiti su più server e infrastrutture, in base alle esigenze aziendali;
 - facilità di mantenimento, correzione e aggiornamento in quanto i servizi vengono sviluppati e distribuiti indipendentemente;
 - eliminazione dei singoli punti di guasto: è difficile che un problema possa riflettersi sull’intero sistema, solitamente è possibile isolare il servizio difettoso fino al momento della riparazione senza compromettere le funzionalità dell’applicazione;
 - time-to-market più rapido: consentendo di abbreviare i cicli di sviluppo, un'architettura basata su microservizi supporta deployment e aggiornamenti più agili;
 - accessibilità: per gli sviluppatori è molto più semplice comprendere, aggiornare e migliorare tali componenti permettendo di accelerare i cicli di sviluppo;
 - maggiore apertura: grazie alle API indipendenti dal linguaggio, gli sviluppatori sono liberi di scegliere la tecnologia e il linguaggio ottimali per la funzione da creare.


![Architettura](https://github.com/guglielmonavarra/technical-documentation/blob/main/Architecture.png?raw=true)


## Backend

Come è possibile vedere dall'immagine il Backend è composto da 13 microservizi che interagiscono fra loro. I microservizi possono interagire tra loro in maniera sincrona utilizzando il protocollo HTTP oppure in maniera asincrona utilizzando il protocollo AMQP.


## Microservizi principali
Oltre ai servizi di utilità, come Gateway ed Eureka, i restanti microservizi si occupano di precisi  compiti rispettando il principio di responsabilità. In particolare:

- **Oauth2**: si occupa dell'autenticazione degli utenti e il rilascio di token, nonché di alcune informazioni statistiche rilevanti legate agli accessi. Il token prodotto da tale microservizio contiene al suo interno informazioni relative al ruolo e ai permessi dell'utente, nonché dell'identificativo dell'utente, compresa la username;
- **Account**: si occupa della gestione degli account utente sia verso la piattaforma di gioco che verso il mondo backoffice. Interagisce con i microservizi **Wallet** e **Messaging** per le rispettive operazioni sul wallet o sull'invio di messaggi. Tale microservizio rappresenta il punto focale per tutti gli altri microservizi che hanno bisogno di acquisire le informazioni circa l'utente;
- **Casino**: da un lato si occupa dell'integrazione di tutti i partner di gaming di terze parti e dall'altro di fornire una interfaccia comune lato piattaforma di gioco.  Interagisce con i microservizi **Account** e **Wallet** per l'acquisizione delle informazioni dell'utente e delle transazioni di gioco durante le sessioni;
- **Gaming**: si occupa della gestione del coupon e del calcolo di tutte le forme di ticket, dalla singola, multipla o integrale). Si occupa della gestione del rischio durante le giocate. Interagisce con i microservizi **Account** e **Wallet** per l'acquisizione delle informazioni dell'utente e delle transazioni di gioco durante le sessioni;
- **Sport**: si occupa della gestione del palinsesto sia Live che Prematch nonché l'acquisizione delle quote di gioco da tutte le fonti disponibili per la piattaforma;
 - **Wallet**: si occupa della gestione del conto gioco dal punto di vista contabile. Tale microservizio è responsabile delle transazioni da tutte le aree tematiche di gioco nonché del motore relativo ai bonus utente;
 - **Payment**: si occupa dell'integrazione con i sistemi di pagamento previsti per la piattaforma.  Consente di eseguire prelievi e versamenti e di movimentare quindi il wallet dell'utente. Interagisce con i microservizi **Account** e **Wallet** per l'acquisizione delle informazioni dell'utente e della movimentazione dei saldi;
 - **Commission**: si occupa dell'elaborazione delle provvigioni su tutta sezione di affiliazione della piattaforma. Interagisce con i microservizi **Account** e **Wallet** per l'acquisizione delle informazioni dell'utente e della loro movimentazione economica;
 - **Messaging**: si occupa dell'interfacciamento con le API di terze parti (attualmente Mailgun) per l'invio di messaggi di posta elettronica e di tutta la messagistica interna della piattaforma; 
 - **Media**: si occupa dello storage di tutte le tipologie dei documenti/allegati necessari alla piattaforma. Fornisce sia funzionalità di storage che di visualizzazione dei contenuti tipici delle piattaforme di File Storage come ad esempio **AWS S3**;  
 - **Logger**: si occupa dello storage di tutti i LOG/EVENTI che ogni microservizio può generare e vuole catalogare;   




## Feign

Feign è una libreria Java open-source che semplifica notevolmente il processo di effettuare richieste web in applicazioni basate su microservizi. La sua funzionalità principale consiste nell'offrire un'interfaccia semplice e intuitiva per la chiamata di servizi web RESTful, riducendo così al minimo il lavoro di sviluppo necessario.

Una delle caratteristiche distintive di Feign è la sua capacità di fornire un'astrazione di alto livello sopra le richieste HTTP. Ciò significa che gli sviluppatori possono concentrarsi sulla definizione di interfacce di servizio e delegare a Feign tutto il lavoro di gestione delle richieste e delle risposte HTTP sottostanti. Questo porta a una riduzione significativa del boilerplate code, ovvero del codice ripetitivo e di basso livello, che altrimenti sarebbe necessario per effettuare chiamate HTTP manualmente.

Questo approccio semplifica notevolmente lo sviluppo di client per servizi web RESTful, consentendo agli sviluppatori di scrivere codice più pulito, leggibile e manutenibile. Inoltre, l'uso di Feign favorisce una maggiore coerenza nell'implementazione delle chiamate ai servizi, poiché le logiche di comunicazione sono centralizzate all'interno delle interfacce dichiarate.



## AMQP
AMQP, o Advanced Message Queuing Protocol, è un protocollo di messaggistica progettato per facilitare lo scambio affidabile e scalabile di messaggi tra applicazioni e sistemi distribuiti. Offre affidabilità nella consegna dei messaggi, flessibilità nei modelli di messaggistica, interoperabilità tra diverse implementazioni, scalabilità per gestire grandi volumi di messaggi e funzionalità di sicurezza avanzate. 
Tutti i microservizi hanno la possibilità di comunicare l'uno con l'altro in modalità asincrona tramite code **RabbitMQ**. In particolare attualmente il sistema gestisce le seguenti code:

- **registration**: contiene tutto il payload generato durante la registrazione di un utente dalla sezione pubblica;
- **request**: contiene tutte le request che vengono richieste alle API dell'intera piattaforma;
- **ticket**: contiene tutti i ticket che vengono giocati e che magari devono essere sottoposti a modifica da parte del reparto rischio per una attenta visione o revisione;
- **transaction**: contiene tutte le transazioni che vengono generate dalla piattaforma;



## Frontend
React è una libreria JavaScript per creare interfacce utente. Esso è basato su un modello di tipo MVVM (Model View View-Model) che sfrutta la teconologia di virtual dom per ottimizzare il rendering delle interfacce grafiche. React permette di utilizzare un meccanismo di templating mediante l’utilizzo di componenti e attraverso un approccio dichiarativo. Questo permette una gestione più semplice del frontend, una separazione netta degli elementi grafici in dei componenti, rendendone più facile il riutilizzo e il testing, e introduce uno stato a dei componenti grafici. Inoltre, può essere utilizzato anche nei frontend di applicazioni mobile grazie a React-Native.





## Database

Ogni microservizio interagisce con un proprio database MariaDB. Tale configurazione ci consente di diminuire il carico e rendere il sistema scalabile orizzontalmente. Tuttavia per 	raggiungere questa configurazioni è necessario aggiungere alcune ridondanze nei vari schemi dei database in modo di avere una solida integrità.


- **powerskin_user_db**: Tale database viene utilizzato, gestito e migrato dal microservizio oauth2. Rappresenta il database principale, pertanto deve essere il primo ad essere generato;
- **powerskin_wallet_db**: Tale database viene utilizzato, gestito e migrato dal microservizio wallet. E' il database in stretta relazione con quello user, pertanto deve essere il secondo ad essere generato;
- **powerskin_sport_db**: Tale database viene utilizzato, gestito e migrato dal microservizio sport;
- **powerskin_casino_db**: Tale database viene utilizzato, gestito e migrato dal microservizio casino;
- **powerskin_gaming_db**: Tale database viene utilizzato, gestito e migrato dal microservizio gaming;
- **powerskin_log_db**: Tale database viene utilizzato, gestito e migrato dal microservizio logger;
- **powerskin_media_db**: Tale database viene utilizzato, gestito e migrato dal microservizio media;
- **powerskin_messaging_db**: Tale database viene utilizzato, gestito e migrato dal microservizio messaging;
- **powerskin_payment_db**: Tale database viene utilizzato, gestito e migrato dal microservizio payment;
- **powerskin_rubyplay_db**: Tale database viene utilizzato, gestito e migrato dal microservizio casino. Per ogni servizio di terze parti verrà generato un database specifico;




## Eureka
Eureka è il componente che funge da service registry, cioè un registro in cui i vari servizi si registrano indicando il loro indirizzo fisico, la porta su cui sono in ascolto ed un service ID, rendendo così possibile ad altri servizi di interagire con questi senza dover conoscere a priori la loro posizione. Questo approccio prende il nome di service discovery ed è di estrema importanza nelle architetture distribuite: dal momento che per utilizzare un servizio non è necessario conoscerne la posizione effettiva ma solo il nome, è possibile aggiungere o togliere istanze di un certo servizio pur mantenendolo sempre raggiungibile, in questo modo aumentando la flessibilità e la scalabilità e permettendo di aumentare la resilienza dell’applicazione in caso di malfunzionamento di un servizio dato che la richiesta fallita può essere dirottata su di una replica funzionante dello stesso servizio.




## Gateway
Spring Cloud Gateway è una libreria che viene usata nelle architetture a microservizi per nascondere i servizi dietro una singola facciata. La funzione principale è quella di determinare cosa fare con le richieste che soddisfano una specifica route. La configurazione delle rotte può essere effettuata sia tramite file di configurazione sia tramite bean. I componenti principali dell’API sono le route: esse sono definite da un identificatore, un URI di destinazione e un insieme di predicati e filtri. I predicati vengono usati per controllare la corrispondenza della richiesta con gli endpoint permessi. I filtri permettono la manipolazione delle richieste e delle risposte, prima e dopo che vengano inoltrate.



# Tecnologie utilizzate

<details open="open">
	<ul>
		<li><a href="#overview">Panoramica</a></li>
		<li><a href="#client---frontend-ui">Client - Frontend/UI</a></li>
		<li><a href="#libraries-and-plugins">Librerie e Plugin</a></li>
		<li><a href="#others">Altro</a></li>
		<li><a href="#external-tools---services">Servizi e strumenti esterni</a></li>
	</ul>
</details>



### Panoramica

|Tecnologia                |Descrizione         |
|--------------------------|--------------------|
|Core Framework            |Spring Boot 2        |
|Gateway            |Spring Cloud        |
|Service Discovery            |Spring Cloud NetFlix        |
|Cache        |Spring Redis               |
|AMQP        |Spring AMQP               |
|Security Framework        |Spring Security, Oauth2|
|Authentication        |OAuth2|
|Persistent Layer Framework|MyBatis     |
|Database                  |MariaDB, H2 Database Engine               |
|Database Migration        |Flyway               |





### Client - Frontend/UI

|                 Tecnologia                               |                                           Descrizione                                                           |
|----------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
|ReactJS         |  React è la libreria per interfacce utente sia web e native. Costruisci interfacce utente da singoli pezzi chiamati componenti scritti in JavaScript.|






###  Librerie e Plugin

|                                      Tecnologia                                               |                              Descrizione                                                                                                                      |
|-----------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------|
|Lombok                                                |Lombok è una libreria di tipo _Annotation processor_ (APT) ovvero durante la compilazione del progetto esegue l’interpretazione delle cosiddette _**annotation**_ (`@NomeAnnotazione` per intenderci) dichiarate a livello di classe. In fase di compilazione Lombok esegue delle operazioni e genera in automatico il codice aggiuntivo.|
|Swagger                                                      |Swagger è un insieme di strumenti e tecnologie per la progettazione, la costruzione e la documentazione delle API RESTful. Viene utilizzato per creare documentazione interattiva, generazione di SDK e test e debug API.           |





### Altro 

|                 Tecnologia                                               |                              Descrizione                                                                                  |
|--------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
|Git                                    |Git è un software di controllo versione distribuito utilizzabile da interfaccia a riga di comando che permette una gestione efficace del codice di un progetto. Git viene implementato da GitHub, un servizio di hosting per progetti software accessibile da browser o da applicazione desktop, che permette l’interazione di altri utenti con il codice sorgente dei progetti.                                                                    |
|Gradle                                    |Gradle è uno strumento di automazione della compilazione (build automation tool) che fornisce un linguaggio specifico di dominio (DSL) basato su Groovy. Viene eseguito su JVM, oltre a consentire il supporto per la creazione di progetti nativi. Gradle consente di utilizzare repository Maven, Ivy o altri definiti dall'utente per la gestione delle dipendenze, supportando build poliglotte (oltre 60 differenti linguaggi di programmazione supportati).                                                                    |
|Slack                                    |Slack è un software di messaggistica istantanea basato sul cloud, creato nel 2013 allo scopo di facilitare la comunicazione tra i dipendenti di un’azienda. È sufficiente lanciare l’app per ritrovarsi in una chat in cui i membri di un team possono inviare messaggi in un canale condiviso o in sede privata, condividere file, effettuare chiamate e via dicendo.                                                                    |



### Servizi e strumenti esterni

|Tecnologia                |Descrizione         |
|--------------------------|--------------------|
|MailGun            |Servizio di invio messaggi SMTP        |
|Jenkins            |Jenkins è un server open source di CI/CD (Continuous Integration, integrazione continua, e Continuous Deployment, implementazione continua) scritto in Java. Jenkins consente di automatizzare le diverse fasi del ciclo di vita del software, dalla compilazione al test alla distribuzione.       |



# Installazione
L'architettura di Moonbetting è stata concepita per essere installata e gestita in CI/CD (Continuous Integration e Continuous Deployment) grazie a Jenkins. E' possibile utilizzare sia un server Jenkins locale per eseguire i nostri processi CI/CD oppure configurare un server Jenkins locale per eseguire i nostri processi su macchine AWS EC2. Per utilizzare Jenkins è necessario installare i seguenti plugin:

-   Nella dashboard di Jenkins seleziona **Gestisci Jenkins** .
-   Seleziona **Gestisci plugin** .
-   Seleziona **Plug-in disponibili** .
-   Nella casella di testo digita "Pipeline" e installa il plugin.
-   Nella casella di testo digita "Folder Properties Plugin" e installa il plugin.
-   Nella casella di testo digita "Publish Over SSH" e installa il plugin.
-   Nella casella di testo digita "SSH Pipeline Steps" e installa il plugin.


Successivamente, per ogni SKIN, in questo caso Moonbetting, bisogna creare un nuovo Folder che conterrà tutti i progetti **Multibranch Pipeline** sia dei microservizi che della parte Frontend. Ogni Folder dovrà contenere tutte le variabili d'ambiente necessarie al corretto funzionamento del progetto che verranno iniettate in fase di building:

|Variabile                |Descrizione         |
|--------------------------|--------------------|
| SPRING_PROFILE           |Modalità di esecuzione, dev o prod        |
| AWS_CI            |Se pubblicare in locale o su AWS        |
| AWS_DEVOP_SECRET_ACCESS_KEY           |Credenziali AWS         |
| AWS_DEVOP_ACCESS_KEY_ID        |Credenziali AWS                |
| AWS_S3_BUCKET        |Bucket S3               |
| AWS_ZONE        |Zona AWS|
| GLOBAL_TRANSACTION_QUEUE        |Nome della coda per le transazioni|
| GLOBAL_TRANSACTION_EXCHANGE        |Coda per le transazioni|
| GLOBAL_REQUEST_EXCHANGE     | Coda delle request
| GLOBAL_REQUEST_QUEUE     | Nome della coda delle request |
| GLOBAL_TICKET_EXCHANGE     | Coda dei ticket |
| GLOBAL_TICKET_QUEUE     | Nome della coda dei ticket |
| GLOBAL_REGISTRATION_EXCHANGE     | Nome della coda delle registrazioni|
| GLOBAL_REGISTRATION_QUEUE     | Nome della coda delle registrazioni |
| GLOBAL_RABBIT_PORT                  |Porta server RabbitMQ               |
| GLOBAL_RABBIT_USERNAME        |Username server RabbitMQ               |
| GLOBAL_RABBIT_PASSWORD        |Password server RabbitMQ               |
| GLOBAL_EUREKA_HOST        |IP del microservizio Discovery               |
| GLOBAL_VERIFICATION_HOST        |IP del microservizio OAuth2               |
| GLOBAL_VERIFICATION_PORT        |Porta del microservizio Oauth2               |
| GLOBAL_VERIFICATION_HOST        |IP del microservizio Discovery               |
| GLOBAL_DOMAIN        |Dominio della skin               |
| GLOBAL_CAPTCHA_KEY        |Chiave del servizio captcha               |
| GLOBAL_SKINPROFILE        |Profilo della skin               |
| GLOBAL_FOOTER_INFO1        |Informazioni per il footer della skin      |       
| GLOBAL_FOOTER_INFO2        |Informazioni per il footer della skin     |          
| GLOBAL_FOOTER_INFO3        |Informazioni per il footer della skin  |             
| GLOBAL_WHATSAPP_NUMBER     |Numero whatsapp del supporto               |
| GLOBAL_HELPDESK_NUMBER     |Numero telefonico del supporto               |
| GLOBAL_DEALER_AMOUNT     |Importo massimo giocabile dall'utente               |
| GLOBAL_COMPANY_PEC     |Indirizzo PEC               |
| GLOBAL_COMPANY_EMAIL     |Indirizzo Email               |
| GLOBAL_COMPANY_ADDRESS     |Indirizzo residenza dell'azienda               |
| GLOBAL_COMPANY_NAME     |Nome dell'azienda               |
| GLOBAL_COMPANY_VAT     |Partita IVA dell'azienda               |
| GLOBAL_GAD     |Numero della concessione ADM               |
| GLOBAL_SUBNET_1     |AWS subnet 1               |
| GLOBAL_SUBNET_2     |AWS subnet 2               |
| GLOBAL_SUBNET_3     |AWS subnet 3               |
| GLOBAL_CIDR     |AWS CIDR               |
| GLOBAL_CIDR     |AWS CIDR               |
| ACCOUNT_LOGNAME| Nome del file di log              |
| ACCOUNT_LOGPATH| Path del file di log              |
| ACCOUNT_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| ACCOUNT_OAUTH_RES_ID| OAuth2 client ID (da non cambiare)              |
| ACCOUNT_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| ACCOUNT_DATABASE_NAME| Nome del database              |
| ACCOUNT_DATABASE_HOST| Indirizzo IP del database               |
| ACCOUNT_DATABASE_PORT| Porta del database              |
| ACCOUNT_DATABASE_USERNAME| Username del database               |
| ACCOUNT_DATABASE_PASSWORD| Password del database               |
| ACCOUNT_BINDING_ETH| Interfaccia d'ascolto              |
| ACCOUNT_BINDING_PORT| Porta d'ascolto              |
| COMMISSION_BINDING_ETH| Interfaccia d'ascolto              |
| COMMISSION_BINDING_PORT| Porta d'ascolto              |
| COMMISSION_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| COMMISSION_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| COMMISSION_OAUTH_RES_ID| OAuth2 client ID (da non cambiare)              |
| COMMISSION_DATABASE_HOST| Indirizzo IP del database               |
| COMMISSION_DATABASE_PORT| Porta del database              |
| COMMISSION_DATABASE_NAME| Nome del database              |
| COMMISSION_DATABASE_USERNAME| Username del database               |
| COMMISSION_DATABASE_PASSWORD| Password del database               |
| COMMISSION_LOGPATH| Path del file di log              |
| COMMISSION_LOGNAME| Nome del file di log              |
| SPORT_PM_SQLDIR| Path dei file di query  per il prematch             |
| SPORT_PM_DATABASE_HOST| Indirizzo IP del database del Prematch                |
| SPORT_PM_DATABASE_PORT| Porta del database del Prematch            |
| SPORT_PM_DATABASE_NAME| Nome del database del Prematch            |
| SPORT_PM_DATABASE_USERNAME| Username del database del Prematch              |
| SPORT_PM_DATABASE_PASSWORD| Password del database del Prematch               |
| SPORT_BINDING_PORT| Porta d'ascolto              |
| SPORT_BINDING_ETH| Interfaccia d'ascolto              |
| SPORT_DATABASE_HOST| Indirizzo IP del database               |
| SPORT_DATABASE_USERNAME| Username del database               |
| SPORT_DATABASE_PASSWORD| Password del database               |
| SPORT_DATABASE_NAME| Nome del database               |
| SPORT_DATABASE_PORT| Porta del database              |
| SPORT_LOGNAME| Nome del file di log              |
| SPORT_LOGPATH| Path del file di log              |
| SPORT_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| SPORT_OAUTH_RES_ID| OAuth2 client ID (da non cambiare)              |
| SPORT_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| SPORT_OR_SQLDIR| Path dei file di query outrights              |
| SPORT_LV_SQLDIR| Path dei file di query  per il Live             |
| SPORT_LV_DATABASE_HOST| Indirizzo IP del database del Live              |
| SPORT_LV_DATABASE_PORT| Porta del database del Live             |
| SPORT_LV_DATABASE_NAME| Nome del database del Live             |
| SPORT_LV_DATABASE_USERNAME| Username del database del Live              |
| SPORT_LV_DATABASE_PASSWORD| Password del database del Live              |
| GAMING_LOGPATH| Path del file di log              |
| GAMING_LOGNAME| Nome del file di log              |
| GAMING_BINDING_ETH| Interfaccia d'ascolto              |
| GAMING_BINDING_PORT| Porta d'ascolto              |
| GAMING_OAUTH_RES_ID| OAuth2 client ID (da non cambiare)              |
| GAMING_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| GAMING_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| GAMING_DATABASE_USERNAME| Username del database               |
| GAMING_DATABASE_PASSWORD| Password del database               |
| GAMING_DATABASE_HOST| Indirizzo IP del database               |
| GAMING_DATABASE_PORT| Porta del database              |
| GAMING_DATABASE_NAME| Nome del database              |
| GAMING_TEMPLATE_DIR| Path per i template               |
| MEDIA_LOGPATH| Path del file di log              |
| MEDIA_LOGNAME| Nome del file di log              |
| MEDIA_DOCUMENT_FOLDER| Path per i documenti              |
| MEDIA_MEDIA_FOLDER| Path per i media              |
| MEDIA_PROFILE_FOLDER| Path per i profili              |
| MEDIA_BINDING_ETH| Interfaccia d'ascolto              |
| MEDIA_BINDING_PORT| Porta d'ascolto              |
| MEDIA_DATABASE_NAME| Nome del database              |
| MEDIA_DATABASE_HOST| Indirizzo IP del database               |
| MEDIA_DATABASE_PORT| Porta del database              |
| MEDIA_DATABASE_USERNAME| Username del database               |
| MEDIA_DATABASE_PASSWORD| Password del database               |
| MEDIA_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| MEDIA_OAUTH_RES_ID| OAuth2 client ID (da non cambiare)              |
| MEDIA_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| DISCOVERY_LOGNAME| Nome del file di log              |
| DISCOVERY_LOGPATH| Path del file di log              |
| DISCOVERY_BINDING_ETH| Interfaccia d'ascolto              |
| DISCOVERY_BINDING_PORT| Porta d'ascolto              |
| WALLET_DATABASE_HOST| Indirizzo IP del database               |
| WALLET_DATABASE_PORT| Porta del database              |
| WALLET_DATABASE_NAME| Nome del database              |
| WALLET_DATABASE_USERNAME| Username del database               |
| WALLET_DATABASE_PASSWORD| Password del database               |
| WALLET_OAUTH_RES_ID| OAuth2 client ID (da non cambiare)              |
| WALLET_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| WALLET_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| WALLET_LOGPATH| Path del file di log              |
| WALLET_LOGNAME| Nome del file di log              |
| WALLET_BINDING_ETH| Interfaccia d'ascolto              |
| WALLET_BINDING_PORT| Porta d'ascolto              |
| MESSAGING_MAILGUN_APIKEY| Chiave di produzione di MailGun              |
| MESSAGING_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| MESSAGING_OAUTH_RES_ID| OAuth2 client ID (da non cambiare)              |
| MESSAGING_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| MESSAGING_LOGNAME| Nome del file di log              |
| MESSAGING_LOGPATH| Path del file di log              |
| MESSAGING_BINDING_ETH| Interfaccia d'ascolto              |
| MESSAGING_BINDING_PORT| Porta d'ascolto              |
| MESSAGING_SMTP_FROM| Indirizzo per il from              |
| MESSAGING_SMTP_REPLYTO| Indirizzo per il reply to              |
| MESSAGING_SMTP_USERNAME| Username SMTP              |
| MESSAGING_SMTP_PASSWORD| Password SMTP              |
| MESSAGING_SMTP_PROTOCOL| Protocollo SMTP da utilizzare              |
| MESSAGING_MAILER_TYPE| Il servizio da utilizzare per la posta               |
| MESSAGING_SMTP_HOST| Indirizzo del server SMTP              |
| MESSAGING_SMTP_PORT| Porta SMTP              |
| MESSAGING_DATABASE_PORT| Porta del database              |
| MESSAGING_DATABASE_HOST| Indirizzo IP del database               |
| MESSAGING_DATABASE_NAME| Nome del database              |
| MESSAGING_DATABASE_USERNAME| Username del database               |
| MESSAGING_DATABASE_PASSWORD| Password del database               |
| LOG_LOGNAME| Nome del file di log              |
| LOG_LOGPATH| Path del file di log              |
| LOG_BINDING_ETH| Interfaccia d'ascolto              |
| LOG_BINDING_PORT| Porta d'ascolto              |
| LOG_DATABASE_NAME| Nome del database              |
| LOG_DATABASE_HOST| Indirizzo IP del database               |
| LOG_DATABASE_PORT| Porta del database              |
| LOG_DATABASE_USERNAME| Username del database               |
| LOG_DATABASE_PASSWORD| Password del database               |
| LOG_OAUTH_RES_ID| OAuth2 client ID (da non cambiare)              |
| LOG_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| LOG_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| OAUTH2_DATABASE_NAME| Nome del database              |
| OAUTH2_DATABASE_PORT| Porta del database              |
| OAUTH2_DATABASE_HOST| Indirizzo IP del database               |
| OAUTH2_KEYSTORE_PASS| Password della chiave privata               |
| OAUTH2_DATABASE_USERNAME| Username del database               |
| OAUTH2_DATABASE_PASSWORD| Password del database               |
| OAUTH2_BINDING_ETH| Interfaccia d'ascolto              |
| OAUTH2_BINDING_PORT| Porta d'ascolto              |
| OAUTH2_LOGNAME| Nome del file di log              |
| OAUTH2_LOGPATH| Path del file di log              |
| GATEWAY_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| GATEWAY_OAUTH_RES_ID| OAuth2 client ID (da non cambiare)              |
| GATEWAY_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| GATEWAY_LOGPATH| Path del file di log              |
| GATEWAY_LOGNAME| Nome del file di log              |
| GATEWAY_BINDING_ETH| Interfaccia d'ascolto              |
| GATEWAY_BINDING_PORT| Porta d'ascolto              |
| CASINO_DATABASE_HOST| Indirizzo IP del database               |
| CASINO_DATABASE_PORT| Porta del database              |
| CASINO_DATABASE_USERNAME| Username del database               |
| CASINO_DATABASE_PASSWORD| Password del database               |
| CASINO_DATABASE_NAME| Nome del database              |
| CASINO_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| CASINO_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| CASINO_OAUTH_RES_ID| OAuth2 resource ID (da non cambiare)              |
| CASINO_BINDING_ETH| Interfaccia d'ascolto              |
| CASINO_BINDING_PORT| Porta d'ascolto              |
| CASINO_LOGPATH| Path del file di log              |
| CASINO_LOGNAME| Nome del file di log              |
| CASINO_MEDIAFOLDER| Path dei media dei provider              |
| CASINO_CONFIGFOLDER| Path configurazione dei singoli provider              |
| CASINO_DATASOURCES| Path del file Json dei datasource              |
| CASINO_PROVIDERS| Path del file Json di configurazione dei provider             |
| PAYMENT_DATABASE_HOST| Indirizzo IP del database               |
| PAYMENT_DATABASE_PORT| Porta del database              |
| PAYMENT_DATABASE_USERNAME| Username del database               |
| PAYMENT_DATABASE_PASSWORD| Password del database               |
| PAYMENT_DATABASE_NAME| Nome del database              |
| PAYMENT_OAUTH_CLIENT_ID| OAuth2 client ID (da non cambiare)              |
| PAYMENT_OAUTH_CLIENT_SECRET| OAuth2 secret (da non cambiare)              |
| PAYMENT_OAUTH_RES_ID| OAuth2 resource ID (da non cambiare)              |
| PAYMENT_BINDING_ETH|  Interfaccia di ascolto             |
| PAYMENT_BINDING_PORT| Porta del microservizio              |
| PAYMENT_LOGPATH|  Path del file di log             |
| PAYMENT_LOGNAME| Nome del file di log              |
| PAYMENT_PAYTEST| Modalità di DEBUG pagamenti               |
| PAYMENT_AESKEY| Chiave di cifratura               |



## Script di pipeline
Ogni progetto **Multibranch Pipeline** e quindi la repository di ogni progetto, deve contenere una pipeline Jenkins includendo un file di testo, denominato **Jenkinsfile** contenente lo script della pipeline. Lo script della pipeline specifica i processi CI/CD attraverso i quali passa l'applicazione. All'interno dello script vi sono le condizioni di building locale oppure building su AWS.

L'immagine sottostante mostra lo schema a blocchi della pipeline:

![flow](https://github.com/guglielmonavarra/technical-documentation/blob/main/realworld-pipeline-flow.png?raw=true)


Il file di script **Jenkinsfile** rappresenta la cosiddetta pipeline dichiarativa. Esso rappresenta un potente strumento di Jenkins per automatizzare il processo d'ntegrazione continua (CI) e distribuzione continua (CD) di un'applicazione software. Piuttosto che utilizzare l'interfaccia grafica di Jenkins per definire passo dopo passo le varie fasi della pipeline, la pipeline dichiarativa permette di definire la pipeline in modo più strutturato e leggibile tramite un file di script chiamato.
Il  **Jenkinsfile** è un file di script scritto utilizzando la sintassi del linguaggio Groovy. Questo file definisce in modo chiaro e strutturato tutte le fasi del processo di CI/CD, inclusi i test, la compilazione, il rilascio e altro. Essendo memorizzato all'interno del repository del progetto, il Jenkinsfile facilita il versionamento.
I blocchi fondamentali contenuti all'interno dello script sono: **Agent**, **Stage**, **Step**, **Direttive**, **Variabili d'ambiente** e i **Trigger**.
    
**Agent**: specifica dove la pipeline deve essere eseguita. Può essere un nodo Jenkins, un agente Docker, una macchina virtuale o qualsiasi altra piattaforma supportata da Jenkins, compreso AWS. La scelta dell'agente dipende dalle esigenze del progetto e dalle risorse disponibili.
    
**Stage**: rappresenta una fase logica della pipeline, come "Build", "Test", "Deploy", ecc. Ogni stage contiene uno o più passaggi (steps) che devono essere completati prima di passare allo stage successivo. Questa struttura modulare e organizzata rende la pipeline più leggibile e gestibile.
    
**Step**: Uno step è un'azione specifica eseguita all'interno di uno stage. Possono essere eseguite varie azioni, come l'esecuzione di comandi di shell, la compilazione del codice, l'esecuzione dei test e così via. Gli steps possono essere definiti in modo dichiarativo nel Jenkinsfile, consentendo un maggiore controllo sul processo di build.
    
**Direttive**: forniscono istruzioni aggiuntive alla pipeline e specificano come Jenkins dovrebbe interpretare e gestire il processo di build. Le direttive comuni includono `agent`, `stages`, `steps`, `environment`, `tools` e altre. Queste direttive consentono di personalizzare la pipeline in base alle esigenze specifiche del progetto.
    
**Variabili d'ambiente**: possono essere definite per passare configurazioni o valori che devono essere utilizzati durante l'esecuzione della pipeline.
    
**Trigger**: determinano quando la pipeline deve essere eseguita. Possono essere attivati da eventi come il commit di codice nel repository di versionamento, l'invio di una richiesta di pull (pull request), un timer programmato, l'aggiornamento di una dipendenza esterna e così via. 
    
La pipeline dichiarativa di Jenkins offre un'approccio strutturato, flessibile e scalabile per automatizzare il processo di CI/CD delle applicazioni software. Grazie alla sua sintassi chiara e alla sua capacità di adattarsi alle esigenze del progetto, la pipeline dichiarativa semplifica la gestione del processo di build e distribuzione, migliorando l'efficienza e la qualità del software prodotto.



# Documentazione API

Tutta la documentazione delle API che i microservizi mettono a disposizione è fruibile dalle  relative pagine swagger. Ogni microservizio in base alla propria BINDING_PORT e al proprio BINDING_HOST è raggiungibile alle seguenti url: 

|Servizio                |Url         |
|--------------------------|--------------------|
| ACCOUNT            |http://address:port/swagger-ui/      |
| WALLET            |http://address:port/swagger-ui/      |
| MEDIA            |http://address:port/swagger-ui/      |
| LOGGER            |http://address:port/swagger-ui/      |
| MESSAGING            |http://address:port/swagger-ui/      |
| COMMISSION            |http://address:port/swagger-ui/      |
| OAUTH2            |http://address:port/swagger-ui/      |
| GAMING            |http://address:port/swagger-ui/      |
| SPORT           |http://address:port/swagger-ui/       |
| CASINO            |http://address:port/swagger-ui/      |
| PAYMENT            |http://address:port/swagger-ui/      |




# Importazione progetto in IntelliJ
Per importare il progetto di un microservizio bisogna seguire i seguenti passi:

1.  **Clonare il repository**: Prima di tutto, è necessario clonare il repository del progetto Java Spring con Gradle da GitHub sul tuo computer. Puoi farlo utilizzando Git attraverso la riga di comando o utilizzando l'interfaccia grafica di un client Git come GitKraken o GitHub Desktop.    
   
    `git clone <URL_del_repository>` 
    
2.  **Aprire IntelliJ IDEA**: Avvia IntelliJ IDEA sul tuo computer.
    
3.  **Aprire il progetto**: Dalla finestra principale di IntelliJ IDEA, seleziona "Open" e naviga fino alla directory in cui hai clonato il repository del progetto. Seleziona la cartella principale del progetto e fai clic su "Apri".
    
4.  **Importare il progetto Gradle**: Se il progetto utilizza Gradle come sistema di build, IntelliJ IDEA dovrebbe rilevarlo automaticamente e proporre di importarlo come progetto Gradle. Assicurati di selezionare l'opzione "Import Gradle project" quando richiesto.
    
5.  **Configurare le impostazioni di Gradle**: Durante il processo di importazione, IntelliJ IDEA chiederà eventuali configurazioni aggiuntive per il progetto Gradle, come la versione di Gradle da utilizzare. Di solito puoi accettare le impostazioni predefinite, ma puoi modificarle se necessario.
    
6.  **Attendere il completamento dell'importazione**: IntelliJ IDEA importerà il progetto e le dipendenze di Gradle dal repository GitHub. Questo processo potrebbe richiedere qualche minuto, a seconda delle dimensioni del progetto e della velocità della connessione internet.
        
7.  **Eseguire il progetto**: Ora che il progetto è stato importato con successo, per eseguirlo bisogna aggiungere tutti i parametri necessari nel file **application.properties**. 
Questo file deve essere editato solo se si intende eseguire il progetto in locale e le modifiche apportate al suo interno non devono mai essere riportate sulla repository, questo perché la pipeline ogni volta che esegue il deploy del progetto si occupa della generazione del file application.properties. Per capire quali parametri sono necessari basta visionare lo script **build.gradle** e capire quali acquisisce (il significato di tali variabili è stato descritto nei capitoli precedenti). Un'altra strada sarebbe quella di inserire in IntelliJ tutte le variabili d'ambiente necessarie (è possibile anche elencarle  tutte).


