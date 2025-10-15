---
title: Singleton
---

# Singleton

## Définition

Le patron de conception **singleton** permet de s'assurer qu'**une classe ne puisse être instanciée qu'une seule fois**, et donne un accès simple à cette instance.

```csharp

public sealed class Singleton
{
    private static Singleton _instance;
  
    // Empecher l'utilisation du constructeur par défaut
    private Singleton() { }

    public static Singleton Instance
    {
        get
        {
            if (_instance == null)
            {
                _instance = new Singleton();
            }
            return _instance;
        }
    }
}
```

## Exemple

Vérifions à présent que cette classe ne peut délivrer qu'une seule instance d'elle même.  
Notez que l'utilisation de ```new Singleton()``` n'est plus possible.
  
```csharp
Singleton a = Singleton.Instance;
Singleton b = Singleton.Instance;
  
Console.WriteLine($"Les objets Singleton sont de la même instance ? : {a.Equals(b)}");
```

Après exécution, nous obtenons bien le résultat escompté :
  
```console
Les objets Singleton sont de la même instance ? : True
```
