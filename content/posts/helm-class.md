---
title: "Representing a Helm chart as a class in Powershell"
date: 2020-10-25T17:16:56+01:00
draft: false
---

A [Helm Chart](https://helm.sh/docs/topics/charts/) is a collection of files inside a directory. At a minimum, the collection should include a chart.yaml and values.yaml file. These two files define the behaviour of the chart and should be in the root of the directory. To indicate a particular chart, all we need is the path to the directory (```$path```).
  
The helm cli will check that the files are formatted to its specifications but there are likely to also be business requirements for your Helm charts which need to be checked in your deployment pipeline. To do this it will be necessary to read and possibly update the contents of the chart and values yaml. All of this could be done using functions but wrapping the chart in a class makes the task clearer.

```powershell
class Chart {
    [system.io.fileInfo]$Path
    [system.io.fileInfo]$ChartPath
    [system.io.fileInfo]$ValuesPath
    [hashtable]$Chart
    [hashtable]$Values

    Chart(
        [system.io.fileInfo]$Path
    ){
        $this.Path = $Path
        $this.ChartPath = Join-Path -Path $Path -ChildPath 'Chart.yaml'
        $this.ValuesPath = Join-Path -Path $Path -ChildPath 'Values.yaml'
        $this.Read()
    }

    [void]Read(){
        $this.Chart = ConvertFrom-Yaml -Path $this.ChartPath
        $this.Values = ConvertFrom-Yaml -Path $this.ValuesPath
    }

    [void]Write(){
        $this.Chart | ConvertTo-Yaml | Set-Content -Path $this.ChartPath
        $this.Values | ConvertTo-Yaml | Set-Content -Path $this.ValuesPath
    }
}

function Build-Chart {
    param (
        [system.io.fileInfo]$Path
    )
    $chart = [chart]::new($Path)
    # EXTRA VALIDATIONS, EG:
    $extraValue = $chart.values.keys |
        Where-Object { $_ -notin $chart.chart.dependencies }
    if($extraValue) {
        throw "Were these supposed to match something? '$extraValue'"
    }
    # BUSINESS LOGIC, EG:
    if($chart.chart.dependencies -contains 'this.repository'){
        $chart.chart.description += ' that'
    }
    $chart.Write()
    $chart
}
```
