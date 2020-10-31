---
title: "Custom Bindings"
date: 2020-10-31T17:05:10+01:00
draft: false
---

Because I use vim keybindings in VSCode, and find them pretty helpful, I decided it made sense to switch over to using them on the commandline as well, with a handful of modifications

## Basic Setup

```powershell
function OnViModeChange {
    if ($args[0] -eq 'Command') {
        # Set the cursor to a blinking block.
        Write-Host -NoNewLine "`e[1 q"
    } else {
        # Set the cursor to a blinking line.
        Write-Host -NoNewLine "`e[5 q"
    }
}
$viModeParams = @{
    EditMode = 'Vi'
    ViModeIndicator = 'Script'
    ViModeChangeHandler = $Function:OnViModeChange
}
Set-PSReadLineOption @viModeParams -BellStyle None
```

This sets up the basic commandline, including changing the cursor when you switch between modes (the function comes from the [Set-PsReadLine Docs](https://docs.microsoft.com/en-us/powershell/module/psreadline/set-psreadlineoption?view=powershell-7)). BellStyle None stops it from beeping, which isn't part of the basic setup but I can't imagine anyone not wanting.

## Extra Features

```powershell
Set-PSReadLineKeyHandler -Chord Ctrl+r -Function ReverseSearchHistory
```

The vim search function is far worse than the emacs version, mostly because it doesn't preview results. This line adds back the old reverse search in edit mode. The vim search is still available with the same command in normal mode.

```powershell
Set-PSReadlineKeyHandler -Chord Ctrl+q -Function Complete
```

Powershell's command completion is normally the better choice for cmdlet parameters, but the bash version of completions often works better for file paths. Also, sometimes it is handy to get a list of all the parameters at once. This maps the bash completion function to ctrl+q, which is close enough to tab to keep it handy for those occasions.

```powershell
Set-PSReadLineKeyHandler -Chord Alt+. -function YankLastArg
```

Maybe the emacs shortcut I use the most. repeat the last word from the previous command, or keep pressing it to cycle through the last words of previous commands. It's almost always the filepath or object being acted on.