---
title: Authentifizierung über Active Directory und Resource Manager| Microsoft Docs
description: Entwicklerhandbuch für die Authentifizierung mit der Azure Resource Manager-API und Active Directory für die Integration einer App in anderen Azure-Abonnements
services: azure-resource-manager,active-directory
documentationcenter: na
author: dushyantgill
manager: timlt
editor: tysonn

ms.service: azure-resource-manager
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 08/31/2016
ms.author: dugill;tomfitz

---
# Gewusst wie: Verwenden von Azure Active Directory und Resource Manager zum Verwalten der Ressourcen eines Kunden
## Einführung
Wenn Sie Softwareentwickler sind, der eine App erstellen möchten, die die Azure-Ressourcen eines Kunden verwaltet, finden Sie in diesem Thema Informationen zur Authentifizierung mit den Azure Resource Manager-APIs und zum Zugriff auf Ressourcen in anderen Abonnements.

Ihre App kann auf die Resource Manager-APIs auf verschiedene Arten zugreifen:

1. **Benutzer- und App-Zugriff**: Für Apps, die für einen angemeldeten Benutzer auf Ressourcen zugreifen. Diese Methode funktioniert für Apps wie Web-Apps und Befehlszeilentools, die nur für die „interaktive Verwaltung“ von Azure-Ressourcen verwendet werden.
2. **Nur App-Zugriff**: Für Apps, die Daemondienste und geplante Aufträge ausführen. Der App-Identität wird direkter Zugriff auf die Ressourcen gewährt. Diese Methode funktioniert für Apps, einen langfristigen „Offlinezugriff“ auf Azure benötigen.

Dieses Thema enthält eine schrittweise Anleitung zum Erstellen einer App, in der beide Autorisierungsmethoden genutzt werden. Es beschreibt, wie jeder Schritt mit REST-API oder C# ausgeführt wird. Die vollständige ASP.NET MVC-Anwendung finden Sie unter [https://github.com/dushyantgill/VipSwapper/tree/master/CloudSense](https://github.com/dushyantgill/VipSwapper/tree/master/CloudSense).

