## <a name="register-mobile-service-aad"></a>Registrieren des mobilen Diensts in Azure Active Directory
In diesem Abschnitt registrieren Sie den mobilen Dienst in Azure Active Directory und konfigurieren Berechtigungen, um Identitätswechsel für das einmalige Anmelden zu ermöglichen.

1. Registrieren Sie Ihre Anwendung in Azure Active Directory, indem Sie die Anleitung im Thema [Registrieren bei Azure Active Directory] befolgen.
2. Gehen Sie im [klassischen Azure-Portal](https://manage.windowsazure.com/) zurück zur Erweiterung "Azure Active Directory" und klicken Sie auf Ihr aktives Verzeichnis.
3. Klicken Sie auf die Registerkarte **Anwendungen** t und dann auf Ihre Anwendung.
4. Klicken Sie auf **Manifest verwalten**. Klicken Sie danach auf **Manifest herunterladen**, und speichern Sie das Anwendungsmanifest im lokalen Verzeichnis.
   
   ![](./media/mobile-services-dotnet-adal-register-service/mobile-services-aad-app-manage-manifest.png)
5. Öffnen Sie die Anwendungsmanifestdatei in Visual Studio. Suchen Sie am Anfang der Datei die Zeile mit den Anwendungsberechtigungen, die etwas wie folgt aussieht:
   
        "oauth2Permissions": [],
   
    Ersetzen Sie diese Zeile durch die folgenden Anwendungsberechtigungen, und speichern Sie die Datei.
   
        "oauth2Permissions": [
            {
                "adminConsentDescription": "Allow the application access to the mobile service",
                "adminConsentDisplayName": "Have full access to the mobile service",
                "id": "b69ee3c9-c40d-4f2a-ac80-961cd1534e40",
                "isEnabled": true,
                "origin": "Application",
                "type": "User",
                "userConsentDescription": "Allow the application full access to the mobile service on your behalf",
                "userConsentDisplayName": "Have full access to the mobile service",
                "value": "user_impersonation"
            }
        ],
6. Klicken Sie im [klassischen Azure-Portal](https://manage.windowsazure.com/) für die Anwendung erneut auf **Manifest verwalten** und anschließend auf **Manifest hochladen**. Navigieren Sie zum Speicherort des Anwendungsmanifests, das Sie gerade aktualisiert haben, und laden Sie das Manifest hoch.

<!-- URLs. -->
[Registrieren bei Azure Active Directory]: ../articles/mobile-services/mobile-services-how-to-register-active-directory-authentication.md

<!---HONumber=AcomDC_0128_2016-->