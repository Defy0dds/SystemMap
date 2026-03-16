# 🗺️ map — A Pretty Tree Command for PowerShell
A minimal, elegant `tree`-style directory visualizer for PowerShell 5.1+.  
No dependencies. No install. Just paste and go.

---

## ⚡ One-liner Install
```powershell
New-Item -Path "$HOME\Documents\WindowsPowerShell" -ItemType Directory -Force; @'
function map {
    param(
        [string]$Path = ".",
        [string]$Indent = "",
        [switch]$ShowSize,
        [int]$Depth = [int]::MaxValue,
        [int]$_level = 0
    )
    $tee    = [char]0x251C + [char]0x2500 + [char]0x2500 + " "
    $corner = [char]0x2514 + [char]0x2500 + [char]0x2500 + " "
    $pipe   = [char]0x2502 + "   "
    $blank  = "    "
    if ($_level -ge $Depth) { return }
    $items = Get-ChildItem -LiteralPath $Path -ErrorAction SilentlyContinue
    for ($i = 0; $i -lt $items.Count; $i++) {
        $item   = $items[$i]
        $isLast = $i -eq $items.Count - 1
        $branch = if ($isLast) { $corner } else { $tee }
        $child  = if ($isLast) { $blank  } else { $pipe }
        $label  = $item.Name
        if ($ShowSize -and -not $item.PSIsContainer) { $label += "  $(Format-FileSize $item.Length)" }
        Write-Output "$Indent$branch$label"
        if ($item.PSIsContainer) { map -Path $item.FullName -Indent "$Indent$child" -ShowSize:$ShowSize -Depth $Depth -_level ($_level + 1) }
    }
}
function Format-FileSize([long]$bytes) {
    if ($bytes -ge 1GB) { return "{0:N1} GB" -f ($bytes / 1GB) }
    if ($bytes -ge 1MB) { return "{0:N1} MB" -f ($bytes / 1MB) }
    if ($bytes -ge 1KB) { return "{0:N1} KB" -f ($bytes / 1KB) }
    return "$bytes B"
}
'@ | Set-Content -Path "$HOME\Documents\WindowsPowerShell\Microsoft.PowerShell_profile.ps1" -Force; . $PROFILE
```
> Paste this into any PowerShell 5.1 terminal. Done.

---

## 🖥️ Output
```
├── src
│   ├── index.ts
│   └── utils.ts
├── tests
│   └── index.test.ts
└── package.json  4.2 KB
```

---

## 📖 Usage
```powershell
# Current directory
map

# Specific path
map C:\Projects\MyApp

# Limit depth
map -Depth 2

# Show file sizes
map -ShowSize

# Combine flags
map C:\Projects\MyApp -Depth 3 -ShowSize
```

---

## ⚙️ Parameters
| Parameter   | Type     | Default | Description                          |
|-------------|----------|---------|--------------------------------------|
| `-Path`     | `string` | `.`     | Root directory to map                |
| `-Depth`    | `int`    | `∞`     | Max folder depth to recurse          |
| `-ShowSize` | `switch` | off     | Display file sizes next to filenames |

---

## 🔧 How It Works
Installs a `map` function into your PowerShell profile (`Microsoft.PowerShell_profile.ps1`), so it's available in every session automatically.

- Uses `├──`, `└──`, and `│` for proper tree branching
- Box-drawing characters are built at runtime via `[char]` and Unicode code points — the profile file stays pure ASCII, so encoding is never an issue regardless of how the file is saved or transferred
- Gracefully skips folders you don't have permission to read
- Human-readable file sizes (B / KB / MB / GB)

---

## 🔄 Reload Profile
If you've already run the one-liner and want to activate changes in your current session:
```powershell
. $PROFILE
```

---

## 📋 Requirements
- Windows PowerShell 5.1  
- No admin rights required

---

## 📄 License
MIT — do whatever you want with it.
