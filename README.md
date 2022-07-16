# powershell
Ici des solutions toutes faites en PS

Note : depuis 2016 PS est Open Source, installable et utilisable sous Linux (=> scripts en "objet") ; de plus sous Linux on peut utiliser **pwsh**

Typo : ici les commandes sont précédées du prompt minimal tel que ```> commande ...```


TODO : fusionner KB dédiée sur pwsh

TODO : fusionner KB dédiée sur accès distant

TODO : gestion correcte des caractères entre pwsh et Powershell + pourquoi avec **pwsh** sous Linux un simple ```> "Accentué"``` (object Strings) fait planter PS avec une erreur **iconv** et on revient à l'invite **pwsh**

## Mémo des alias

TODO : **alias** => ?

**gal** est un alias sur la commande **Get-Alias**

**%** est un alias sur la commande **foreach**

**?** est un alias sur la commande **Where-Object**

## Version de PS
```
> Get-Variable PSVersionTable -ValueOnly
Name                           Value
----                           -----
PSVersion                      4.0
WSManStackVersion              3.0
SerializationVersion           1.1.0.1
CLRVersion                     4.0.30319.34014
BuildVersion                   6.3.9600.17400
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0}
PSRemotingProtocolVersion      2.2  
```
ou directement (je ne sais pas si c'est possible depuis les premières versions)
```
> $PSVersionTable
Name                           Value
----                           -----
PSVersion                      5.1.14409.1005
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.14409.1005
CLRVersion                     4.0.30319.34014
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

## Installer une autre version de PS (W7/8/10)
[Source](https://www.tenforums.com/tutorials/151734-how-install-powershell-7-windows-7-windows-8-windows-10-a.html)

Télécharger la version de PS voulue [ici](https://github.com/PowerShell/PowerShell/releases/tag/v7.2.5) et l'installer en administrateur.

Note : on peut aussi télécharger depuis PS avec wget (voir rubrique dédiée)

## Se connecter à distance depuis un serveur PS
### Depuis le serveur PS
Obtenir les informations de configuration de sesssion :
```
> Get-PSSessionConfiguration
...
Name          : PowerShell.7.2.5
PSVersion     : 7.2
StartupScript : 
RunAsUser     : 
Permission    : AUTORITE NT\INTERACTIF AccessAllowed, BUILTIN\Administrateurs AccessAllowed,
                BUILTIN\Utilisateurs de gestion à distance AccessAllowed
```
### Depuis le client PS
(par exemple depuis **pwsh** sous Linux) :
```
> Enter-PSSession -ComputerName $Host -Credential $User -Authentication Negotiate
```
Si il y a plusieurs serveur PS, il faut préciser sur lequel on se connecte avec le paramètre ```-ConfigurationName $SessionConfigName``` ([source](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enable-psremoting?view=powershell-7.2))
où **$SessionConfigName** est le nom de la configuration de la session renvoyée par la commande **Get-PSSessionConfiguration** depuis le serveur PS (voir dessus.

ex : > Enter-PSSession ... -ConfigurationName "PowerShell.7.2.5"

## Rendre la connexion distante persistante
Modifier le type de démarrage du service "Gestion à distance de Windows (Gestion WSM)" (WinRM) de "Automatique (début différé)" à "Automatique"

Note : pour déterminer le nom du service lié à WinRM, j'ai cherché toutes les infos d'un service qui se réfèrent à winrm (voir [rubrique concernée](#obtenir-des-informations-sur-un-service))

## Résultats dans fenêtre graphique avec possibilité de rechercher textuellement

Event Viewer plus pratique (basé sur [ceci](https://stackoverflow.com/questions/66532033/how-can-i-read-analytical-windows-events-from-applications-and-services-logs-u) et amélioré par [cela](https://devblogs.microsoft.com/scripting/use-powershell-to-create-and-to-use-a-new-event-log/))
```
> Get-WinEvent -ListLog * | % { Get-WinEvent -LogName $_.LogName | Select -Property * } | Out-GridView
```
Nota : si LogName ne contient aucune entrée (cas de "Internet Explorer" après installation), une erreur s'affichera en rouge sans conséquences

Gestionnaire de services plus pratique (basé sur [ceci](https://stackoverflow.com/questions/59725591/powershell-get-service-detailed-description-of-the-windows-service))
```
> Get-WmiObject win32_service | select * | ogv
```
De là on peut chercher le fameux service par son nom en FR ou par son nom de fichier EN

## Obtenir des informations sur un service
```
> Get-WmiObject win32_service | select * | ? { $_ -Match $Pattern }
```
ex : $Pattern = "winrm"

Remarque : $Pattern ne doit PAS contenir de wildcard (*)

## Chronométrer commande
[Source](https://webdevdesigner.com/q/timing-a-commands-execution-in-powershell-667044/)
```
> "Temps execution : " + ( Measure-Command { Test-Connection -Count 9 8.8.8.8 | Out-Host } ).ToString( "hh\:mm\:ss" ) 
```
Exécute la commande en affichant la sortie avec **Out-Host**, renvoie la durée avec **Measure-Command** et enfin convertit en hh:mm:ss avec **ToString**

Nota : les **:** doivent être échappés et on peut optionnellement faire précéder la durée par un texte

## Télécharger un fichier depuis un serveur web
Si le téléchargement échoue en indiquant "Invoke-WebRequest : The request was aborted: Could not create SSL/TLS secure channel." c'est que le site  nécessite SSL/TLS. Il faut alors changer le niveau de sécurité du protocole ([source](https://stackoverflow.com/questions/41618766/powershell-invoke-webrequest-fails-with-ssl-tls-secure-channel)).
```
> [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12 ; `
   wget "https://github.com/PowerShell/PowerShell/releases/download/v7.2.5/PowerShell-7.2.5-win-x64.msi" `
   -OutFile "PowerShell-7.2.5-win-x64.msi"
```
## Écrire la sortie complète sans adapter (fonctionnalité "No Wrap")
Quand on copier tel quel certains résultats de commande, il faut remanier pour éliminer les sauts de lignes et redonner l'unité d'un contenu
```
> COMMAND | Write-Host
```
Nota : ne pas confondre avec tronquer (Truncate) qui n'affiche pas en entier un contenu et termine par une élipse pour indiquer la troncature