Der gesamte Code für dieses Thema wird als Web-App ausgeführt, die Sie unter [http://vipswapper.azurewebsites.net/cloudsense](http://vipswapper.azurewebsites.net/cloudsense) testen können.

## Funktionen der Web-App
Die Web-App führt folgende Aktionen aus:

1. Einen Azure-Benutzer anmelden
2. Benutzer auffordern, der Web-App Zugriff auf Resource Manager zu gewähren
3. Zugriffstoken für den Benutzer und die App für den Zugriff auf Resource Manager abrufen
4. Token verwenden (aus Schritt 3), um Resource Manager aufzurufen und den Dienstprinzipal der App einer Rolle im Abonnement zuzuweisen, wodurch die App langfristigen Zugriff auf das Abonnement erhält
5. Nur für die App geltende Zugriffstoken abrufen
6. Token verwenden (aus Schritt 5), um Ressourcen im Abonnement über Resource Manager zu verwalten

Im Folgenden ist der komplette Datenfluss in der Webanwendung dargestellt.

![Resource Manager-Authentifizierungsvorgang](./media/resource-manager-api-authentication/Auth-Swim-Lane.png)

Als Benutzer geben Sie die Abonnement-ID für das zu verwendende Abonnement an:

![Abonnement-ID angeben](./media/resource-manager-api-authentication/sample-ux-1.png)

Wählen Sie das Konto aus, das für die Anmeldung verwendet werden soll.

![Konto auswählen](./media/resource-manager-api-authentication/sample-ux-2.png)

Geben Sie Ihre Anmeldeinformationen ein.

![Anmeldeinformationen angeben](./media/resource-manager-api-authentication/sample-ux-3.png)

Gewähren Sie der App Zugriff auf Ihre Azure-Abonnements:

![Gewähren von Zugriff](./media/resource-manager-api-authentication/sample-ux-4.png)

Verwalten Sie die verbundenen Abonnements:

![Abonnement verbinden](./media/resource-manager-api-authentication/sample-ux-7.png)

## Registrieren einer Anwendung
Bevor Sie mit dem Programmieren beginnen, müssen Sie Ihre Web-App bei Azure Active Directory (AD) registrieren. Durch die App-Registrierung wird eine zentrale Identität für Ihre App in Azure AD erstellt. Sie enthält grundlegende Informationen über Ihre Anwendung wie die OAuth-Client-ID, Antwort-URLs und Anmeldeinformationen, die Ihre Anwendung zur Authentifizierung von Azure Resource Manager-APIs und zum Zugriff darauf verwendet. Die App-Registrierung zeichnet auch die verschiedenen delegierten Berechtigungen auf, die Ihre Anwendung benötigt, um für den Benutzer auf Microsoft-APIs zuzugreifen.

Da Ihre App auf andere Abonnements zugreift, müssen Sie sie als mehrinstanzenfähige Anwendung konfigurieren. Geben Sie eine Domäne an, die Ihrem Active Directory zugeordnet ist, damit die Überprüfung erfolgreich ausgeführt werden kann. Melden Sie sich beim [klassischen Portal](https://manage.windowsazure.com) an, um die Domänen zu sehen, die Ihrem Active Directory zugeordnet sind. Wählen Sie Ihr Active Directory aus, und wählen Sie anschließend **Domänen**.

Im folgenden Beispiel wird gezeigt, wie Sie die App mithilfe von Azure PowerShell registrieren. Damit dieser Befehl funktioniert, benötigen Sie die aktuelle Version von Azure PowerShell (August 2016).

    $app = New-AzureRmADApplication -DisplayName "{app name}" -HomePage "https://{your domain}/{app name}" -IdentifierUris "https://{your domain}/{app name}" -Password "{your password}" -AvailableToOtherTenants $true

Um sich als AD-Anwendung anzumelden, benötigen Sie die Anwendungs-ID und das Kennwort. Damit die Anwendungs-ID angezeigt wird, die vom vorherigen Befehl ausgegeben wird, verwenden Sie Folgendes:

    $app.ApplicationId

Im folgenden Beispiel wird gezeigt, wie Sie die App mithilfe der Azure-CLI registrieren.

    azure ad app create --name {app name} --home-page https://{your domain}/{app name} --identifier-uris https://{your domain}/{app name} --password {your password} --available true

Die Ergebnisse enthalten die AppID, die Sie beim Authentifizieren als Anwendung benötigen.

### Optionale Konfiguration – Zertifikatanmeldeinformationen
Azure AD unterstützt außerdem Zertifikatanmeldeinformationen für Anwendungen: Sie erstellen ein selbstsigniertes Zertifikat, behalten den privaten Schlüssel und fügen den öffentlichen Schlüssel der Azure AD-Anwendungsregistrierung hinzu. Zur Authentifizierung sendet Ihre Anwendung eine kleine Nutzlast an Azure AD, die mit Ihrem privaten Schlüssel signiert ist, und Azure AD überprüft die Signatur mithilfe des öffentlichen Schlüssels, den Sie registriert haben.

Informationen zum Erstellen einer AD-App mit einem Zertifikat finden Sie unter [Erstellen eines Dienstprinzipals für den Zugriff auf Ressourcen mithilfe von Azure PowerShell](resource-group-authenticate-service-principal.md#create-service-principal-with-certificate) oder unter [Erstellen eines Dienstprinzipals für den Zugriff auf Ressourcen mithilfe der Azure-CLI](resource-group-authenticate-service-principal-cli.md#create-service-principal-with-certificate).

## Abrufen der Mandanten-ID ausgehend von der Abonnement-ID
Um ein Token anzufordern, das zum Aufrufen von Resource Manager verwendet werden kann, benötigt Ihre Anwendung die Mandanten-ID des Azure AD-Mandanten, der das Azure-Abonnement hostet. In den meisten Fällen kennen Benutzer Ihre Abonnement-ID, unter Umständen kennen sie ihre Mandanten-ID für Active Directory jedoch nicht. Fragen Sie den Benutzer nach der Abonnement-ID, um die Mandanten-ID abrufen zu können. Geben Sie die Abonnement-ID beim Senden einer Anforderung zum Abonnement an:

    https://management.azure.com/subscriptions/{subscription-id}?api-version=2015-01-01

Die Anforderung schlägt fehl, da der Benutzer nicht noch angemeldet ist, aus der Antwort können Sie jedoch die Mandanten-ID abrufen. Rufen Sie in dieser Ausnahme die Mandanten-ID aus dem Wert des Antwortheaders für **WWW-Authenticate** ab. Diese Implementierung sehen Sie in der Methode [GetDirectoryForSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L20).

## Zugriffstoken für den Benutzer und die App abrufen
Die Anwendung leitet den Benutzer mit einer OAuth 2.0-Autorisierungsanforderung zu Azure AD weiter, um die Anmeldeinformationen des Benutzers zu authentifizieren und einen Autorisierungscode zu erhalten. Die Anwendung verwendet den Autorisierungscode, um ein Zugriffstoken für Resource Manager zu erhalten. Die Methode [ConnectSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/Controllers/HomeController.cs#L42) erstellt die Autorisierungsanforderung.

Dieses Thema beschreibt die REST-API-Anforderungen zum Authentifizieren des Benutzers. Sie können auch Hilfsbibliotheken verwenden, um die Authentifizierung in Ihrem Code durchzuführen. Weitere Informationen zu diesen Bibliotheken finden Sie unter [Azure Active Directory-Authentifizierungsbibliotheken](active-directory/active-directory-authentication-libraries.md). Informationen zum Integrieren des Identitäts-Managements in einer Anwendung finden Sie im [Entwicklerhandbuch zu Azure Active Directory](active-directory/active-directory-developers-guide.md).

### Autorisierungsanforderung (OAuth 2.0)
Geben Sie eine Open ID Connect/OAuth2.0-Autorisierungsanforderung an den Azure AD-Autorisierungsendpunkt aus:

    https://login.microsoftonline.com/{tenant-id}/OAuth2/Authorize

Die Abfragezeichenfolgen-Parameter, die für diese Anforderung verfügbar sind, werden im Thema [Anfordern eines Autorisierungscodes](active-directory/active-directory-protocols-oauth-code.md#request-an-authorization-code) beschrieben.

Das folgende Beispiel zeigt, wie Sie eine OAuth2.0-Autorisierung anfordern:

    https://login.microsoftonline.com/{tenant-id}/OAuth2/Authorize?client_id=a0448380-c346-4f9f-b897-c18733de9394&response_mode=query&response_type=code&redirect_uri=http%3a%2f%2fwww.vipswapper.com%2fcloudsense%2fAccount%2fSignIn&resource=https%3a%2f%2fgraph.windows.net%2f&domain_hint=live.com

Azure AD authentifiziert den Benutzer und fordert ihn ggf. auf, der App eine Berechtigung zu erteilen. Dann wird der Autorisierungscode an die Antwort-URL der Anwendung zurückgegeben. Abhängig vom angeforderten Wert für „response\_mode“ sendet Azure AD die Daten in einer Abfragezeichenfolge oder als POST-Daten zurück.

    code=AAABAAAAiL****FDMZBUwZ8eCAA&session_state=2d16bbce-d5d1-443f-acdf-75f6b0ce8850

### Autorisierungsanforderung (Open ID Connect)
Wenn Sie nicht nur für den Benutzer auf Azure Resource Manager zugreifen möchten, sondern es dem Benutzer auch ermöglichen möchten, sich bei Ihrer Anwendung mit einem Azure AD-Konto anzumelden, geben Sie eine Open ID Connect-Autorisierungsanforderung aus. Mit Open ID Connect empfängt die Anwendung auch ein „id\_token“ von Azure AD, das Ihre App zum Anmelden des Benutzers verwenden kann.

Die Abfragezeichenfolgen-Parameter, die für diese Anforderung verfügbar sind, werden im Thema [Senden der Anmeldeanforderung](active-directory/active-directory-protocols-openid-connect-code.md#send-the-sign-in-request) beschrieben.

Beispiel für eine Open ID Connect-Anforderung:

     https://login.microsoftonline.com/{tenant-id}/OAuth2/Authorize?client_id=a0448380-c346-4f9f-b897-c18733de9394&response_mode=form_post&response_type=code+id_token&redirect_uri=http%3a%2f%2fwww.vipswapper.com%2fcloudsense%2fAccount%2fSignIn&resource=https%3a%2f%2fgraph.windows.net%2f&scope=openid+profile&nonce=63567Dc4MDAw&domain_hint=live.com&state=M_12tMyKaM8

Azure AD authentifiziert den Benutzer und fordert ihn ggf. auf, der App eine Berechtigung zu erteilen. Dann wird der Autorisierungscode an die Antwort-URL der Anwendung zurückgegeben. Abhängig vom angeforderten Wert für „response\_mode“ sendet Azure AD die Daten in einer Abfragezeichenfolge oder als POST-Daten zurück.

Beispiel für eine Open ID Connect-Antwort:

    code=AAABAAAAiL*****I4rDWd7zXsH6WUjlkIEQxIAA&id_token=eyJ0eXAiOiJKV1Q*****T3GrzzSFxg&state=M_12tMyKaM8&session_state=2d16bbce-d5d1-443f-acdf-75f6b0ce8850

### Tokenanforderung (Datenfluss für OAuth2.0-Codeberechtigungen)
Nachdem die Anwendung den Autorisierungscode von Azure AD erhalten hat, ist es an der Zeit, das Zugriffstoken für Azure Resource Manager abzurufen. Senden Sie eine Anforderung für ein OAuth2.0-Codeberechtigungstoken an den Azure AD-Tokenendpunkt:

    https://login.microsoftonline.com/{tenant-id}/OAuth2/Token

Die Abfragezeichenfolgen-Parameter, die für diese Anforderung verfügbar sind, werden im Thema [Fordern Sie ein Zugriffstoken mithilfe des Autorisierungscodes an.](active-directory/active-directory-protocols-oauth-code.md#use-the-authorization-code-to-request-an-access-token) beschrieben.

Das folgende Beispiel zeigt eine Anforderung für ein Codeberechtigungstoken mit Kennwort:

    POST https://login.microsoftonline.com/7fe877e6-a150-4992-bbfe-f517e304dfa0/oauth2/token HTTP/1.1

    Content-Type: application/x-www-form-urlencoded
    Content-Length: 1012

    grant_type=authorization_code&code=AAABAAAAiL9Kn2Z*****L1nVMH3Z5ESiAA&redirect_uri=http%3A%2F%2Flocalhost%3A62080%2FAccount%2FSignIn&client_id=a0448380-c346-4f9f-b897-c18733de9394&client_secret=olna84E8*****goScOg%3D

Wenn Sie Zertifikatanmeldeinformationen verwenden, erstellen Sie ein JSON-Webtoken (JWT), und signieren Sie (RSA-SHA256) mit dem privaten Schlüssel der Zertifikatanmeldeinformationen Ihrer Anwendung. Die Anspruchstypen für das Token finden Sie unter [JWT-Tokenansprüche](active-directory/active-directory-protocols-oauth-code.md#jwt-token-claims). Informationen zum Signieren von JWT-Token für die Clientassertion finden Sie im [Code der Active Directory Authentication Library (.NET)](https://github.com/AzureAD/azure-activedirectory-library-for-dotnet/blob/dev/src/ADAL.PCL.Desktop/CryptographyHelper.cs).

Informationen zur Clientauthentifizierung finden Sie in der [Open ID Connect-Spezifikation](http://openid.net/specs/openid-connect-core-1_0.html#ClientAuthentication).

Das folgende Beispiel zeigt eine Anforderung für ein Codeberechtigungstoken mit Zertifikatanmeldeinformationen:

    POST https://login.microsoftonline.com/7fe877e6-a150-4992-bbfe-f517e304dfa0/oauth2/token HTTP/1.1

    Content-Type: application/x-www-form-urlencoded
    Content-Length: 1012

    grant_type=authorization_code&code=AAABAAAAiL9Kn2Z*****L1nVMH3Z5ESiAA&redirect_uri=http%3A%2F%2Flocalhost%3A62080%2FAccount%2FSignIn&client_id=a0448380-c346-4f9f-b897-c18733de9394&client_assertion_type=urn%3Aietf%3Aparams%3Aoauth%3Aclient-assertion-type%3Ajwt-bearer&client_assertion=eyJhbG*****Y9cYo8nEjMyA

Eine Beispielantwort für ein Codeberechtigungstoken:

    HTTP/1.1 200 OK

    {"token_type":"Bearer","expires_in":"3599","expires_on":"1432039858","not_before":"1432035958","resource":"https://management.core.windows.net/","access_token":"eyJ0eXAiOiJKV1Q****M7Cw6JWtfY2lGc5A","refresh_token":"AAABAAAAiL9Kn2Z****55j-sjnyYgAA","scope":"user_impersonation","id_token":"eyJ0eXAiOiJKV*****-drP1J3P-HnHi9Rr46kGZnukEBH4dsg"}

#### Verarbeiten der Antwort für ein Codeberechtigungstoken
Eine erfolgreiche Antwort enthält das Zugriffstoken (Benutzer und App) für Azure Resource Manager. Ihre Anwendung verwendet dieses Zugriffstoken, um für den Benutzer auf Resource Manager zuzugreifen. Die Lebensdauer des von Azure AD ausgestellten Zugriffstokens beträgt eine Stunde. Es ist unwahrscheinlich, dass Ihre Webanwendung das Zugriffstoken (für den Benutzer und die App) erneuern muss. Wenn das Zugriffstoken erneuert werden muss, verwenden Sie das Aktualisierungstoken, dass die Anwendung in der Tokenantwort erhält. Senden Sie eine Anforderung für ein OAuth2.0-Token an den Azure AD-Tokenendpunkt:

    https://login.microsoftonline.com/{tenant-id}/OAuth2/Token

Die Parameter für die Aktualisierungsanforderung sind unter [Aktualisieren der Zugriffstoken](active-directory/active-directory-protocols-oauth-code.md#refreshing-the-access-tokens) beschrieben.

Das folgende Beispiel zeigt die Verwendung des Aktualisierungstokens:

    POST https://login.microsoftonline.com/7fe877e6-a150-4992-bbfe-f517e304dfa0/oauth2/token HTTP/1.1

    Content-Type: application/x-www-form-urlencoded
    Content-Length: 1012

    grant_type=refresh_token&refresh_token=AAABAAAAiL9Kn2Z****55j-sjnyYgAA&client_id=a0448380-c346-4f9f-b897-c18733de9394&client_secret=olna84E8*****goScOg%3D

Mit Aktualisierungstoken können Sie zwar neue Zugriffstoken für Azure Resource Manager abrufen, sie sind aber für Offlinezugriff durch Ihre Anwendung nicht geeignet. Die Lebensdauer der Aktualisierungstoken ist begrenzt, und Aktualisierungstoken sind an den Benutzer gebunden. Wenn der Benutzer die Organisation verlässt, hat die Anwendung, die das Aktualisierungstoken nutzt, keinen Zugriff mehr. Dieser Ansatz ist nicht für Anwendungen geeignet, die von Teams zum Verwalten ihrer Azure-Ressourcen verwendet werden.

## Prüfen Sie, ob der Benutzer Zugriff auf Abonnement gewähren kann.
Ihre Anwendung verfügt jetzt über ein Token, um für den Benutzer auf Azure Resource Manager zuzugreifen. Der nächste Schritt besteht darin, Ihre App mit dem Abonnement zu verbinden. Wenn die Verbindung hergestellt ist, kann die App die Abonnements verwalten, auch wenn der Benutzer nicht anwesend ist (langfristiger Offlinezugriff).

Sie rufen für jedes Abonnement, mit dem eine Verbindung hergestellt werden soll, die [Resource Manager-API zum Auflisten von Berechtigungen](https://msdn.microsoft.com/library/azure/dn906889.aspx) auf, um zu bestimmen, ob der Benutzer über Zugriffsverwaltungsrechte für das Abonnement verfügt.

Die Methode [UserCanManagerAccessForSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L44) der ASP.NET MVC-Beispiel-App implementiert diesen Aufruf.

Im folgenden Beispiel wird veranschaulicht, wie die Berechtigungen eines Benutzers für ein Abonnement angefordert werden. „83cfe939-2402-4581-b761-4f59b0a041e4“ ist die ID des Abonnements.

    GET https://management.azure.com/subscriptions/83cfe939-2402-4581-b761-4f59b0a041e4/providers/microsoft.authorization/permissions?api-version=2015-07-01 HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV1QiLC***lwO1mM7Cw6JWtfY2lGc5A

Beispiel für die Antwort beim Abrufen der Berechtigungen eines Benutzers für ein Abonnement:

    HTTP/1.1 200 OK

    {"value":[{"actions":["*"],"notActions":["Microsoft.Authorization/*/Write","Microsoft.Authorization/*/Delete"]},{"actions":["*/read"],"notActions":[]}]}

Die API für Berechtigungen gibt mehrere Berechtigungen zurück. Jede Berechtigung besteht aus zulässigen Aktionen (actions) und nicht zulässigen Aktionen (notactions). Wenn eine Aktion in der Liste der zulässigen Aktionen für eine Berechtigung vorhanden und in der Liste „notactions“ für diese Berechtigung nicht vorhanden ist, darf der Benutzer diese Aktion durchführen. **microsoft.authorization/roleassignments/write** ist die Aktion, mit der Zugriffsverwaltungsrechte gewährt werden. Ihre Anwendung muss das Ergebnis für die Berechtigungen analysieren und nach einem übereinstimmenden regulären Ausdruck für diese Aktionszeichenfolge in den zulässigen und nicht zulässigen Aktionen der einzelnen Berechtigungen suchen.

## Abrufen von Zugriffstoken, die nur für die App gültig sind
Sie wissen nun, ob der Benutzer Zugriff auf das Azure-Abonnement gewähren kann. Die nächsten Schritte sind:

1. Der Anwendungsidentität die entsprechende RBAC-Rolle für das Abonnement zuweisen
2. Die Zugriffszuweisung überprüfen, indem die Anwendungsberechtigung für das Abonnement abgefragt wird oder indem mit dem nur für die App geltenden Token auf Resource Manager zugegriffen wird
3. Die Verbindung in der Datenstruktur für „verbundene Abonnements“ Ihrer Anwendung aufzeichnen, sodass die ID des Abonnements beibehalten wird

Sehen wir uns den ersten Schritt näher an. Um der Anwendungsidentität die entsprechende RBAC-Rolle zuzuweisen, müssen Sie Folgendes bestimmen:

* Die Objekt-ID Ihrer Anwendungsidentität im Azure Active Directory-Verzeichnis des Benutzers
* Den Bezeichner der RBAC-Rolle, die Ihre Anwendung für das Abonnement benötigt

Wenn Ihre Anwendung einen Benutzer aus Azure AD authentifiziert, wird ein Dienstprinzipalobjekt für die Anwendung in Azure AD erstellt. Azure ermöglicht es, dass RBAC-Rollen Dienstprinzipalen zugewiesen werden, um direkten Zugriff auf entsprechende Anwendungen auf Azure-Ressourcen zu gewähren. Dies ist genau das, was wir tun möchten. Fragen Sie die Azure AD Graph-API ab, um den Bezeichner des Dienstprinzipals der Anwendung im Azure AD-Verzeichnis des angemeldeten Benutzers zu bestimmen.

Sie haben nur ein Zugriffstoken für Azure Resource Manager. Sie benötigen ein neues Zugriffstoken zum Aufrufen der Azure AD Graph-API. Jede Anwendung in Azure AD verfügt über Berechtigungen, um das eigene Dienstprinzipalobjekt abzufragen. Ein nur für die App geltendes Zugriffstoken ist daher ausreichend.

<a id="app-azure-ad-graph">

### Abrufen eines nur für die App geltenden Zugriffstokens für die Azure AD Graph-API
Geben Sie zum Authentifizieren Ihrer App und zum Abrufen eines Tokens für die Azure AD Graph-API im OAuth2.0-Datenfluss eine Tokenanforderung zur Gewährung von Clientanmeldeinformationen an den Azure AD-Tokenendpunkt aus (**https://login.microsoftonline.com/{directory\_domain\_name}/OAuth2/Token**).

Die Methode [GetObjectIdOfServicePrincipalInOrganization](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureADGraphAPIUtil.cs) der ASP.NET MVC-Beispielanwendung ruft unter Verwendung der Active Directory Authentication Library für .NET ein nur für die App geltendes Zugriffstoken für die Graph-API ab.

Die Abfragezeichenfolgen-Parameter, die für diese Anforderung verfügbar sind, werden im Thema [Anfordern eines Zugriffstokens](active-directory/active-directory-protocols-oauth-service-to-service.md#request-an-access-token) beschrieben.

Beispielanforderung für ein Token zur Gewährung von Clientanmeldeinformationen:

    POST https://login.microsoftonline.com/62e173e9-301e-423e-bcd4-29121ec1aa24/oauth2/token HTTP/1.1
    Content-Type: application/x-www-form-urlencoded
    Content-Length: 187</pre>
    <pre>grant_type=client_credentials&client_id=a0448380-c346-4f9f-b897-c18733de9394&resource=https%3A%2F%2Fgraph.windows.net%2F &client_secret=olna8C*****Og%3D

Beispielantwort für ein Token zur Gewährung von Clientanmeldeinformationen:

    HTTP/1.1 200 OK

    {"token_type":"Bearer","expires_in":"3599","expires_on":"1432039862","not_before":"1432035962","resource":"https://graph.windows.net/","access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOiJSUzI1NiIsIng1dCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSIsImtpZCI6Ik1uQ19WWmNBVGZNNXBPWWlKSE1iYTlnb0VLWSJ9.eyJhdWQiOiJodHRwczovL2dyYXBoLndpbmRv****G5gUTV-kKorR-pg"}

### Abrufen der Objekt-ID des Dienstprinzipals der Anwendung im Azure AD-Verzeichnis des Benutzers
Verwenden Sie jetzt das nur für die App geltende Zugriffstoken, um die API für [Azure AD Graph-Dienstprinzipale](https://msdn.microsoft.com/Library/Azure/Ad/Graph/api/entity-and-complex-type-reference#serviceprincipal-entity) abzufragen und die Objekt-ID des Dienstprinzipals der Anwendung im Verzeichnis zu ermitteln.

Die Methode [GetObjectIdOfServicePrincipalInOrganization](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureADGraphAPIUtil.cs#) der ASP.NET MVC-Beispielanwendung implementiert diesen Aufruf.

Im folgenden Beispiel wird gezeigt, wie der Dienstprinzipal einer Anwendung angefordert wird. „a0448380-c346-4f9f-b897-c18733de9394“ ist die Client-ID der Anwendung.

    GET https://graph.windows.net/62e173e9-301e-423e-bcd4-29121ec1aa24/servicePrincipals?api-version=1.5&$filter=appId%20eq%20'a0448380-c346-4f9f-b897-c18733de9394' HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJK*****-kKorR-pg

Das folgende Beispiel zeigt eine Antwort auf die Anforderung des Dienstprinzipals einer Anwendung.

    HTTP/1.1 200 OK

    {"odata.metadata":"https://graph.windows.net/62e173e9-301e-423e-bcd4-29121ec1aa24/$metadata#directoryObjects/Microsoft.DirectoryServices.ServicePrincipal","value":[{"odata.type":"Microsoft.DirectoryServices.ServicePrincipal","objectType":"ServicePrincipal","objectId":"9b5018d4-6951-42ed-8a92-f11ec283ccec","deletionTimestamp":null,"accountEnabled":true,"appDisplayName":"CloudSense","appId":"a0448380-c346-4f9f-b897-c18733de9394","appOwnerTenantId":"62e173e9-301e-423e-bcd4-29121ec1aa24","appRoleAssignmentRequired":false,"appRoles":[],"displayName":"CloudSense","errorUrl":null,"homepage":"http://www.vipswapper.com/cloudsense","keyCredentials":[],"logoutUrl":null,"oauth2Permissions":[{"adminConsentDescription":"Allow the application to access CloudSense on behalf of the signed-in user.","adminConsentDisplayName":"Access CloudSense","id":"b7b7338e-683a-4796-b95e-60c10380de1c","isEnabled":true,"type":"User","userConsentDescription":"Allow the application to access CloudSense on your behalf.","userConsentDisplayName":"Access CloudSense","value":"user_impersonation"}],"passwordCredentials":[],"preferredTokenSigningKeyThumbprint":null,"publisherName":"vipswapper"quot;,"replyUrls":["http://www.vipswapper.com/cloudsense","http://www.vipswapper.com","http://vipswapper.com","http://vipswapper.azurewebsites.net","http://localhost:62080"],"samlMetadataUrl":null,"servicePrincipalNames":["http://www.vipswapper.com/cloudsense","a0448380-c346-4f9f-b897-c18733de9394"],"tags":["WindowsAzureActiveDirectoryIntegratedApp"]}]}

### Abrufen des Bezeichners der Azure-RBAC-Rolle
Damit Sie Ihrem Dienstprinzipal die entsprechende RBAC-Rolle zuweisen können, müssen sie den Bezeichner der Azure-RBAC-Rolle bestimmen.

Die richtige RBAC-Rolle für Ihre Anwendung:

* Wenn Ihre Anwendung das Abonnement nur überwacht, ohne Änderungen vorzunehmen, sind für das Abonnement nur Leseberechtigungen erforderlich. Weisen Sie die Rolle **Leser** zu.
* Wenn Ihre Anwendung das Azure-Abonnement verwaltet und Entitäten erstellt/ändert/löscht, sind die Berechtigungen eines Mitwirkenden erforderlich.
  * Um einen bestimmten Typ von Ressourcen zu verwalten, weisen Sie die ressourcenspezifischen Rollen für Mitwirkende zu (Mitwirkender von virtuellen Computern, Mitwirkender von virtuellem Netzwerk, Speicherkontomitwirkender usw.).
  * Weisen Sie die Rolle **Mitwirkender** zu, um einen beliebigen Ressourcentyp zu verwalten.

Die Rollenzuweisung für die Anwendung wird den Benutzern angezeigt, wählen Sie also nur die mindestens erforderlichen Berechtigungen aus.

Rufen Sie die [Resource Manager-API für die Rollendefinition](https://msdn.microsoft.com/library/azure/dn906879.aspx) zum Auflisten aller Azure-RBAC-Rollen auf. Durchsuchen Sie anschließend das Ergebnis, um die gewünschte Rollendefinition anhand des Namens zu ermitteln.

Die Methode [GetRoleId](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L246) der ASP.NET MVC-Beispiel-App implementiert diesen Aufruf.

Das folgende Anforderungsbeispiel zeigt, wie Azure-RBAC-Rollenbezeichner abgerufen werden. „09cbd307-aa71-4aca-b346-5f253e6e3ebb“ ist die ID des Abonnements.

    GET https://management.azure.com/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions?api-version=2015-07-01 HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV*****fY2lGc5

Die Antwort hat folgendes Format:

    HTTP/1.1 200 OK

    {"value":[{"properties":{"roleName":"API Management Service Contributor","type":"BuiltInRole","description":"Lets you manage API Management services, but not access to them.","scope":"/","permissions":[{"actions":["Microsoft.ApiManagement/Services/*","Microsoft.Authorization/*/read","Microsoft.Resources/subscriptions/resources/read","Microsoft.Resources/subscriptions/resourceGroups/read","Microsoft.Resources/subscriptions/resourceGroups/resources/read","Microsoft.Resources/subscriptions/resourceGroups/deployments/*","Microsoft.Insights/alertRules/*","Microsoft.Support/*"],"notActions":[]}]},"id":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/312a565d-c81f-4fd8-895a-4e21e48d571c","type":"Microsoft.Authorization/roleDefinitions","name":"312a565d-c81f-4fd8-895a-4e21e48d571c"},{"properties":{"roleName":"Application Insights Component Contributor","type":"BuiltInRole","description":"Lets you manage Application Insights components, but not access to them.","scope":"/","permissions":[{"actions":["Microsoft.Insights/components/*","Microsoft.Insights/webtests/*","Microsoft.Authorization/*/read","Microsoft.Resources/subscriptions/resources/read","Microsoft.Resources/subscriptions/resourceGroups/read","Microsoft.Resources/subscriptions/resourceGroups/resources/read","Microsoft.Resources/subscriptions/resourceGroups/deployments/*","Microsoft.Insights/alertRules/*","Microsoft.Support/*"],"notActions":[]}]},"id":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/ae349356-3a1b-4a5e-921d-050484c6347e","type":"Microsoft.Authorization/roleDefinitions","name":"ae349356-3a1b-4a5e-921d-050484c6347e"}]}

Sie müssen diese API nicht regelmäßig aufrufen. Wenn Sie die bekannte GUID der Rollendefinition ermittelt haben, können Sie die ID von Rollendefinitionen wie folgt erstellen:

    /subscriptions/{subscription_id}/providers/Microsoft.Authorization/roleDefinitions/{well-known-role-guid}

Dies sind die bekannten GUIDs der häufig verwendeten integrierten Rollen:

| Rolle | Guid |
| --- | --- |
| Leser |acdd72a7-3385-48ef-bd42-f606fba81ae7 |
| Mitwirkender |b24988ac-6180-42a0-ab88-20f7382dd24c |
| Mitwirkender von virtuellen Computern |d73bb868-a0df-4d4d-bd69-98a00b01fccb |
| Mitwirkender von virtuellem Netzwerk |b34d265f-36f7-4a0d-a4d4-e158ca92e90f |
| Mitwirkender von Speicherkonto |86e8f5dc-a6e9-4c67-9d15-de283e8eac25 |
| Mitwirkender von Website |de139f84-1756-47ae-9be6-808fbbe84772 |
| Mitwirkender von Webplan |2cc479cb-7b4d-49a8-b449-8c00fd0f0a4b |
| Mitwirkender von SQL Server |6d8ee4ec-f05a-4a1d-8b00-a9b17e38b437 |
| Mitwirkender von SQL DB |9b7fa17d-e63e-47b0-bb0a-15c516ac86ec |

### Zuweisen der RBAC-Rolle zur Anwendung
Sie haben alles, was Sie benötigen, um dem Dienstprinzipal mithilfe der [Resource Manager-API zum Erstellen einer Rollenzuweisung](https://msdn.microsoft.com/library/azure/dn906887.aspx) die entsprechende RBAC-Rolle zuzuweisen.

Die Methode [GrantRoleToServicePrincipalOnSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L170) der ASP.NET MVC-Beispiel-App implementiert diesen Aufruf.

Beispielanforderung zum Zuweisen einer RBAC-Rolle zur Anwendung:

    PUT https://management.azure.com/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/microsoft.authorization/roleassignments/4f87261d-2816-465d-8311-70a27558df4c?api-version=2015-07-01 HTTP/1.1

    Authorization: Bearer eyJ0eXAiOiJKV1QiL*****FlwO1mM7Cw6JWtfY2lGc5
    Content-Type: application/json
    Content-Length: 230

    {"properties": {"roleDefinitionId":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7","principalId":"c3097b31-7309-4c59-b4e3-770f8406bad2"}}

In der Anforderung werden die folgenden Werte verwendet:

| GUID | Beschreibung |
| --- | --- |
| 09cbd307-aa71-4aca-b346-5f253e6e3ebb |Die ID des Abonnements |
| c3097b31-7309-4c59-b4e3-770f8406bad2 |Die Objekt-ID des Dienstprinzipals der Anwendung |
| acdd72a7-3385-48ef-bd42-f606fba81ae7 |Die ID der Leserrolle |
| 4f87261d-2816-465d-8311-70a27558df4c |Eine neue GUID, erstellt für die neue Rollenzuweisung |

Die Antwort hat folgendes Format:

    HTTP/1.1 201 Created

    {"properties":{"roleDefinitionId":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleDefinitions/acdd72a7-3385-48ef-bd42-f606fba81ae7","principalId":"c3097b31-7309-4c59-b4e3-770f8406bad2","scope":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb"},"id":"/subscriptions/09cbd307-aa71-4aca-b346-5f253e6e3ebb/providers/Microsoft.Authorization/roleAssignments/4f87261d-2816-465d-8311-70a27558df4c","type":"Microsoft.Authorization/roleAssignments","name":"4f87261d-2816-465d-8311-70a27558df4c"}

### Abrufen eines nur für die App geltenden Zugriffstokens für Azure Resource Manager
Führen Sie im Abonnement mithilfe eines Tokens, das nur für die App gilt, eine Testaufgabe aus, um zu prüfen, ob die App über den gewünschten Zugriff verfügt.

Befolgen Sie die Anweisungen im Abschnitt [Abrufen eines nur für die App geltenden Zugriffstokens für die Azure AD Graph-API](#app-azure-ad-graph), und verwenden Sie dabei einen anderen Wert für den Ressourcenparameter:

    https://management.core.windows.net/

Die Metode [ServicePrincipalHasReadAccessToSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L110) der ASP.NET MVC-Beispielanwendung ruft unter Verwendung der Active Directory Authentication Library für .NET ein nur für die App geltendes Zugriffstoken für Azure Resource Manager ab.

#### Abrufen der Anwendungsberechtigungen für das Abonnement
Um zu überprüfen, ob die Anwendung den gewünschten Zugriff auf ein Azure-Abonnement hat, können Sie auch die [Resource Manager-API für Berechtigungen](https://msdn.microsoft.com/library/azure/dn906889.aspx) aufrufen. Dies ist eine ähnliche Vorgehensweise wie wenn Sie bestimmen, ob der Benutzer über Zugriffsverwaltungsrechte für das Abonnement verfügt. Rufen Sie jedoch dieses Mal die Berechtigungs-API mit dem nur für die App geltenden Token auf, das Sie im vorherigen Schritt erhalten haben.

Die Methode [ServicePrincipalHasReadAccessToSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L110) der ASP.NET MVC-Beispiel-App implementiert diesen Aufruf.

## Verwalten verbundener Abonnements
Wenn dem Dienstprinzipal Ihrer Anwendung die entsprechende RBAC-Rolle für das Abonnement zugewiesen wurde, kann Ihre Anwendung es mit den nur für die App geltenden Zugriffstoken für Azure Resource Manager überwachen und verwalten.

Wenn ein Abonnementbesitzer die Rollenzuweisung Ihrer Anwendung mithilfe des klassischen Portals oder über Befehlszeilentools entfernt, kann Ihre Anwendung nicht mehr auf das Abonnement zugreifen. In diesem Fall sollten Sie den Benutzer informieren, dass die Verbindung mit dem Abonnement von außerhalb der Anwendung unterbrochen wurde, und ihm die Möglichkeit geben, die Verbindung zu „reparieren“. Durch das „Reparieren“ wird einfach die Rollenzuweisung neu erstellt, die offline gelöscht wurde.

So wie Sie es dem Benutzer ermöglicht haben, seine Abonnements mit der Anwendung zu verbinden, müssen Sie ihm auch das Trennen von Abonnements ermöglichen. Aus Sicht der Zugriffsverwaltung bedeutet Trennen, dass die Rollenzuweisung entfernt wird, die der Dienstprinzipal der Anwendung für das Abonnement hat. Optional kann jeder Status in der Anwendung für das Abonnement ebenfalls entfernt werden. Nur Benutzer mit Berechtigungen zur Zugriffsverwaltung für das Abonnement können das Abonnement trennen.

Die Methode [RevokeRoleFromServicePrincipalOnSubscription](https://github.com/dushyantgill/VipSwapper/blob/master/CloudSense/CloudSense/AzureResourceManagerUtil.cs#L200) der ASP.NET MVC-Beispiel-App implementiert diesen Aufruf.

Jetzt können Benutzer Ihre Anwendung verwenden, um problemlos eine Verbindung mit ihren Azure-Abonnements herzustellen und sie zu verwalten.

<!---HONumber=AcomDC_0831_2016-->