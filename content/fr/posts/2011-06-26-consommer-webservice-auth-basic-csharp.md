---
translationKey: "2011-06-26-consommer-webservice-auth-basic-csharp"
categories: ["csharp", "soap", "dotnet"]
date: "2011-06-26T11:00:00Z"
title: "Consommer un Web Service SOAP avec Authentification Basique en C#"
---

Interagir avec des services web SOAP, par exemple ceux développés en PHP, depuis une application .NET est une tâche courante. Lorsque ces services sont protégés par une **authentification HTTP basique (Basic Auth)**, la configuration du client C# nécessite une petite étape supplémentaire, mais elle est plus simple qu'il n'y paraît.

Ce guide montre la méthode standard et recommandée pour consommer un web service SOAP sécurisé avec Basic Auth en utilisant C#.

## Le défi : l'authentification basique

Lors de l'ajout d'une "Référence de service" à un projet Visual Studio pour un web service SOAP, le client généré ne gère pas automatiquement l'envoi des informations d'authentification. Si vous essayez d'appeler une méthode du service directement, vous obtiendrez probablement une erreur `(401) Unauthorized`.

La solution consiste à fournir explicitement les informations d'identification (login et mot de passe) à l'instance du client de service.

## La méthode standard et recommandée

La classe de base pour les clients de service générés (`System.Web.Services.Protocols.SoapHttpClientProtocol`) possède une propriété `Credentials` conçue exactement pour ce scénario.

Voici comment l'utiliser :

```csharp
// 1. Instanciez le client de service généré par Visual Studio
var service = new My.Web.Service();

// 2. Créez un objet NetworkCredential avec votre login et mot de passe
var credentials = new System.Net.NetworkCredential("VOTRE_LOGIN", "VOTRE_MOT_DE_PASSE");

// 3. Assignez cet objet à la propriété Credentials du service
// C'est cette étape qui indique à .NET d'utiliser l'authentification basique
service.Credentials = credentials;

// 4. (Optionnel mais recommandé) Activez la pré-authentification
// Cela envoie l'en-tête "Authorization" dès la première requête,
// évitant ainsi un aller-retour inutile (challenge 401 puis requête authentifiée).
service.PreAuthenticate = true;

// 5. Appelez votre méthode de service web
// La requête HTTP inclura désormais l'en-tête d'authentification requis.
var result = service.Method();
```

Cette approche est propre, simple et ne nécessite aucune modification du code généré (`Reference.cs`). Elle est donc robuste aux mises à jour de la référence de service.

## Ancienne approche : Surcharger `GetWebRequest` (Non recommandé)

Une autre méthode, parfois rencontrée dans d'anciens projets, consiste à modifier directement le fichier `Reference.cs` pour surcharger la méthode `GetWebRequest`. **Cette approche n'est pas recommandée** car toute mise à jour de la référence de service écrasera vos modifications.

Elle consistait à ajouter manuellement l'en-tête `Authorization`. Si vous la rencontrez, voici à quoi elle ressemble (avec les corrections nécessaires) :

```csharp
// Dans le fichier Reference.cs, à l'intérieur de la classe du client de service...
protected override System.Net.WebRequest GetWebRequest(Uri uri)
{
    // Récupère la requête web de base
    HttpWebRequest request = (HttpWebRequest)base.GetWebRequest(uri);

    // Si la pré-authentification est activée, on construit l'en-tête manuellement
    if (this.PreAuthenticate)
    {
        NetworkCredential nc = this.Credentials.GetCredential(uri, "Basic");
        if (nc != null)
        {
            byte[] credentialBuffer = new System.Text.UTF8Encoding().GetBytes(nc.UserName + ":" + nc.Password);
            request.Headers["Authorization"] = "Basic " + Convert.ToBase64String(credentialBuffer);
        }
    }

    return request;
}
```
Cette méthode est plus complexe et fragile que l'utilisation directe de la propriété `Credentials`.

## Vérification de la requête

Pour confirmer que l'authentification fonctionne, vous pouvez utiliser un outil d'analyse de trafic HTTP comme [Fiddler](https://www.telerik.com/fiddler) ou Wireshark. En inspectant la requête sortante, vous devriez voir un en-tête HTTP qui ressemble à ceci :

```http
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```

La chaîne de caractères après `Basic` est votre `login:motdepasse` encodé en Base64.

Pour en savoir plus sur le fonctionnement de cette authentification, consultez la [page Wikipédia sur l'authentification HTTP basique](http://fr.wikipedia.org/wiki/Authentification_HTTP#Authentification_basique).