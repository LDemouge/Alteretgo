#Cahier des charges
## Vocabulaire




###<a name="ui">UI</a> 

> Interface Utiliseur

***UI*** désigne l'ensemble des éléments à disposition d'un utilisateur pour envoyer des commandes à l'[API](#api) et  consulter l'état du domain model.

Sur ce projet, la ***UI*** sera de type **web responsive**, indépendamment de l'OS et du navigateur de l'utilisateur, à l'exception de IE pour lequel ce projet n'aura pas à être optimisé.

La ***UI*** est composée de différentes vues parmi lesquelles l'utilisateur doit pouvoir naviguer sans chargement ni page reload.

Le bouton back du navigateur doit renvoyer l'exact même vue que la dernière précédemment consultée.

La ***UI*** doit refléter en temps quasi-réel l'état du [Domain Model](#domainModel), sans que l'utilisateur n'ait à recharger sa page, les informations lui sont donc pushées.

La ***UI*** doit disposer de sa propre base de donnée, indépendante de l'[API](#api), que l'on appellera [Read Model](#readModel).

Si la ***UI*** ne dispose d'aucune connexion internet, elle doit être en mesure de continuer les traitements et de se synchroniser par la suite, dès lors que la connexion est revenue, et éventuellement informé l'utilisateur des commandes que l'[API](#api) aura rejeté.

------------------------------------------------------------------

###<a name="api">API</a>

> L'API est le coeur du réacteur, son rôle est de réceptionner des commandes, de les traiter ou de les rejeter.
> 
> Elle n'a aucune responsabilité sur la [UI](#ui), à part permettre la mise à jour de sa (ses) base(s) de données.
> 
> L'API est composé de l'[infrastructure technique](#infra) d'une part, et du [Domain Model](#domainModel) d'autre part

--------------------------------------------


###<a name="infra">Infrastructure technique</a>
> L'infrastructure permettant d'accueillir le code source du [Domain Model](#domainModel).
> 
> Elle est de l'autorité exclusive du Lead de l'infrastructure technique.


**L'infrastructure technique contient  :**

- le code source nécessaire à l'**acceptation de requêtes HTTP**(S) `POST`
	- quelqu'en soit leur provenance (application mobile, UI web, webservice...)
		- quelqu'en soit leur format (`xml`, `json`, `csv`...)
		- sur ce projet, nous fixerons toutefois le format de réception à `json`
- Le code source nécessaire pour **retourner un `HTTPResponseHeaders`**		
- Le code soure nécessaire pour **distribuer les messages en partance pour et émanant du [Domain Model](#domainModel)** :
	- dans la suite de ce document, cette partie du code source sera dénommée `MessageBus`
- Le code source de type **Helpers** :
	- ex. `Serializers`, `Validators`, `Tokenizers`, `ServiceContainers`, etc..
- Le code source nécessaire pour **authentifier les utilisateurs**
- le code source permettant de **persister les messages** émanant du [Domain Model](#domainModel)  :
	- dans une base de données, relationnelle ou pas
	- avec ou sans librairie d'ObjectMapping
- le code source permettant de **persiter des données dans la ou les base(s) de données UI**
- le code source permettant de **pré-autoriser les requêtes HTTP** :
	- dans le cadre de ce projet, ce code source se contentera la plupart du temps de rejeter les utilisateurs non-authentifiés
	- C'est en effet le [Domain Model](#domainModel) qui a autorité sur les autorisations, ces dernières faisant partie intégrale de la logique métier	


**L'infrastructure technique ne contient pas :**

- de code source traitant les requêtes HTTP(S) `GET`, `PUT`, `PATCH`, `DELETE`
- de code source retournant un `HTTPResponseBody`
- de code source contenant de la logique métier
- de code source refusant l'autorisation à une Command que le Domain Model aurait accepté.


**L'infrastructure technique peut avoir des dépendances vers des librairies techniques tierces :**


> `Serializers`, `Validators`, `Tokenizers`, `ServiceContainers`, `HTTPFoundation`, `MessageBus`, `PersistenceLibrairies` etc..

- il sera de la **responsabilité du Lead de l'infrastructure technique** de correctement gérer les dépendances et le versionning des librairies tierces qu'il aura décidé d'utiliser.
- il est  **recommandé au Lead de l'infrastructure technique** ne pas consommer les librairies tierces de manière concrète mais de mettre en place une couche d'abstraction (interface) entre la libraire tierce et le consumer.






-------------------------------------------


###<a name="domainModel">Domain & domain expert & domain model</a>

> Le **Domain** désigne l'ensemble des micro-process composant le fonctionnement interne de la société Alter&Go.
> 
> Le **Domain Model**  désigne la modélisation logicielle du *Domain*
> 
> Seul le **Domain Expert** a autorité sur le Domain
>
> Le Domain Model doit être `framework agnostic`


 

Le *Domain*, son contenu et son organisation, ne peuvent être de l'autorité exclusive du ***Domain Expert***, à savoir la société Alter&Go, cliente de ce projet.

Le ***Domain Model*** désigne la modélisation logicielle du *Domain*, et donc forcément une abstraction réduite du *Domain*.

Le *Domain Model* regroupe l'intégralité du code traitant la logique métier de l'application.

Le *Domain Model* doit pouvoir s'exécuter en parfaite indépendance de l'infrastructure technique et doit donc être `framework agnotic`. 

Par convention dans la suite du document, on considérera que le code source afférant au *Domain Model* sera placé dans un répertoire `./DomainModel/`.

Toujours par convention, nous considérerons que le repertoire `Domain Model` sera situé dans le répertoire `src/AppBundle/` d'un framework, mais il est de l'**autorité exclusive du [Lead de l'infrastructure technique](#infra) de définir la manière dont il intègre le *Domain Model* à son [infrastructure technique](#infra)**. 


____________________________________________


###<a name="BC">Bounded Context</a> 

> Bounded Context = Champs d'application, contexte dialectique d'un métier ou sous-métier dans lequel une solution logicielle va être apportée
> `src/AppBundle/DomainModel/BoundedContext [./SubContext [./SubSubContext]]`

Le [Domain Model](#domainModel) ne pouvant évidemment pas modelliser la globalité du [Domain](#domainModel), il s'organise en morceau, chaque morceau correspondant à un métier ou composant de métier, au sens organisationnel interne Alter&Go (RH, Gestion, Commercial, Opérations etc...), qu'il se propose de solutionner.

Ces "morceaux" sont appelés ***Bounded Context***

Les ***Bounded Context*** à traiter dans le cadre de ce projet **sont au nombre de 18** et organisés de la manière suivante :
    
    src/AppBundle/DomainModel/ <------ Alter&Go Groupe <-- niveau macro 
	|	
	|--	AlterEtGoGroup/
	|	|-------HumanRessources/  
	|	|		|---EmployeesCatalog/						<--BC1
	|	|		|---HRManagement/							<--BC2
	|	|		|---Leaves/									<--BC3
	|	|		|---Evaluations/							<--BC4
	|	|
	|	|------	Finances/
	|	|		|--	RefundableExpenseManagement/			<--BC5
	|	|		|--	Invoicing/
	|	|			|--	CustomerInvoicing/
	|	|			|	|--	PerformanceInvoicing/			<--BC6
	|	|			|	|--	RechargeableExpenseInvoicing/	<--BC7
	|	|			|--	InternalInvoicing/					<--BC8
	|	|
	|	|------	ActivityManagement/
	|	|		|--	ActivityReport/							<--BC9
	|	|			
	|	|------	Catalogs/
	|			|--	Customers/								<--BC10
	|			|--	Suppliers/								<--BC11
	|			|--	BusinessUnits/							<--BC12
	|			|--	Products
	|
    |--	OperationalBusinessUnit/
		|--	Operations/
			|--	RevenueManagement/
			|	|--	FeesCharging/							<--BC13
			|	|--	OutsourcedDeliveriesCharging/			<--BC14
			|	|--	ProductLeasing/							<--BC15
			|	
			|--	Catalogs/
				|--	ContractsCatalog/						<--BC16
				|--	RessourcesCatalog/						<--BC17
				|--	ProductsCatalog/						<--BC18

		
				
			

 

En définissant les bounded context et sub-context comme racine de l'arborescence du code du [domain model](#domainModel), le code est logiquement organisé selon la logique métier Alter&Go, et permet de mieux anticiper les évolutions organisationnelles futures (avec la création éventelle de sub-context).

*Le **[domain model](#domainModel)** est découpé en contextes métiers s'exprimant avec leur dialectique propre.*

*D'un context à l'autre, une même entité (ex. une personne physqiue) peut avoir un nom différent. Ainsi, dans le contexte RH, on parlera de Collaborateurs, alors que dns le contexte Opération, on parlera de Ressources*

-------------------------------------------------------


###<a name="ac">Autonomous components</a>  

> Autonomous components = Composants logiciels autonomes
> 
> Les Autonomous Components apportent une solution logicielle aux problématiques métier et organisationnelle identifiées dans un [bounded context](#BC).
> 
> La modification, la suppression ou le remplacement d'un Autonomous Component ne doit avoir aucun impact sur un autre Autonomous Component

Un Bounded Context peut donc être composé de un (souvent) ou plusieurs (rarement) Autonomous Components.

 	src/AppBundle/DomainModel/ <------ Alter&Go Groupe <-- niveau macro 
		
		AlterEtGoGroup/HumanRessources/  
					EmployeesCatalog/							<--BC1 
							EmployeesCatalogComponent/			<--AC1 <-- Autonomous Component 1

				
**Par autonome, on désigne le fait qu'un composant doit pouvoir être étendu, supprimé, remplacé par un ou plusieurs autre(s), sans que le code d'un autre composant ne soit affecté.**

	
Un Autonomous Component reçoit des messages  en provenance du `MessageBus` :

- soit par l'intermédiaire de ses [CommandHandlers](#commandHandlers) s'il s'agit de [commandes](#commands)
- soit par l'intermédiaire de ses [Process Managers](#sagas) s'il s'agit d'[évènements](#events), émis par un autre *Autonomous Component*, auquel il a décidé de souscrire.

Un Autonomous Component émet des message vers le `MessageBus`

Par convention, nous considérerons qu'un Autonomous components est organisé de la manière suivante :

    src/AppBundle/DomainModel/AlterEtGoGroup/HumanRessources/EmployeesCatalog/
		
		EmployeesCatalogComponent/
    		Aggregates/
				EmployeeAggregate/
					employeeAggregate.php
					Entities/
						employee.php
						...
					ValueObjects/
						Person.php
						...
					DomainEvents/
						employeeHired.php
						...
    		Commanding/
				CommandHandlers/
					hireEmployeeCommandHandler.php
					...
				Commands/
					hireEmployeeCommand.php
					...
    		ProcessManagers/
				processManager1.php
				...
   			Denormalizers/
				employeesListViewDenormalizer.php			<--- 
				employeeViewDenormalizer.php
				...

__________________________________

###<a name="commands">Commands</a>



> Commands = les ordres donnés au [Domain Model](#domainModel).*

>Exemple : `RejectLeaveRequestCommand` (Ordre de refuser une demande d'absence).

Les ordres pouvant émaner de n'importe quelle source (un web browser, une application mobile, une ligne de commande, un webservice...),  sous différents formats (`xml`, `json`, `txt`...), il est de l'autorité de l'[infrastructure technique](#infra) de prévoir les différents formateurs et serializer nécessaires pour que les ordres, lorsqu'ils arrivent au [Domain Model](#domainModel), soient totalement `format and source agnostic`.

Il est recommandé à l'infrastructure technique de prévoir ces opérations avant d'envoyer les ordres sur le `MessageBus`.





---
###<a name="commandHandlers">Command Handlers</a>

> Il existe un et un seul [CommandHandler](#commandHandlers) par [Command](#commands).
> 
> Il n'y a pas de [Command](#commands) sans [CommandHandler](#commandHandlers).

Le rôle des [CommandHandlers](#commandHandlers) est de :


-  vérifier que les [Commands](#commands) ne sont pas incomplètes ou malformée.
- Si elles le sont, ils les rejettent
- Sinon,
	- ils les adressent à l' [Aggregate](#ar) pour exécution.
	- ils récupèrent et transmettent le résultat de l'exécution de l' [Aggregate](#ar) vers le `MessageBus`.

Il est recommandé à l'infrastructure technique d'injecter le `MessageBus` dans les [CommandHandlers](#commandHandlers).

---
###<a name="ar">Aggregates & Aggregate Roots</a>



> **Toute la logique métier du [Domain Model](#domainModel) doit être traitée au sein des *Aggregates***
> 
> Un **Aggregate**  est un ensemble abstrait, une agglomération d'entités métier concrêtes (ex. un collaborateur, un client, une facture...) dépendant les une des autres pour accomplir une tâche métier spécifique.
> 
> La communication avec un *Aggregate* ne peut se faire que par le biais de son **Aggregate Root**
> 
> Un *Aggregate root* a l'autorité exclusive sur le comportement des entités de son *Aggregate*.
> 
> Un *Aggregate root* est garant du cycle de vie de son *Aggregate*
> 
> Un *Aggregate root* doit pouvoir être supprimé, remplacé, étendu sans qu'aucune modification de code ne soit nécessaire ailleurs que dans cet aggrégat

Un *Aggregate root* est une entité concrète dotée de l'exclusive autorité  sur le comportement des entités aggrégées dans l'*Aggregate*.

C'est aux *Aggregates* que les [CommandHandlers](#commandHandlers) adressent les [commandes](#commands) qu'ils ont reçu du `MessageBus`.

Lorsqu'un *Aggregate Root* accepte de traiter une [Command](#commands), il l'exécute puis émet un ou plusieurs [Domain Event ](#events) relatant l'opération qu'il vient d'exécuter.

Un *Aggregate Root* peut et doit refuser d'exécuter une [Command](#commands) si cette dernière n'est pas compatible avec le comportement et le cycle de vie attendu de son *Aggregate* :


- Par exemple, si `EmployeeAggregate` reçoit la commande `IncreaseSalaryCommand` alors qu'il est au statut `EmployeeFired`

- Dans ce cas, il doit se contenter d'émettre  une `\Exception`:


**Il est recommandé [au responsable de la UI](#ui) d'effectuer un maximum de vérification en amont pour que les commandes arrivant aux aggrégats aient une probabilité minimale d'être rejetée. Ce pré-requis devrait être facilité que les informations sont pushés sur la [UI](#ui)**





----


###<a name="events">Domain Events</a> 

> Evénements métier produit par les *Aggregates*

Exemple : `LeaveRequestRejected` (demande d'absence refusée).*

 une commande, lorsqu'elle est acceptée, génère un évènement (par le biais de l'aggrégat) qui :

- est propagé dans la [UI](#ui) par le biais de `Denormalizers`
- est capté par les agents de communication inter-aggrégats, les `Sagas` ou `Process managers`


    
----
###<a name="sagas">Process Managers (aka. Sagas)</a>

> Les [sagas](#sagas) permettent d'attendre qu'une certaine configuration d'[évènements](#events) se soit produite pour émettre à leur tour une [Command](#Command).

*to be completed...*

----
###<a name="denorm">Denormalizers</a>
> les [Denormalizers](#denorm) réagissent aux [évènements](#events) en créant un jeu de données à transmettre au [Read Model](#readModel) pour mise-à-jour.

*to be completed...*

---
###<a name="readModel">Read Model</a>
> le Read Model désigne la (les) base(s) de données auquel accède le(s) composant(s) de la UI pour afficher l'état du Domain Model. 

*to be completed...*
