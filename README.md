# Powershell
Ici des solutions toutes faites en PS

Note : depuis 2016 PS est Open Source, installable et utilisable sous Linux (=> scripts en "objet") ; de plus sous Linux on peut utiliser **pwsh**

Typo : ici les commandes sont précédées du prompt minimal tel que ```> commande ...```


TODO : fusionner KB dédiée sur pwsh

TODO : fusionner KB dédiée sur accès distant

TODO : gestion correcte des caractères entre pwsh et Powershell

TODO : pourquoi avec **pwsh** sous Linux un simple ```> "Accentué"``` (object Strings) fait planter PS avec une erreur **iconv** et on revient à l'invite **pwsh**

## Mémo des alias et raccourcis
### Alias
**gal** est un alias sur la commande **Get-Alias**

Note : la commande **alias** n'est pas un alias mais un raccourci (voir dessous les raccourcis de commande)

**%** ou **foreach** est un alias sur la commande **ForEach-Object**

**?** est un alias sur la commande **Where-Object**

Trouver toutes les infos d'un alias ($Pattern ne peut pas contenir de wildcards comme * ) :
```
> alias | select * | ? { $_ -Match $Pattern }
```

### Raccourcis de commandes
Au lieu de Get-**Alias** on peut taper (sans distinction de casse) **alias** tout simplement (voir aussi [debugguer une commande](#debugguer-une-commande) pour plus de compréhension)

Attention : utiliser un raccourci peut être dangereux car les fichiers exécutables ont précédence sur les raccourcis ; ainsi si depuis le dossier en cours de PS, j'ai un exécutable (binaire ou script) nommé **alias** (soit alias.exe, alias.msi, alias.com, alias.bat, alias.ps, etc sans distinction de casse,  donc ALIAS.EXE sera aussi candidat), il aura précédence sur le raccourci et en croyant exécuter le raccourci on exécutera l'exécutable du dossier en cours, donc méfiance ([source](https://stackoverflow.com/questions/21033379/what-is-the-alias-keyword-in-powershell/21052658#21052658))

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

## Lister les volumes
### Pour tous les providers
```
> Get-PSDrive
```

### Pour les systèmes de fichiers uniquement
```
> Get-PSDrive -PSProvider 'FileSystem' | Format-table -autosize 

Name Used (GB) Free (GB) Provider   Root      CurrentLocation
---- --------- --------- --------   ----      ---------------
C        16,08      3,58 FileSystem C:\  Users\user\Documents
D       220,61    710,91 FileSystem H:\     
```

## Gérer les processus
```
> Get-Process
```
Ex : trouver le processus lié à "wsmprovhost" pour le supprimer (cas concret rencontré lors de l'interruption d'une copie de fichier depuis Linux par **pwsh** où le fichier était insupprimable depuis Windows avec un message "Cette action ne peut être réalisée car le fichier est ouvert dans wsmprovhost.exe")
```
> Get-Process | ? { $_.Name -Match "wsmprovhost" } | Select -Last 1 | Select Name , ID , Description                              

Name          Id Description
----          -- -----------
wsmprovhost 2864 Host process for WinRM plug-ins
```
=> supprimer ce processus a résolu le problème (en graphique afficher les ID de processus pour ne pas supprimer le mauvais processus)

Note : ```Select -Last 1``` trouve le PID le plus récent ; si on supprime le plus ancien, ... on ferme la session !

## Copier avec progression
Avertissement : on peut le faire en PS pur, mais j'ai l'impression que la commande Copy-Item n'est PAS est interruptible comme robocopy
Basé sur [ceci](https://stackoverflow.com/questions/13883404/custom-robocopy-progress-bar-in-powershell/25334958#25334958)
```
> robocopy H:\ E:\ pbr_image.wim | %{$data = $_.Split([char]9); if("$($data[4])" -ne "") { $file = "$($data[4])"} ;Write-Progress "Percentage $($data[0])" -Activity "Robocopy" -CurrentOperation "$($file)" -ErrorAction SilentlyContinue; }
```

#### Installer un module PS
```
# Déterminer si le module WinSCP est déjà installé

#  Note : dans le cas de multi-installation de PS, si à l'ouverture de la session distante on a précisé quelle installation utiliser (-ConfigurationName), sans -ListAvailable le module ne sera pas listé

> get-module -ListAvailable | Where-object {$_.Name -like '*winscp*'}        

    Directory: C:\Users\user\Documents\PowerShell\Modules

ModuleType Version    PreRelease Name                                PSEdition ExportedCommands
---------- -------    ---------- ----                                --------- ----------------
Manifest   5.17.10.0             WinSCP                              Desk      {ConvertTo-WinSCPEscapedString, Copy-WinSCPItem, Get-WinSCPChildItem, Get-WinSCPItem…}

# Déterminer si WinSCP est disponible et dans le cas contraire l'installer en se laissant guider

> Install-Module -Name WinSCP


# Déterminer dans quel dossier est installé le module (voir dessus pourquoi on utilise -ListAvailable)

> Get-Module WinSCP -ListAvailable | ft Path

Path
----
C:\Users\user\Documents\PowerShell\Modules\WinSCP\5.17.10.0\WinSCP.psd1

# => installé dans C:\Users\user\Documents\PowerShell\Modules\WinSCP\5.17.10.0
```

## Communiquer entre Windows et Linux par PowerShell
### WinSCP

# TODO : télé-charger/verser dossier

# TODO : afficher progression lors du transfert

[Source](https://www.it-connect.fr/comment-utiliser-le-module-winscp-de-powershell/)

Requis : le module WinSCP (voir **Installer un module PS**)

Confirmer que dans le **\bin** on trouve bien l'exécutable **WinSCP.exe** :
```
> dir C:\Users\user\Documents\PowerShell\Modules\WinSCP\5.17.10.0\bin

    Directory: C:\Users\user\Documents\PowerShell\Modules\WinSCP\5.17.10.0\bin

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          24/02/2021    18:07       26847216 WinSCP.exe
-a---          13/07/2022    11:20             79 winscp.ini
```

Avertissement : dans ce guide la connexion est volatile et n'existe que le temps de la session PS ; dès qu'on éteint ou ferme la session, les variables n'existent plus

#### Ouvrir connexion SCP
```
# Créer une connexion WinSCP vers l'hôte SCP voulu authentifié par User/Password

#  Note : si Get-WinSCPHostKeyFingerprint échoue, c'est que les identifiants, le port sont erronés, etc. ou que le serveur n'est pas opérationnel sur l'hôte SCP => bien revérifier

> $SCPHost = "192.168.0.11"

> $WinSCPSessionOption = New-WinSCPSessionOption -HostName $SCPHost -Protocol Scp -PortNumber 22 -Credential (Get-Credential)            

PowerShell Credential Request: PowerShell credential request
Warning: A script or application on the remote computer 192.168.0.10 is requesting your credentials. Enter your credentials only if you trust the remote computer and
 the application or script that is requesting them.

Enter your credentials.
User: user
Password for user user: ***********

# [option] modifier certains réglages avant d'ouvrir la session : $WinSCPSessionOption.PortNumber = 222

> $sshHostKeyFingerprint = Get-WinSCPHostKeyFingerprint -SessionOption $WinSCPSessionOption -Algorithm SHA-256

> $WinSCPSessionOption.SshHostKeyFingerprint = $sshHostKeyFingerprint

> $WinSCPSession = New-WinSCPSession -SessionOption ($WinSCPSessionOption)

# => si la dernière commande a été exécutée avec succès, c'est que la connexion est opérationnelle


# Confirmer que la session est bien ouverte

> Get-WinSCPSession

Opened       Timeout HostName
------       ------- --------
True        00:01:00 192.168.0.11


# Obtenir le chemin actuel par défaut depuis le client SCP de la session

> Get-WinSCPChildItem -WinSCPSession $WinSCPSession -Path .

   Directory: ./home/user

Mode                  LastWriteTime     Length Name
----                  -------------     ------ ----
rwxr-xr-x       18/06/2022 12:13:13            .audacity-data
rw-r--r--       14/07/2022 21:06:34     911302 .bash_history
...

# => par défaut on est dans le home de l'utilisateur user
```

#### Téléverser par SCP
```
# Téléverser depuis serveur WinSCP vers hôte client SCP (exemple : installer Bonjour Avahi/mDNS/ZeroConf pour communiquer par NOM pour éviter les changements d'adresse IP dus au DHCP)

# Exemple : copier l'historique des commandes PS depuis le serveur WinSCP vers le client SCP

> history > history.txt

> Send-WinSCPItem -WinSCPSession $WinSCPSession -Path ".\history.txt" -Destination "/home/user/Bureau/"                 

   Destination: \home\user\Bureau

IsSuccess FileName
--------- --------
True      history.txt
```

#### Télécharger par SCP
```
# Télécharger depuis hôte client SCP vers serveur WinSCP

#  Attention : si on se connecte depuis PWSH, il ne faut PAS que -RemotePath contienne autre chose que de l'ASCII (ex : pas d'accents), sinon la commande fait planter la connexion distante avec une erreur iconv et on revient à l'invite pwsh ( - voir https://github.com/lenainjaune/powershell/edit/main/README.md)


# Exemple : télécharger Bonjour Avahi/mDNS/ZeroConf pour permettre de communiquer par NOM et ainsi éviter les changements d'adresse IP dus au DHCP.

> Receive-WinSCPItem -WinSCPSession $WinSCPSession -RemotePath '/media/DATA/utile_windows/Bonjour/Bonjour64.msi' -LocalPath "C:\"                  

   Destination: C:\

IsSuccess FileName
--------- --------
True      Bonjour64.msi
```

#### Fermer connexion SCP
```
> Remove-WinSCPSession

> Remove-Variable WinSCPSessionOption

```

## Debugguer une commande
[Source](https://stackoverflow.com/questions/21033379/what-is-the-alias-keyword-in-powershell/21052658#21052658)
```
> Trace-Command -Expression { alias } -Name CommandDiscovery -PSHost
DEBUG: CommandDiscovery Information: 0 : Looking up command: alias
DEBUG: CommandDiscovery Information: 0 : PATH: C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Program Files\PowerShell\7\
DEBUG: CommandDiscovery Information: 0 : Looking for alias.* in C:\Windows\system32
DEBUG: CommandDiscovery Information: 0 : Looking for alias.* in C:\Windows
DEBUG: CommandDiscovery Information: 0 : Looking for alias.* in C:\Windows\System32\Wbem
DEBUG: CommandDiscovery Information: 0 : Looking for alias.* in C:\Windows\System32\WindowsPowerShell\v1.0\
DEBUG: CommandDiscovery Information: 0 : Looking for alias.* in C:\Program Files\PowerShell\7\
DEBUG: CommandDiscovery Information: 0 : The command [alias] was not found, trying again with get- prepended
DEBUG: CommandDiscovery Information: 0 : Looking up command: get-alias
DEBUG: CommandDiscovery Information: 0 : Cmdlet found: Get-Alias  Microsoft.PowerShell.Commands.GetAliasCommand

Suivi du résultat de la commande alias
...
```
=> on voit que la commande **alias** (sans distinction de casse) n'est pas un alias à proprement parler mais un raccourci de **Get-Alias**
