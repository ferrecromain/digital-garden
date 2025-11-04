---
Title: Asynchronisme
---

# Asynchronisme

## Présentation

L’asynchronisme consiste à exécuter une tâche en arrière-plan tout en poursuivant le traitement principal, puis à récupérer le résultat une fois la tâche terminée.

Prenons comme analogie la préparation d’un déjeuner à la maison, où :
- Une **personne** représente un [```Thread```][Thread].
- Une **opération rouge** représente un groupe de [```Task```][Task].
- Une **opération verte** ne peut être exécutée qu'une fois les [```Task```][Task] nécessaires terminées.

### Exécution synchrone

La personne est sollicitée durant toute la préparation et attend devant la plaque chauffante pour chaque cuisson.

```mermaid
stateDiagram
    Direction LR
    [*] --> CookChicken
    CookChicken --> CookPasta
    CookPasta --> DressTable
    DressTable --> Serve
    Serve --> [*]

CookChicken: Cuire le poulet
CookPasta: Cuire les pâtes
DressTable: Dresser la table
Serve: Servir
```

### Exécution asynchrone

La personne programme les cuissons sur les plaques chauffantes, dresse la table, puis est libérée. Lorsque le minuteur signalera la fin des cuissons, elle ou une autre personne se chargera du service.

```mermaid
stateDiagram

    classDef asyncTask fill:#B00
    classDef await fill:#0B0,color:white

    Direction LR
    [*] --> Cooking:::asyncTask
    state Cooking {
        Direction LR
        CookChicken --> CookPasta
    }

    Cooking --> DressTable
    DressTable --> Serve:::await
    note left of DressTable : La personne est libérée à la fin
    note left of Serve : Une fois les cuissons terminées, une personne s'en occupera
    Serve --> [*]

Cooking: Cuisine
CookChicken: Cuire le poulet
CookPasta: Cuire les pâtes
DressTable: Dresser la table
Serve: Servir
```

> [!NOTE]
> Dans les bibliothèques .NET, la plupart des opérations asynchrones (requêtes TCP/IP, système de fichiers, compte à rebours...) sont déléguées à des composants du système d'exploitation (comme [IOCP] sous Windows). Un peu de la même manière qu'une *cuisson* est réalisée par une *plaque chauffante* qui est un composant d'une *cuisine*. 
> 
> Une fois complétées, elles envoient un signal au **runtime**, qui demande au [```ThreadPool```][ThreadPool] un [```Thread```][Thread] disponible afin de poursuivre l’exécution du programme.


## Implémentation

La version 5 de C# a introduit une approche simplifiée pour rendre des méthodes asynchrones : 
1. Sa **signature** doit comprendre le modificateur [```async```][async], indiquant qu'une méthode ou expression lambda est asynchrone.
2. Son **nom** doit comprendre le suffixe *Async*.
3. Le **type de retour** doit être au choix :
    - [```Task```][Task] : une opération asynchrone.
    - [```Task<TResult>```][Task<TResult>] : une opération asynchrone qui produit un résultat.
4. Elle doit comporter au moins une expression [```await```][await], qui 
   1. Attend le retour de la tâche de la méthode asynchrone associée.
   2. Rend la main au code appelant.

> [!NOTE]
> - Un [```Thread```][Thread] est un fil d’exécution d'instructions auquel le système d'exploitation alloue du temps processeur.
> - Le [```ThreadPool```][ThreadPool] est un ensemble de [```Thread```][Thread] gérés et réservés par le runtime .NET.
> - Une [```Task```][Task] (on parle aussi de [future]) s’exécute sur un [```Thread```][Thread] disponible dans le [```ThreadPool```][ThreadPool].

## Exemple

```csharp
using System.Diagnostics;

async Task TaskOneAsync()
{
    WriteLine(nameof(TaskOneAsync), "started");
    await Task.Delay(1000);
    WriteLine(nameof(TaskOneAsync), "ended");
}

async Task TaskTwoAsync()
{
    WriteLine(nameof(TaskTwoAsync), "started");
    await Task.Delay(2000);
    WriteLine(nameof(TaskTwoAsync), "ended");
}

void WriteLine(string name, string state)
{
    Console.WriteLine($"{name} {state} on thread {Environment.CurrentManagedThreadId}");
}

WriteLine("Main", "started");
Stopwatch sw = new();
sw.Start();

Task operationOne = TaskOneAsync();
Task operationTwo = TaskTwoAsync();

// await Task.WaitAll(operationOne, operationTwo)
await operationOne;
await operationTwo;

sw.Stop();
WriteLine("Main", $"ended in {sw.ElapsedMilliseconds} ms");
```

```console
Main started on thread 1
TaskOneAsync started on thread 1
TaskTwoAsync started on thread 1
TaskOneAsync ended on thread 5
TaskTwoAsync ended on thread 5
Main ended in 2023ms on thread 5
```

```mermaid
sequenceDiagram
    autonumber
    participant Main
    participant TaskOne
    participant TaskTwo
    participant Timer

    Main->>TaskOne: start TaskOneAsync()
    TaskOne->>Timer: schedule delay(1000)
    TaskOne-->>Main: await (returns to Main)

    Main->>TaskTwo: start TaskTwoAsync()
    TaskTwo->>Timer: schedule delay(2000)
    TaskTwo-->>Main: await (returns to Main)

    Main->>Main: await operationOne
    Note over Main: Main suspended, thread released

    Timer-->>TaskOne: delay(1000) expired → resume continuation

    TaskOne-->>Main: TaskOne completed
    Main->>Main: await operationTwo
    Note over Main: Main remains suspended, waiting for TaskTwo

    Timer-->>TaskTwo: delay(2000) expired → resume continuation

    TaskTwo-->>Main: TaskTwo completed

    Main->>Main: print "All operations finished"
    Note over Main: Main resumed on a Threadpool thread.
```

[Task]: https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task
[Task<TResult>]: https://learn.microsoft.com/en-us/dotnet/api/system.threading.tasks.task-1
[ThreadPool]: https://learn.microsoft.com/en-us/dotnet/api/system.threading.threadpool
[Thread]: https://learn.microsoft.com/en-us/dotnet/api/system.threading.thread
[future]: https://fr.wikipedia.org/wiki/Futures_(informatique)
[async]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/async
[await]: https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/await
[IOCP]: https://learn.microsoft.com/en-us/windows/win32/fileio/i-o-completion-ports