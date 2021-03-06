---
title: Verwalten der rollenbasierten Zugriffssteuerung (Role-Based Access Control, RBAC) mit Azure PowerShell | Microsoft Docs
description: So verwalten Sie RBAC mit Azure PowerShell einschließlich der Auflistung von Rollen, der Zuweisung von Rollen und dem Löschen von Rollenzuweisungen.
services: active-directory
documentationcenter: ''
author: kgremban
manager: femila
editor: ''

ms.service: active-directory
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/22/2016
ms.author: kgremban

---
# Verwalten der rollenbasierten Zugriffssteuerung mit Azure PowerShell
> [!div class="op_single_selector"]
> * [PowerShell](role-based-access-control-manage-access-powershell.md)
> * [Azure-Befehlszeilenschnittstelle](role-based-access-control-manage-access-azure-cli.md)
> * [REST-API](role-based-access-control-manage-access-rest.md)
> 
> 

Mit der rollenbasierten Zugriffssteuerung (Role-Based Access Control, RBAC) im Azure-Portal und in der Azure Resource Management-API können Sie den Zugriff auf Ihr Abonnement differenziert steuern. Mithilfe dieser Funktion lassen sich Zugriffsberechtigungen für Active Directory-Benutzer, -Gruppen oder -Dienstprinzipale festlegen, indem ihnen bestimmte Rollen für einen bestimmten Bereich zugewiesen werden.

Bevor Sie PowerShell verwenden können, um die RBAC zu verwalten, benötigen Sie Folgendes:

