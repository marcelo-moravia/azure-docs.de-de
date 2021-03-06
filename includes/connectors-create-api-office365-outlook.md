#### Voraussetzungen
* Ein Azure-Konto. Sie können ein [kostenloses Konto](https://azure.microsoft.com/free) erstellen.
* Ein [Office 365](https://office365.com)-Konto

Bevor Sie Ihr Office 365-Konto in einer Logik-App verwenden, autorisieren Sie die Logik-App für die Verbindung mit Ihrem Office 365-Konto. Sie können dies ganz einfach in Ihrer Logik-App im Azure-Portal durchführen.

Führen Sie folgende Schritte aus, um Ihre Logik-App für die Verbindung mit Ihrem Office 365-Konto zu autorisieren:

1. Erstellen Sie eine Logik-App. Wählen Sie im Designer für Logik-Apps in der Dropdownliste **Von Microsoft verwaltete APIs anzeigen** aus, und geben Sie dann „Office 365“ in das Suchfeld ein. Wählen Sie einen Trigger oder eine Aktion aus: ![Schritt zur Erstellung der Office 365-Verbindung](./media/connectors-create-api-office365-outlook/office365-sendemail.png)
2. Wenn Sie bisher noch keine Verbindung mit Office 365 hergestellt haben, werden Sie aufgefordert, sich mit Ihren Office 365-Anmeldeinformationen anzumelden: ![Schritt zur Erstellung der Office 365-Verbindung](./media/connectors-create-api-office365-outlook/office365-signin.png)
3. Wählen Sie **Anmelden** aus, und geben Sie Ihren Benutzernamen und Ihr Kennwort ein. Wählen Sie **Anmelden** aus: ![Schritt zur Erstellung der Office 365-Verbindung](./media/connectors-create-api-office365-outlook/office365-usernamepassword.png)
   
    Mit diesen Anmeldeinformationen wird Ihre Logik-App autorisiert, eine Verbindung mit Ihrem Office 365-Konto herzustellen und darauf zuzugreifen.
4. Beachten Sie, dass die Verbindung erstellt wurde. Fahren Sie nun mit den weiteren Schritten in Ihrer Logik-App fort: ![Schritt zur Erstellung der Office 365-Verbindung](./media/connectors-create-api-office365-outlook/office365-sendemailproperties.png)

<!---HONumber=AcomDC_0727_2016-->