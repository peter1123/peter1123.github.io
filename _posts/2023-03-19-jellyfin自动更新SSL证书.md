Windows平台

Jellyfin没有注册为系统服务

需要将certbot和openssl添加到环境变量中

把下面的powershell脚本添加到计划任务中

```powershell
# 1.定义变量
$domain = "your.domain.com"
$pfxPath = "C:\emby\$domain.pfx"
$privkeyPath = "C:\Certbot\live\$domain\privkey.pem"
$certPath = "C:\Certbot\live\$domain\cert.pem"
$password = 'password0！@#'
$daysLeft = 30

# 2.检查证书是否存在并且即将过期
if (Test-Path $pfxPath) {
    $cert = openssl pkcs12 -in $pfxPath  -passin pass:$password -nodes | openssl x509 -noout -dates
    $expirationDate = ($cert | Select-String "notAfter=").ToString().Split('=')[1]
    $remainingDays = [DateTime]::ParseExact($expirationDate.trim(), "MMM dd HH:mm:ss yyyy 'GMT'", [Globalization.CultureInfo]::InvariantCulture) - (Get-Date)
    if ($remainingDays.Days -gt $daysLeft) {
        Write-Host "证书还有 $($remainingDays.Days) 天过期，无需更新。"
        exit
    }
}

# 3.停止Jellyfin进程
Stop-Process -Name Jellyfin.Windows.Tray -Force
Stop-Process -Name Jellyfin -Force

# 4.使用standalone方式运行Certbot获取SSL证书
certbot certonly --standalone --force-renewal -d $domain

# 5.将证书转换为PFX格式

openssl pkcs12 -export -out $pfxPath -inkey $privkeyPath -in $certPath -passout pass:$password

# 6.启动Jellyfin进程
Start-Process -FilePath "C:\Program Files\Jellyfin\Server\Jellyfin.Windows.Tray.exe"

# 7.等待Jellyfin启动，然后检查证书是否更新成功

```
