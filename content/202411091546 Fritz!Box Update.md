- eva_tools runterladen
- Mit Windows PowerShell in den Ordner wechseln (cd)
- Mit folgendem Befehl die Fritzbox im Bootloader anhalten:
`./EVA-Discover.ps1`
- Dann mit folgenden Befehlen RTL Status von n auf y setzen
`./EVA-FTP-Client.ps1 -ScriptBlock { SetEnvironmentValue DMC "RTL=y,SL1"}` 
- Neustarten
`./EVA-FTP-Client.ps1 -ScriptBlock { RebootTheDevice }`