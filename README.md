# powershell
Ici des solutions toutes faites en PS

## Résultat dans fenêtre graphique avec possibilité de rechercher textuellement

Event Viewer plus pratique
```
Get-WinEvent -ListLog * | % { Get-WinEvent -LogName $_.LogName | Select -Property * } | Out-GridView
```
Nota : si LogName ne contient aucune entrée (cas de "Internet Explorer" après installation), une erreur s'affichera en rouge sans conséquences