* Azure PowerShell, Version 0.8.8 oder höher. Um die neueste Version zu installieren und sie Ihrem Azure-Abonnement zuzuordnen, lesen Sie [Installieren und Konfigurieren von Azure PowerShell](../powershell-install-configure.md).
* Azure Resource Manager-Cmdlets Installieren Sie die [Azure Resource Manager-Cmdlets](https://msdn.microsoft.com/library/mt125356.aspx) in PowerShell.

## Auflisten der Rollen
### Auflisten aller verfügbaren Rollen
Zum Auflisten von verfügbaren RBAC-Rollen für die Zuweisung und Überprüfung der Vorgänge, auf die sie Zugriff gewähren, verwenden Sie `Get-AzureRmRoleDefinition`.

```
Get-AzureRmRoleDefinition | FT Name, Description
```

![RBAC PowerShell – Get-AzureRmRoleDefinition – Screenshot](./media/role-based-access-control-manage-access-powershell/1-get-azure-rm-role-definition1.png)

### Auflisten der Aktionen einer Rolle
Zum Auflisten der Aktionen für eine bestimmte Rolle verwenden Sie `Get-AzureRmRoleDefinition <role name>`.

```
Get-AzureRmRoleDefinition Contributor | FL Actions, NotActions

(Get-AzureRmRoleDefinition "Virtual Machine Contributor").Actions
```

![RBAC PowerShell – Get-AzureRmRoleDefinition für eine bestimmte Rolle – Screenshot](./media/role-based-access-control-manage-access-powershell/1-get-azure-rm-role-definition2.png)

## Sehen, wer Zugriff hat
Zum Auflisten der RBAC-Zugriffszuweisungen verwenden Sie `Get-AzureRmRoleAssignment`.

### Auflistung der Rollenzuweisungen für einen bestimmten Bereich
Sie können alle Zugriffszuweisungen für ein angegebenes Abonnement, eine angegebene Ressourcengruppe oder Ressource sehen. Um beispielsweise alle aktiven Zuweisungen für eine Ressourcengruppe zu sehen, verwenden Sie `Get-AzureRmRoleAssignment -ResourceGroupName <resource group name>`.

```
Get-AzureRmRoleAssignment -ResourceGroupName Pharma-Sales-ProjectForcast | FL DisplayName, RoleDefinitionName, Scope
```

![RBAC PowerShell – Get-AzureRmRoleAssignment für eine Ressourcengruppe – Screenshot](./media/role-based-access-control-manage-access-powershell/4-get-azure-rm-role-assignment1.png)

### Zugewiesenen Rollen eines Benutzers auflisten
Verwenden Sie `Get-AzureRmRoleAssignment -SignInName <User email> -ExpandPrincipalGroups` zum Auflisten aller Rollen, die einem bestimmten Benutzer zugewiesen sind, und der Rollen, die den Gruppen zugewiesen sind, zu denen der Benutzer gehört.

```
Get-AzureRmRoleAssignment -SignInName sameert@aaddemo.com | FL DisplayName, RoleDefinitionName, Scope

Get-AzureRmRoleAssignment -SignInName sameert@aaddemo.com -ExpandPrincipalGroups | FL DisplayName, RoleDefinitionName, Scope
```

![RBAC PowerShell – Get-AzureRmRoleAssignment für einen Benutzer – Screenshot](./media/role-based-access-control-manage-access-powershell/4-get-azure-rm-role-assignment2.png)

### Auflisten der klassischen Rollenzuweisungen für die Dienstadministrator- und Co-Administrator-Rolle
Zum Auflisten der Zugriffszuweisungen für die klassischen Abonnementadministratoren und -Co-Administratoren verwenden Sie Folgendes:

    Get-AzureRmRoleAssignment -IncludeClassicAdministrators

## Gewähren von Zugriff
### Suchen nach Objekt-IDs
Um eine Rolle zuzuweisen, müssen Sie sowohl das Objekt (Benutzer, Gruppe oder Anwendung) als auch den Bereich identifizieren.

Falls Sie die Abonnement-ID nicht kennen, können Sie diese im Blatt **Abonnements** im Azure-Portal finden. Wie Sie die Abonnement-ID abfragen, erfahren Sie im MSDN unter [Get-AzureSubscription](https://msdn.microsoft.com/library/dn495302.aspx).

So rufen Sie die Objekt-ID für eine Azure AD-Gruppe ab:

    Get-AzureRmADGroup -SearchString <group name in quotes>

So rufen Sie die Objekt-ID für einen Azure AD-Dienstprinzipal oder eine Anwendung ab:

    Get-AzureRmADServicePrincipal -SearchString <service name in quotes>

### Zuweisen einer Rolle zu einer Anwendung im Abonnementkontext
So gewähren Sie einer Anwendung Zugriff im Abonnementkontext:

    New-AzureRmRoleAssignment -ObjectId <application id> -RoleDefinitionName <role name> -Scope <subscription id>

![RBAC PowerShell – New-AzureRmRoleAssignment – Screenshot](./media/role-based-access-control-manage-access-powershell/2-new-azure-rm-role-assignment2.png)

### Zuweisen einer Rolle zu einem Benutzer im Ressourcengruppenkontext
So gewähren Sie einem Benutzer Zugriff im Ressourcengruppenkontext:

    New-AzureRmRoleAssignment -SignInName <email of user> -RoleDefinitionName <role name in quotes> -ResourceGroupName <resource group name>

![RBAC PowerShell – New-AzureRmRoleAssignment – Screenshot](./media/role-based-access-control-manage-access-powershell/2-new-azure-rm-role-assignment3.png)

### Zuweisen einer Rolle zu einer Gruppe im Ressourcenkontext
So gewähren Sie einer Gruppe Zugriff im Ressourcenkontext:

    New-AzureRmRoleAssignment -ObjectId <object id> -RoleDefinitionName <role name in quotes> -ResourceName <resource name> -ResourceType <resource type> -ParentResource <parent resource> -ResourceGroupName <resource group name>

![RBAC PowerShell – New-AzureRmRoleAssignment – Screenshot](./media/role-based-access-control-manage-access-powershell/2-new-azure-rm-role-assignment4.png)

## Zugriff entfernen
So entfernen Sie den Zugriff für Benutzer, Gruppen und Anwendungen:

    Remove-AzureRmRoleAssignment -ObjectId <object id> -RoleDefinitionName <role name> -Scope <scope such as subscription id>

![RBAC PowerShell – Remove-AzureRmRoleAssignment - screenshot](./media/role-based-access-control-manage-access-powershell/3-remove-azure-rm-role-assignment.png)

## Erstellen einer benutzerdefinierten Rolle
Verwenden Sie zum Erstellen einer benutzerdefinierten Rolle den Befehl `New-AzureRmRoleDefinition`.

Wenn Sie eine benutzerdefinierte Rolle in PowerShell erstellen, müssen Sie mit einer der [integrierten Rollen](role-based-access-built-in-roles.md) beginnen. Bearbeiten Sie die Attribute, und fügen Sie die gewünschten *Actions*, *notActions* oder *scopes* hinzu. Speichern Sie die Änderungen anschließend als neue Rolle.

Das folgende Beispiel beginnt mit der Rolle *Virtual Machine Contributor*. Diese Rolle wird zum Erstellen einer benutzerdefinierten Rolle namens *Virtual Machine Operator* verwendet. Die neue Rolle gewährt Zugriff auf alle Lesevorgänge der Ressourcenanbieter *Microsoft.Compute*, *Microsoft.Storage* und *Microsoft.Network* sowie zum Starten, Neustarten und Überwachen virtueller Computer. Die benutzerdefinierte Rolle kann in zwei Abonnements verwendet werden.

```
$role = Get-AzureRmRoleDefinition "Virtual Machine Contributor"
$role.Id = $null
$role.Name = "Virtual Machine Operator"
$role.Description = "Can monitor and restart virtual machines."
$role.Actions.Clear()
$role.Actions.Add("Microsoft.Storage/*/read")
$role.Actions.Add("Microsoft.Network/*/read")
$role.Actions.Add("Microsoft.Compute/*/read")
$role.Actions.Add("Microsoft.Compute/virtualMachines/start/action")
$role.Actions.Add("Microsoft.Compute/virtualMachines/restart/action")
$role.Actions.Add("Microsoft.Authorization/*/read")
$role.Actions.Add("Microsoft.Resources/subscriptions/resourceGroups/read")
$role.Actions.Add("Microsoft.Insights/alertRules/*")
$role.Actions.Add("Microsoft.Support/*")
$role.AssignableScopes.Clear()
$role.AssignableScopes.Add("/subscriptions/c276fc76-9cd4-44c9-99a7-4fd71546436e")
$role.AssignableScopes.Add("/subscriptions/e91d47c4-76f3-4271-a796-21b4ecfe3624")
New-AzureRmRoleDefinition -Role $role
```

![RBAC PowerShell – Get-AzureRmRoleDefinition – Screenshot](./media/role-based-access-control-manage-access-powershell/2-new-azurermroledefinition.png)

## Ändern einer benutzerdefinierten Rolle
Um eine benutzerdefinierte Rolle zunächst zu ändern, verwenden Sie den Befehl `Get-AzureRmRoleDefinition`, um die Rollendefinition abzurufen. Nehmen Sie zweitens die gewünschten Änderungen an der Rollendefinitionsdatei vor. Verwenden Sie abschließend den Befehl `Set-AzureRmRoleDefinition`, um die geänderte Rollendefinition zu speichern.

Im folgenden Beispiel wird der Vorgang `Microsoft.Insights/diagnosticSettings/*` zur benutzerdefinierten Rolle *Virtual Machine Operator* hinzugefügt.

```
$role = Get-AzureRmRoleDefinition "Virtual Machine Operator"
$role.Actions.Add("Microsoft.Insights/diagnosticSettings/*")
Set-AzureRmRoleDefinition -Role $role
```

![RBAC PowerShell – Set-AzureRmRoleDefinition – Screenshot](./media/role-based-access-control-manage-access-powershell/3-set-azurermroledefinition-1.png)

Im folgenden Beispiel wird ein Azure-Abonnement den zuweisbaren Bereichen der benutzerdefinierten Rolle *Virtual Machine Operator* hinzugefügt.

```
Get-AzureRmSubscription - SubscriptionName Production3

$role = Get-AzureRmRoleDefinition "Virtual Machine Operator"
$role.AssignableScopes.Add("/subscriptions/34370e90-ac4a-4bf9-821f-85eeedead1a2"
Set-AzureRmRoleDefinition -Role $role)
```

![RBAC PowerShell – Set-AzureRmRoleDefinition – Screenshot](./media/role-based-access-control-manage-access-powershell/3-set-azurermroledefinition-2.png)

## Löschen einer benutzerdefinierten Rolle
Verwenden Sie zum Löschen einer benutzerdefinierten Rolle den Befehl `Remove-AzureRmRoleDefinition`.

Im folgenden Beispiel wird die benutzerdefinierte Rolle *Virtual Machine Operator* entfernt.

```
Get-AzureRmRoleDefinition "Virtual Machine Operator"

Get-AzureRmRoleDefinition "Virtual Machine Operator" | Remove-AzureRmRoleDefinition
```

![RBAC PowerShell – Remove-AzureRmRoleDefinition – Screenshot](./media/role-based-access-control-manage-access-powershell/4-remove-azurermroledefinition.png)

## Auflisten benutzerdefinierter Rollen
Zum Auflisten der Rollen, die in einem Bereich für die Zuweisung verfügbar sind, wählen Sie den Befehl `Get-AzureRmRoleDefinition`.

Im folgenden Beispiel werden alle Rollen aufgelistet, die für die Zuweisung im ausgewählten Abonnement verfügbar sind.

```
Get-AzureRmRoleDefinition | FT Name, IsCustom
```

![RBAC PowerShell – Get-AzureRmRoleDefinition – Screenshot](./media/role-based-access-control-manage-access-powershell/5-get-azurermroledefinition-1.png)

Im folgenden Beispiel ist die benutzerdefinierte Rolle *Virtual Machine Operator* im Abonnement *Production4* nicht verfügbar, da dieses Abonnement nicht in **AssignableScopes** für die Rolle enthalten ist.

![RBAC PowerShell – Get-AzureRmRoleDefinition – Screenshot](./media/role-based-access-control-manage-access-powershell/5-get-azurermroledefinition2.png)

## Siehe auch
* [Verwenden von Windows PowerShell mit dem Azure-Ressourcen-Manager](../powershell-azure-resource-manager.md)
  [!INCLUDE [role-based-access-control-toc.md](../../includes/role-based-access-control-toc.md)]

<!---HONumber=AcomDC_0810_2016-->