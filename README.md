# Useful-PS-Scripts
Assortment of commands

<details>
    <summary> <b> Get all registered windows URI handlers </b> </summary>
	I couldn't find anything anywhere to get a list of all registered protocol/uri handlers so I put this together.<br/>
    Outputs the uri scheme name, registry description and the command or ddeexec + verb invoked
	
```powershell
Param(
    [Switch]
    $WrapOutput,
    [Switch]
    $List
)

Push-Location

cd HKLM:\SOFTWARE\Classes

ls |
Get-ItemProperty |
where -Property "(default)" -Like "URL:*" |
    select -Property "PSChildName", "(default)", @{
        l="Shell Command";
        e ={
            ls $_.PSChildName -Recurse -Include "command","ddeexec" |
            gp |
            %{ "$($_.PSChildName) $(Split-Path $_.PSParentPath -Leaf) $(select -InputObject $_ -ExpandProperty '(default)')" }
        }
    } -OutVariable res 1>$null

if ($List.IsPresent) {
    fl -InputObject $res
} else {
    ft -InputObject $res -Wrap:$WrapOutput
}

Pop-Location
```
</details>

<details>
<summary> <b> Hash contents of folder </b> </summary>
    
```powershell
Get-ChildItem | ForEach-Object {Get-FileHash $_.FullName -Algorithm MD5}
```
</details>

<details>
<summary> <b> Force change desktop wallpaper </b> </summary>
    
```powershell

H:;
cd H:\Downloads;
$file = Read-Host -Prompt "Filename";
$file = "/" + $file;
$path = "C:\Users\[PROFILENAME]\AppData\Roaming\Microsoft\Windows\Themes";
cp .\$file $path ; rm $path\TranscodedWallpaper -ErrorAction SilentlyContinue;
rni ($path + $file) $path\TranscodedWallpaper;
rundll32.exe USER32.DLL,UpdatePerUserSystemParameters 1, True;
```
</details>

<details>
<summary> <b> Powershell Autoclicker </b> </summary>
    
```powershell
    
Add-Type -TypeDefinition @'

using System;
using System.Runtime.InteropServices;

public class Clicker{

    [StructLayout(LayoutKind.Sequential)]
    struct INPUT{
        public int type;

        public MOUSEINPUT mi;
    }

    [StructLayout(LayoutKind.Sequential)]
    struct MOUSEINPUT{
       public int dx;
       public int dy;
       public int mouseData;
       public int dwFlags;
       public int time;
       public IntPtr dwExtraInfo; 
    }

    const int MOUSEEVENTF_MOVED      = 0x0001 ;
    const int MOUSEEVENTF_LEFTDOWN   = 0x0002 ;
    const int MOUSEEVENTF_LEFTUP     = 0x0004 ;
    const int MOUSEEVENTF_RIGHTDOWN  = 0x0008 ;
    const int MOUSEEVENTF_RIGHTUP    = 0x0010 ;
    const int MOUSEEVENTF_MIDDLEDOWN = 0x0020 ;
    const int MOUSEEVENTF_MIDDLEUP   = 0x0040 ;
    const int MOUSEEVENTF_WHEEL      = 0x0080 ;
    const int MOUSEEVENTF_XDOWN      = 0x0100 ;
    const int MOUSEEVENTF_XUP        = 0x0200 ;
    const int MOUSEEVENTF_ABSOLUTE   = 0x8000 ;

    [DllImport("User32.dll")]
    extern static uint SendInput(uint nInputs, INPUT[] pInputs, int cbSize);

    public static void Click() {
        INPUT[] input = new INPUT[2];
        input[0].mi.dwFlags = MOUSEEVENTF_LEFTDOWN;
        input[1].mi.dwFlags = MOUSEEVENTF_LEFTUP;
        SendInput(2,input,Marshal.SizeOf(input[0]));
    }
}


'@

while (1 -eq 1) { start-sleep -m 5; [Clicker]::Click() }
```
</details>

<details>
<summary> <b> View system proxy settings </b> </summary>
<br>
    
```powershell

Get-ItemProperty 'HKCU:\Software\Microsoft\Windows\CurrentVersion\Internet Settings'
```
or
```powershell

netsh winhttp show proxy
```
However this sometimes returns direct access no proxy (when there is) depending on how it is called.
</details>

<details>
<summary> <b> Constrained Language Mode -> Full Language Mode bypass </b> </summary>
<br>
    
Powershell 2.0 will need to be installed for this.
Check for current language mode:

```powershell
$ExecutionContext.SessionState.LanguageMode
```
[The full list of what CLM blocks can be found here](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/)

If the system still has poweshell 2.0 (which the majority of people dont disable) you can run:

```powershell

powershell.exe -Version 2
```
to get a FLM session.
</details>
