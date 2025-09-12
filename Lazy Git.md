Normal push:
```PowerShell
lazygit "Fixed login bug" 
```

Add and commit only:
```PowerShell
lazygit "Fixed login bug" -NP
```

Test with:
```PowerShell
Test-Path $PROFILE
```

And if false, use:
```PowerShell
New-Item -Type File -Force $PROFILE
```

To open the file:
```PowerShell
notepad $PROFILE
```

And to update:
```PowerShell
. $PROFILE
```

```PowerShell
function lazygit {
    param(
        [Parameter(Mandatory = $true)]
        [string]$Message,

        [switch]$NP
    )

    git rev-parse --is-inside-work-tree > $null 2>&1
    if ($LASTEXITCODE -ne 0) {
        Write-Host "============================================="
        Write-Host "[EXIT] Not inside a Git repository. Aborting."
        Write-Host "============================================="
        return
    }

    # Get current branch for logs
    $branch = git rev-parse --abbrev-ref HEAD
    Write-Host "============================================="
    Write-Host "[INFO] Current branch: $branch"

    Write-Host "============================================="
    Write-Host "[INFO] Staging changes..."
    Write-Host "============================================="
    git add . --dry-run
    git add .
    Write-Host "[INFO] Changes staged."

    # Check if there are staged changes
    git diff --cached --quiet
    if ($LASTEXITCODE -eq 0) {
        Write-Host "============================================="
        Write-Host "[EXIT] No changes to commit."
        Write-Host "============================================="
        return
    }

    if ([string]::IsNullOrWhiteSpace($Message)) {
        Write-Host "============================================="
        Write-Host "[EXIT] Commit message is empty. Aborting."
        Write-Host "============================================="
        return
    }


    Write-Host "============================================="
    Write-Host "[INFO] Committing with message: $Message"
    Write-Host "============================================="
    git commit -a -m $Message

    if ($LASTEXITCODE -ne 0) {
        Write-Host "============================================="
        Write-Host "[EXIT] Commit failed. Aborting push."
        Write-Host "============================================="
        return
    }

    if (-not $NP) {
        Write-Host "============================================="
        Write-Host "[INFO] Pushing to remote..."
        git push

        if ($LASTEXITCODE -eq 0) {
            Write-Host "============================================="
            Write-Host "[EXIT] Push successful!"
            Write-Host "============================================="
        } else {
            Write-Host "============================================="
            Write-Host "[EXIT] Push failed."
            Write-Host "============================================="
        }
    } else {
        Write-Host "============================================="
        Write-Host "[EXIT] Push skipped (NP flag used)"
        Write-Host "============================================="
    }
}
```
