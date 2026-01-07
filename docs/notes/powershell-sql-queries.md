---
title: 'Using Powershell for SQL queries'
taxonomy:
    category:
        - docs
---

```
Push-Location

# load sql module
Write-Host "Loading Powershell SQL ..."
Set-ExecutionPolicy -Scope CurrentUser Unrestricted
Import-Module -DisableNameChecking SQLPS

# variables
$instance = ".\sqlexpress"
$database = "db"

$now = ((Get-Date -format s) -replace":","")
$workdir = "\\server01\share"
$companies_txt = $workdir + "\exports\companies.txt"

$sqlCompanies = "SELECT 
[ACCOUNT],[NAME],[ADDRESS1],[ADDRESS2],[ZIPCITY],[COUNTRY],[ATTENTION],[PHONE],[SALESREP],[EMAIL],[CELLPHONE] 
FROM [dbo].[CUSTTABLE]"


if (Test-Path -Path $companies_txt -PathType Leaf) {
    Remove-Item $companies_txt -Force
}

Write-Host "Exporting companies"
Invoke-Sqlcmd -ServerInstance $instance -Database $database -Query $sqlCompanies | Export-Csv -Path $companies_txt -NoTypeInformation -Encoding UTF8

#$utf8 = New-Object System.Text.UTF8Encoding $False
#$rawData = Get-Content -Raw $companies_txt
#[System.IO.File]::WriteAllLines($companies_txt, $rawData, $utf8)

Write-Host "Done!"

cd $workdir
```