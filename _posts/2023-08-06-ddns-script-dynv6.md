弄了个PowerShell脚本来注册ddns到dynv6

请升级到powershell 7.x
- Invoke-RestMethod才能支持NoProxy，如果不需要这个参数则无所谓了。
- 需要先建立一个日志
```powershell
New-EventLog -LogName "ddns" -Source "ddns"
```

以下是脚本，可以保存到计划任务里执行
```powershell
#Requires -Version 7.0

# Get actual ipv6
$current = Get-NetIPAddress -AddressFamily IPv6| Where-Object { $_.PrefixOrigin -match 'RouterAdvertisement'}| Where-Object { $_.SuffixOrigin -match 'Link'} | Where-Object {$_.AddressState -match 'Preferred'} | Where-Object {$_.IPAddress -like '24*'}| Select-Object -ExpandProperty IPAddress

$token = "your token"
$hostname ="your domain"
$url = ("http://dynv6.com/api/update?hostname=" + $hostname + "&ipv6=" + $current + "&token=" + $token)

try {
    # 尝试执行 Invoke-RestMethod 并获取返回结果
    $response = Invoke-RestMethod -NoProxy -Uri $url

    # 判断返回值是否包含 "addresses updated"
    if ($response -like "*addresses updated*") {
        # 如果返回值表明成功，则写入成功日志
        Write-EventLog -LogName "ddns" -Source "ddns" -EventID 3102 -EntryType Information -Message "$hostname($current)已更新" -Category 1 -RawData 10,20
    } else {
        # 如果返回值不符合预期，则记录警告日志
        Write-EventLog -LogName "ddns" -Source "ddns" -EventID 3104 -EntryType Warning -Message "返回值异常：$response" -Category 1
    }
} catch {
    # 捕获异常并记录错误日志
    Write-EventLog -LogName "ddns" -Source "ddns" -EventID 3103 -EntryType Error -Message "更新失败：$($_.Exception.Message)" -Category 1
}
```
