---
layout: post
categories: php soap csharp
date: 2011-06-26 11:00:00 +0200
lang: fr
ref: 2011-06-26-consommer-webservice-auth-basic-csharp
title: "Consommer WebService Auth Basic avec C#"
---

J'ai du valider différents clients pour se connecter à un webservice (SOAP 1.2) réaliser en PHP avec une authentification basique.
Mais il semble que DotNet n'est pas prévu d'utiliser cette méthode d'authentification pour se connecter.
Donc on ouvre VisualStudio et on générer une référence de service web....

On ouvre le fichier contenant le code de la référence web, un fichier Reference.cs. Et on ajoute cette méthode :
```
protected override System.Net.WebRequest GetWebRequest(Uri uri)
{
  request = (HttpWebRequest)base.GetWebRequest(uri);
  HttpWebRequest request;

  if (PreAuthenticate) {
    NetworkCredential networkCredentials = Credentials.GetCredential(uri, "Basic");
    if (networkCredentials != null) {
      byte[] credentialBuffer = new UTF8Encoding().GetBytes(networkCredentials.UserName + ":" + networkCredentials.Password);
      request.Headers["Authorization"] = "Basic " + Convert.ToBase64String(credentialBuffer);
    } else {
      throw new ApplicationException(“No network credentials”);
    }
  }

  return request;
}
```

On surcharge donc la méthode GetWebRequest() de la classe System.Web.Services.Protocols.SoapHttpClientProtocol.
Le problème de cette méthode est que à chaque mise à jour de la référence, le code ajouté est donc supprimer. Il existe certainement une méthode pour faire autrement mais je ne suis pas un expert en C#.

Ensuite pour utiliser le code, cela reste très simple.
```
My.Web.Service service = new My.Web.Service();
NetworkCredential netCredential = new NetworkCredential("LOGIN", "PASSWORD");
Uri uri = new Uri(service.Url);
ICredentials credentials = netCredential.GetCredential(uri, "Basic");
service.Credentials = credentials;
service.PreAuthenticate = true;
service.Method();
```

Et maintenant avec un logiciel tel que wireshark ou un analyser (Fiddler) de requette HTTP, vous devriez avoir une ligne dans les headers HTTP, du genre :

`Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==`

Pour référence, voir ici : http://en.wikipedia.org/wiki/Basic_access_authentication
