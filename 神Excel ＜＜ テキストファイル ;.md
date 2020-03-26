---
title: 神Excel << テキストファイル ;
tags: PowerShell Excel
author: zetamatta
slide: false
---
* **(2018.07.05) 追記：Go言語に移植しました**
   <br /> → https://github.com/zetamatta/godexcel

```shell-session
[C:] GodExcel.cmd C3 AA30 tmp.xls ./GodExcel.cmd
```

![無題.png](https://qiita-image-store.s3.amazonaws.com/0/29454/5f00cd76-589a-eb96-d298-b3a89847a62f.png)

```ps1:GodExcel.cmd
@set args=%*
@powershell "iex( (@('','','')+(cat '%~f0'|select -skip 3))-join[char]10)"
@exit /b %ERRORLEVEL%

$std_cell_width = 1.50

function Split-LikeShell($s){
    $rx = [regex]'"[^"]*"'
    while( $true ){
        $m = $rx.Match($s)
        if( -not $m.Success ){
            break
        }
        $left = $s.SubString(0,$m.Index)
        $right = $s.SubString($m.Index+$m.Length)
        $mid = (($m.Value -replace " ",[char]1) -replace '"','')
        $s = $left + $mid + $right
    }
    ($s -split " ") | ForEach-Object{ $_ -replace [char]1," " }
}

$args = (Split-LikeShell $env:args)

function Conv-RC($str){
    $i = 0
    $str = $str.ToUpper()
    $col = 0
    while( $i -lt $str.Length ){
        $index = "ABCDEFGHIJKLMNOPQRSTUVWXYZ".IndexOf($str.substring($i,1))
        if( $index -lt 0 ){
            break
        }
        $col = $col * 26 + ($index+1)
        $i++
    }
    $row = 0
    while( $i -lt $str.Length ){
        $index = "0123456789".IndexOf($str.substring($i,1))
        if( $index -lt 0 ){
            break
        }
        $row = $row * 10 + $index
        $i++
    }
    return @($row,$col)
}

$left = 1
$top = 3
$right = 200
$bottom = 200

if( $args.Length -lt 2 ){
    Write-Output `
        "Usage: godexcel.ps1 [LeftTop] [RightBottom] ExcelFile TextFile..."
    exit
}

if( $args.Length -ge 1 -and $args[0] -match "^[A-Za-z]+[0-9]+$" ){
    $rc = Conv-RC($args[0])
    $top = $rc[0]
    $left = $rc[1]
    $args = $args[1..($args.Length-1)]
    if( $args.Length -ge 1 -and $args[0] -match "^[A-Za-z]+[0-9]+$" ){
        $rc = Conv-RC($args[0])
        $bottom = $rc[0]
        $right = $rc[1]
        $args = $args[1..($args.Length-1)]
    }
}

$excel = New-Object -ComObject Excel.Application
$cellwidth = @{}

try{
    $excel.Visible = $true
    
    $filename = [System.IO.Path]::GetFullPath($args[0])
    [System.Console]::WriteLine("Excel=$filename")
    if( Test-Path $args[0] ){
        $book = $excel.WorkBooks.Open($filename)
        $new = $false
    }else{
        $book = $excel.WorkBooks.Add()
        $new = $true
    }
    $sheet = $book.ActiveSheet

    $row = $top
    for($i = 1 ; $i -lt $args.Length ; $i++){
        Get-Content $args[1] |
        ForEach-Object {
            if( $row -le $bottom ){
                $count = $_.Length
                $col = $left
                for($j=0 ; $j -lt $count ; $j++){
                    if( $col -gt $right ){
                        $col = $left
                        $row++
                    }
                    if( $new -and -not $cellwidth.ContainsKey($col) ){
                        $sheet.Columns($col).ColumnWidth = $std_cell_width
                        $cellwidth[$col] = $true
                    }
                    $sheet.Cells.Item($row,$col) = $_.substring($j,1)
                    $col++
                }
                while( $col -le $right ){
                    $sheet.Cells.Item($row,$col) = ""
                    $col++
                }
                $row++
            }
        }
    }
    if( $new ){
        $book.SaveAs($filename)
    }
}finally{
    $excel.Quit()
}

# vim:set ft=ps1:
```

