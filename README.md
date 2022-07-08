# powershell
Ici des solutions toutes faites en PS

TODO : pouquoi sous **pwsh** un simple ```> "Accentué"``` (object Strings) fait planter PS avec une erreur **iconv**

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

## Résultats dans fenêtre graphique avec possibilité de rechercher textuellement

Event Viewer plus pratique (basé sur [ceci](https://stackoverflow.com/questions/66532033/how-can-i-read-analytical-windows-events-from-applications-and-services-logs-u) et amélioré par [cela](https://devblogs.microsoft.com/scripting/use-powershell-to-create-and-to-use-a-new-event-log/))
```
Get-WinEvent -ListLog * | % { Get-WinEvent -LogName $_.LogName | Select -Property * } | Out-GridView
```
Nota : si LogName ne contient aucune entrée (cas de "Internet Explorer" après installation), une erreur s'affichera en rouge sans conséquences

Gestionnaire de services plus pratique (basé sur [ceci](https://stackoverflow.com/questions/59725591/powershell-get-service-detailed-description-of-the-windows-service))
```
Get-WmiObject win32_service | select * | ogv
```
De là on peut chercher le fameux service par son nom en FR ou par son nom de fichier EN

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
-OutFile "ps725_64.msi"
```
