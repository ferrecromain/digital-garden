---
title: Facade
---

# Facade

## Définition
Le patron de conception **façade** fournit une API simplifiée pour un ou plusieurs sous-systèmes.

Une façade permet de :
* De créer une nouvelle API facilement utilisable, compréhensible et testable.
* Réduire les dépendances entre le code extérieur et le ou les sous-systèmes.

## Exemple

Dans l'exemple ci-dessous :

* ```AccountRepository```, ```LoggerService```, ```EmailService``` sont les *sous-systèmes*
* ```AccountService``` est la *façade* que les clients utiliseront pour interagir avec les comptes utilisateurs.

```csharp
/// <summary>
/// Facade class
/// </summary>
public class AccountService
{
    ...
    public void RemoveAccount(int id)
    {
        Account account = _accountRepository.Get(id);
        if(account != null)
        {
            _accountRepository.Delete(id);
            _emailService.Send(Account, EmailTemplate.AccountRemoved);
        }
        _loggerService.Log(logLevel.Warn, $"Account id : {id} does not exists");
    }
  
    public CreateAccount(Account account)
    {
        _accountRepository.Create(account);
        _emailService.Send(Account, EmailTemplate.AccountCreated);
    }
    ...
}
```
