```bash
@echo off
chcp 65001
echo.
setlocal
echo 你需要有github账号和具有gist权限的access_token;
echo.
echo 新建一个txt文件，复制全部代码，将上述的access_token填入代码中，重命名为.bat后缀的文件即可使用;
echo.
echo 按理说不限制上传文件大小，支持txt、yaml等文本文件，支持csv文件;
echo.
echo 要求所有文件的编码为utf8，否则会出现中文和符号乱码;
echo.
echo 实测win10自带的powershell版本5.1上传中文乱码，最好升级你的powershell版本到7+;
echo.
echo 确认你的powershell目录存在的是pwsh.exe还是powershell.exe，若是后者，将代码默认的pwsh替换成powershell。
echo.
rem 替换YOUR_ACCESS_TOKEN为你的GitHub访问令牌
set TOKEN=YOUR_ACCESS_TOKEN

rem 设置上传到指定gist的gistId，不填此项则创建新的gist
set GIST_ID=

rem 设置上传后的Gist描述
set GIST_DESCRIPTION=你的gist描述

rem 检查是否有文件拖入
if "%~1"=="" (
    echo No file provided.
    goto :eof
)

set "FILE_PATH=%~1"
set "CHUNK_SIZE=1000000"  rem 设置每个分片的大小，单位为字节

pwsh -Command ^
    "$token = '%TOKEN%';" ^
    "$filePath = '%FILE_PATH%';" ^
    "$fileName = [System.IO.Path]::GetFileName($filePath);" ^
    "$fileContent = [System.IO.File]::ReadAllText($filePath);" ^
    "$chunkSize = %CHUNK_SIZE%;" ^
    "$totalLength = $fileContent.Length;" ^
    "$numChunks = [Math]::Ceiling($totalLength / $chunkSize);" ^
    "$gistId = '%GIST_ID%';" ^
    "for ($i = 0; $i -lt $numChunks; $i++) {" ^
        "$start = $i * $chunkSize;" ^
        "$end = [Math]::Min($totalLength, ($i + 1) * $chunkSize);" ^
        "$chunkContent = $fileContent.Substring($start, $end - $start);" ^
        "if ($i -eq 0 -and $gistId.Length -eq 0) {" ^
            "$gist = @{ description = '%GIST_DESCRIPTION%'; files = @{ ($fileName) = @{ content = $chunkContent } }; public = $false };" ^
            "$response = Invoke-RestMethod -Uri 'https://api.github.com/gists' -Headers @{ Authorization = 'token ' + $token } -Method POST -ContentType 'application/json' -Body ($gist | ConvertTo-Json);" ^
            "$gistId = $response.id;" ^
        "} else {" ^
            "$update = @{ description = '%GIST_DESCRIPTION%'; files = @{ ($fileName) = @{ content = $chunkContent } } };" ^
            "$response = Invoke-RestMethod -Uri ('https://api.github.com/gists/' + $gistId) -Headers @{ Authorization = 'token ' + $token } -Method PATCH -ContentType 'application/json' -Body ($update | ConvertTo-Json);" ^
        "}" ^
    "}" ^
    "Write-Output ('文件上传到Gist完成！	URL: https://gist.github.com/' + $gistId)" ^

    "Write-Output ('')" ^

    "Write-Output ('请保留你gistId：	' + $gistId + '	填入本代码以便覆盖同名文件！')" ^
	
endlocal
echo.
echo 上传完成！10秒后自动关闭窗口……
timeout /t 10 >nul
exit
```
